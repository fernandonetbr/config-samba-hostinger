# config-samba-hostinger

Criar uma VPS na Hostinger para ser usada como repositório de arquivos pode ser uma boa solução para armazenar e acessar arquivos remotamente. Abaixo, vou descrever o processo passo a passo de como fazer isso.

### 1. Adquirir uma VPS na Hostinger

1. **Acesse o site da Hostinger**:
   - Vá para [hostinger.com](https://www.hostinger.com/).

2. **Faça login ou crie uma conta**:
   - Se já tiver uma conta, faça login.
   - Caso contrário, crie uma nova conta.

3. **Escolha um plano de VPS**:
   - Navegue até a seção de VPS.
   - Escolha um plano que atenda às suas necessidades de armazenamento e recursos.

4. **Configure e compre o plano**:
   - Siga as instruções para configurar e comprar o plano escolhido.
   - Isso pode incluir a seleção do sistema operacional (por exemplo, Ubuntu, CentOS, Debian, etc.).

### 2. Configuração Inicial da VPS

1. **Acesse o Painel de Controle da Hostinger**:
   - Após a compra, acesse o painel de controle (hPanel).

2. **Inicialize a VPS**:
   - Vá para a seção de VPS no hPanel.
   - Inicialize a sua VPS usando o sistema operacional escolhido.

3. **Obtenha os detalhes de acesso**:
   - Anote o endereço IP da VPS, nome de usuário (geralmente 'root') e senha.

### 3. Conectar à VPS

1. **Use SSH para conectar à VPS**:
   - No seu computador local, abra um terminal (ou use um cliente SSH como PuTTY no Windows).
   - Digite o comando:
     ```bash
     ssh root@seu_ip_da_vps
     ```
   - Substitua `seu_ip_da_vps` pelo endereço IP da sua VPS.
   - Quando solicitado, insira a senha.

### 4. Instalar um Servidor de Arquivos

Para criar um repositório de arquivos, você pode usar o `Samba`, `FTP` ou até mesmo um `NFS`. Aqui, vamos usar `Samba` como exemplo.

1. **Atualize os pacotes do sistema**:
   ```bash
   apt update && apt upgrade -y  # Para distribuições baseadas em Debian/Ubuntu
   yum update -y  # Para distribuições baseadas em CentOS/RHEL
   ```

2. **Instale o Samba**:
   ```bash
   apt install samba -y  # Para distribuições baseadas em Debian/Ubuntu
   yum install samba -y  # Para distribuições baseadas em CentOS/RHEL
   ```

3. **Configurar o Samba**:
   - Abra o arquivo de configuração do Samba:
     ```bash
     nano /etc/samba/smb.conf
     ```
   - Adicione uma nova seção ao final do arquivo para o compartilhamento de arquivos:
     ```ini
     [FileShare]
     path = /srv/samba/share
     browseable = yes
     read only = no
     guest ok = yes
     ```
   - Salve e feche o arquivo (`Ctrl+X`, depois `Y` e `Enter`).

4. **Crie o diretório para compartilhamento de arquivos**:
   ```bash
   mkdir -p /srv/samba/share
   chmod 0777 /srv/samba/share
   ```

5. **Reinicie o serviço Samba**:
   ```bash
   systemctl restart smbd
   ```

### 1. Instalar Nginx

1. **Atualize os pacotes do sistema**:
   ```bash
   apt update && apt upgrade -y  # Para distribuições baseadas em Debian/Ubuntu
   yum update -y  # Para distribuições baseadas em CentOS/RHEL
   ```

2. **Instale o Nginx**:
   ```bash
   apt install nginx -y  # Para distribuições baseadas em Debian/Ubuntu
   yum install nginx -y  # Para distribuições baseadas em CentOS/RHEL
   ```

### 2. Configurar Nginx

1. **Crie um diretório para os arquivos compartilhados** (se não tiver feito anteriormente):
   ```bash
   mkdir -p /srv/samba/share
   chmod 0777 /srv/samba/share
   ```

2. **Configure Nginx para servir essa pasta**:
   - Crie um novo arquivo de configuração para o seu site. Por exemplo, `/etc/nginx/sites-available/fileshare`:
     ```bash
     nano /etc/nginx/sites-available/fileshare
     ```
   - Adicione a seguinte configuração:
     ```nginx
     server {
         listen 80;
         server_name sua_dominio.com;  # Substitua pelo seu domínio ou IP público
         
         location / {
             root /srv/samba/share;
             autoindex on;
             autoindex_exact_size off;
             autoindex_localtime on;
         }
     }
     ```

3. **Ative a configuração do site**:
   ```bash
   ln -s /etc/nginx/sites-available/fileshare /etc/nginx/sites-enabled/
   ```

4. **Teste a configuração do Nginx**:
   ```bash
   nginx -t
   ```

5. **Reinicie o Nginx**:
   ```bash
   systemctl restart nginx
   ```

### 3. Configurar HTTPS com Let's Encrypt

Para fornecer HTTPS, você pode usar Let's Encrypt para obter um certificado SSL gratuito.

1. **Instale Certbot**:
   ```bash
   apt install certbot python3-certbot-nginx -y  # Para distribuições baseadas em Debian/Ubuntu
   yum install certbot python3-certbot-nginx -y  # Para distribuições baseadas em CentOS/RHEL
   ```

2. **Obtenha e instale o certificado SSL**:
   ```bash
   certbot --nginx -d sua_dominio.com  # Substitua pelo seu domínio
   certbot --nginx -d seudominio.com -d subdominio.seudominio.com # Substitua pelo seu domínio e subdomínio
   ```

3. **Siga as instruções para concluir a configuração**. Certbot irá automaticamente configurar o Nginx para usar HTTPS.

### 4. Acessar Externamente

Após configurar o Nginx e obter um certificado SSL, você deve ser capaz de acessar a pasta compartilhada via HTTPS:

1. **No Navegador**:
   - Abra um navegador web.
   - Digite `https://sua_dominio.com` (ou o IP público se não tiver um domínio).

2. **Visualizar os Arquivos**:
   - Você deve ver uma lista de arquivos e pastas no diretório `/srv/samba/share`.

### 5. Considerações de Segurança

- **Autenticação**: Considere adicionar autenticação básica no Nginx para proteger o acesso aos seus arquivos.
- **Firewall**: Certifique-se de que as portas 80 (HTTP) e 443 (HTTPS) estejam abertas no seu firewall.
- **Permissões**: Verifique as permissões dos arquivos e pastas para evitar acessos não autorizados.

### Exemplo de Autenticação Básica no Nginx

1. **Instale o utilitário htpasswd**:
   ```bash
   apt install apache2-utils  # Para distribuições baseadas em Debian/Ubuntu
   yum install httpd-tools  # Para distribuições baseadas em CentOS/RHEL
   ```

2. **Crie um arquivo de senha**:
   ```bash
   htpasswd -c /etc/nginx/.htpasswd usuario
   ```

3. **Atualize a configuração do Nginx para usar autenticação básica**:
   - Edite o arquivo `/etc/nginx/sites-available/fileshare`:
     ```nginx
     server {
         listen 80;
         server_name sua_dominio.com;
         
         location / {
             root /srv/samba/share;
             autoindex on;
             autoindex_exact_size off;
             autoindex_localtime on;
             auth

Claro, continuando de onde parei:

### 5. Adicionar Autenticação Básica ao Nginx

3. **Atualize a configuração do Nginx para usar autenticação básica**:
   - Edite o arquivo `/etc/nginx/sites-available/fileshare`:
     ```nginx
     server {
         listen 80;
         server_name sua_dominio.com;
         
         location / {
             root /srv/samba/share;
             autoindex on;
             autoindex_exact_size off;
             autoindex_localtime on;
             auth_basic "Restricted Content";
             auth_basic_user_file /etc/nginx/.htpasswd;
         }
     }
     ```

4. **Teste a configuração do Nginx**:
   ```bash
   nginx -t
   ```

5. **Reinicie o Nginx**:
   ```bash
   systemctl restart nginx
   ```

### 6. Atualizar Configuração para HTTPS

Se você já configurou HTTPS usando Let's Encrypt, certifique-se de que a autenticação básica também esteja aplicada à configuração HTTPS. Geralmente, Certbot atualiza automaticamente a configuração para incluir as diretivas SSL, mas você pode editar o arquivo de configuração para garantir que está tudo correto.

1. **Edite o arquivo de configuração do Nginx**:
   - Abra `/etc/nginx/sites-available/fileshare`:
     ```nginx
     server {
         listen 80;
         server_name sua_dominio.com;
         
         location / {
             return 301 https://$host$request_uri;
         }
     }

     server {
         listen 443 ssl;
         server_name sua_dominio.com;

         ssl_certificate /etc/letsencrypt/live/sua_dominio.com/fullchain.pem;
         ssl_certificate_key /etc/letsencrypt/live/sua_dominio.com/privkey.pem;
         include /etc/letsencrypt/options-ssl-nginx.conf;
         ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

         location / {
             root /srv/samba/share;
             autoindex on;
             autoindex_exact_size off;
             autoindex_localtime on;
             auth_basic "Restricted Content";
             auth_basic_user_file /etc/nginx/.htpasswd;
         }
     }
     ```

2. **Teste a configuração do Nginx**:
   ```bash
   nginx -t
   ```

3. **Reinicie o Nginx**:
   ```bash
   systemctl restart nginx
   ```

### 7. Acessar Externamente com HTTPS e Autenticação

Agora, ao acessar `https://sua_dominio.com`, você será solicitado a inserir um nome de usuário e senha antes de poder visualizar a lista de arquivos no diretório compartilhado.

### 8. Manutenção e Segurança

- **Renovação de Certificados**: Let's Encrypt os certificados são válidos por 90 dias. Certifique-se de configurar a renovação automática:
  ```bash
  certbot renew --dry-run
  ```
- **Monitoramento**: Monitore os logs de acesso e erro do Nginx para detectar qualquer atividade suspeita.
  ```bash
  tail -f /var/log/nginx/access.log
  tail -f /var/log/nginx/error.log
  ```

- **Exluir usuário**: Para excluir usuário criado atráves desse comando "htpasswd -c /etc/nginx/.htpasswd usuario" siga os passos abaixo:       
 ```bash
  Abra o arquivo .htpasswd para edição:
  Você pode usar qualquer editor de texto, como vi, vim, nano, etc: sudo nano /etc/nginx/.htpasswd
  As linhas no arquivo .htpasswd têm o formato usuario:senha_hash. Simplesmente delete a linha inteira que contém o nome do usuário que você deseja remover.
  Salvar e sair do editor.  
  ```
### Conclusão

Seguindo esses passos, você configurou um repositório de arquivos acessível externamente via HTTPS com autenticação básica, utilizando uma VPS na Hostinger. Essa configuração garante que seus arquivos estejam protegidos por criptografia SSL e também por um mecanismo de autenticação básica.
