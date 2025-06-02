# Instalação do Zabbix 7.2 no Ubuntu 24.04 LTS

Este documento descreve o passo a passo da instalação do Zabbix Server, frontend e agente no Ubuntu 24.04 LTS utilizando MariaDB e Apache.

---

## 🧰 Pré-requisitos

Execute todos os comandos como `root` ou com `sudo`.

Atualize os pacotes:
```bash
sudo apt update && sudo apt upgrade -y
```

Instale dependências básicas:
```bash
sudo apt install wget curl gnupg2 -y
```

---

## 📦 Instalando o repositório oficial do Zabbix

```bash
wget https://repo.zabbix.com/zabbix/7.2/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.2-1+ubuntu24.04_all.deb
sudo dpkg -i zabbix-release_7.2-1+ubuntu24.04_all.deb
sudo apt update
```

---

## 🐬 Instalando o MariaDB Server

```bash
sudo apt install mariadb-server -y
sudo systemctl enable mariadb
sudo systemctl start mariadb
```

Crie o banco de dados e usuário do Zabbix:
```bash
sudo mysql -uroot
```

No prompt do MariaDB:
```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'SENHA_FORTE';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## 📥 Instalando o Zabbix Server, Frontend, Agente e Apache

```bash
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y
```

Importe o schema inicial do banco:
```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -p zabbix
```

---

## ⚙️ Configurar o Zabbix Server

Edite o arquivo:
```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

Altere as seguintes linhas:
```conf
DBPassword=(Sua Senha)
```

---

## 🕒 Ajustar timezone do frontend

Edite:
```bash
sudo nano /etc/zabbix/apache.conf
```

Altere a linha:
```apache
php_value date.timezone America/Sao_Paulo
```

---

## 📂 Corrigir caminho do frontend (Zabbix 7.2+)

```bash
sudo cp /etc/apache2/conf-available/zabbix.conf /etc/apache2/conf-available/zabbix.conf.bak
sudo sed -i 's:/usr/share/zabbix:/usr/share/zabbix/ui:g' /etc/apache2/conf-available/zabbix.conf
```

---

## ▶️ Iniciar os serviços

```bash
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```

---

## 🌐 Acessar o frontend

Abra o navegador e acesse:

```
http://SEU-IP/zabbix
```

Exemplo: http://192.168.0.232/zabbix

---

## 🔐 Credenciais iniciais

- **Usuário:** Admin  
- **Senha:** zabbix

- 🌐 Configurar Domínio e SSL com Let's Encrypt
🔧 Adicionando o domínio no Apache

Edite o arquivo de configuração do VirtualHost Zabbix:

sudo nano /etc/apache2/sites-available/zabbix.conf

Altere para algo como:

<VirtualHost *:80>
    ServerAdmin webmaster@seudominio.com.br
    ServerName zabbix.seudominio.com.br 
  

    DocumentRoot /usr/share/zabbix/ui

    <Directory /usr/share/zabbix/ui>
        Options FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/zabbix_error.log
    CustomLog ${APACHE_LOG_DIR}/zabbix_access.log combined
</VirtualHost>

Salve e ative a configuração:

sudo a2ensite zabbix.conf
sudo systemctl reload apache2

🔒 Instalando SSL com Let's Encrypt

Instale o Certbot:

sudo apt install certbot python3-certbot-apache -y

Em seguida, execute o comando abaixo substituindo pelo seu domínio:

sudo certbot --apache -d zabbix.seudominio.com.br

Siga os passos na tela. O Certbot cuidará de:

    Obter o certificado SSL

    Configurar o Apache

    Redirecionar automaticamente HTTP → HTTPS

🔁 Renovação Automática do Certificado

Verifique o agendador (cron) de renovação automática:

sudo certbot renew --dry-run

---

## ✅ Finalização

Após o login, configure hosts, templates e comece o monitoramento.

---
