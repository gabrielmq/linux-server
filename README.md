# Configuração Servidor Linux

Este projeto é um tutorial para configurar um servidor remoto hospedado na AWS [Lightsail](https://aws.amazon.com/pt/lightsail/), fazendo com que este servidor remoto disponibilize uma aplicação [web](https://github.com/gabrielmq/catalogo-itens) desenvolvida em python.

## Configurando uma instancia AWS Lightsail

### Passo 1 - Criando uma instância de servidor na AWS Lightsail

- Faça o login na [AWS Lightsail](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Flightsail.aws.amazon.com%2Fls%2Fwebapp%2Fhome%2Finstances%3Fstate%3DhashArgs%2523%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fparksidewebapp&forceMobileApp=0)
- Na tela principal, clique no botão laranja `Criar instância`.
- Selecione a plataforma `Linux/UNIX`
- Em Selecionar esquema, escolha `Somente SO` e o SO `Ubuntu 18.04 LTS`
- Escolha o plano mais barato, que garante o 1º mês gratuito.
- No campo `Identifique sua instância` de um nome exclusivo a sua instância de servidor.
- Clique em criar instância para que a instância do servidor seja criada

### Passo 2 - Acessando o servidor via SSH

- Faça o download da chave SSH localizada nas configurações de conta da Amazon Lightsail.
- Após realizar o download, mova o arquivo para o diretório `.ssh`.
  - Exemplo: `sudo mv ~/Downloads/lightsail-key.pem ~/.ssh`.
- No prompt de comandos execute o comando `sudo chmod 600 ~/.ssh/lightsail-key.pem`
- Ainda no prompt de comandos execute o comando `ssh -i ~/.ssh/lightsail-key.pem ubuntu@<ip_server>` para se conectar ao servidor.

### Passo 3 - Atualizando os pacotes do servidor e configurando time zone UTC

```
   sudo apt-get update
   sudo apt-get upgrade
   sudo apt-get dist-upgrade
   sudo timedatectl set-timezone UTC
```

- Configure o update automático dos pacotes

```
sudo apt install unattended-upgrades
```

- Altere o arquivo `sudo nano /etc/apt/apt.conf.d/50unattended-upgrades` removendo comentário.

```
Unattended-Upgrade::Allowed-Origins {
        "${distro_id}:${distro_codename}";
        "${distro_id}:${distro_codename}-security";
//      "${distro_id}:${distro_codename}-updates"; # descomente essa linha
//      "${distro_id}:${distro_codename}-proposed";
//      "${distro_id}:${distro_codename}-backports";
};
```

- Altere o arquivo `sudo nano /etc/apt/apt.conf.d/20auto-upgrades` adicionando:

```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

- Execute o comando `sudo dpkg-reconfigure --priority=low unattended-upgrades` para habilitar a atualização automática dos pacotes.

### Passo 4 - Configurando usuário grader

- Execute os comandos abaixo para criar o usuário grader

```
sudo adduser grader
sudo usermod -a -G sudo grader
```

### Passo 5 - Configurando chaves SSH para usuário grader

- Gere as chaves ssh localmente executando o comando `ssh-keygen` e salve no diretório `~/.ssh`
- Configure a chave pública no servidor remoto

```
su - grader # para mudar de usuário
sudo mkdir .ssh/
sudo chown grader:grader .ssh/
sudo touch .ssh/authorized_keys
sudo nano .ssh/authorized_keys #cole o valor da chave.pub
sudo chmod 700 .ssh
sudo chmod 600 .ssh/authorized_keys
```

- Agora já é possível utilizar o comando `ssh -i ~/.ssh/grader_key grader@<ip_server>` para acessar o servidor com o novo usuário.

### Passo 6 - Removendo acesso para usuário root

- Execute o comando `sudo nano /etc/ssh/sshd_config`
- Altere a linha `PermitRootLogin prohibit-password` para `PermitRootLogin no`
- Altere a linha `PasswordAuthentication yes` para `PasswordAuthentication no`
- Adicione `DenyUsers root`

### Passo 7 - Alterar porta SSH de 22 para 2200

- Nas configurações de rede do servidor na AWS LightSail, configure uma porta personalizada no Firewall como: `Custom/TCP 2200`
- Remova a configuração padrão de SSH
- Conectado no servidor remoto pelo prompt execute o comando abaixo e altere a porta de 22 para 2200

```
sudo nano /etc/ssh/sshd_config
```

- Execute o comando `sudo service ssh restart` para reinicializar o serviço.

### Passo 8 - Configurando Firewall

```
sudo ufw status # verifica o estado do firewall
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw enable # habilita o firewall
sudo ufw status # valida se o firewall foi ativado
```

## Realizando deploy da aplicação no servidor

### Passo 1 - Configurando APACHE para servir uma aplicação Python mod_wsgi

- Logado com o usuário grader, execute os comandos

```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi-py3 python3-dev
sudo service apache2 restart
```

- Desabilite a página padrão e reinicie o apache

```
sudo a2dissite 000-default.conf
sudo service apache2 reload
```

### Passo 2 - Configurando o PostgreSQL

- Execute os passos abaixo para instalar e configurar um banco de dados PostgreSql

```
sudo apt-get install postgresql
sudo su - postgres
```

- Execute o comando `psql` para entrar no shell do Postgres e digite os seguintes comandos:

```
postgres=# CREATE DATABASE catalogo;
postgres=# CREATE USER grader;
postgres=# ALTER ROLE grader WITH PASSWORD 'udacity';
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalogo TO grader;
postgres=# \q
```

- Execute o comando `exit` para sair do usuário postgress

### Passo 3 - Instalando Git e clonando projeto

```
sudo apt-get install git
cd /var/www
sudo mkdir FlaskApp && cd FlaskApp/
sudo git clone https://github.com/gabrielmq/catalogo-itens.git
sudo mv ./catalogo-itens ./catalogo && cd catalogo/
```

### Passo 4 - Configurando ambiente e a aplicação

- Altere as variáveis `SQLALCHEMY_DATABASE_URI` e `OAUTH_CREDENTIALS` no arquivo `config.py`

```
SQLALCHEMY_DATABASE_URI = "postgresql://grader:udacity@localhost/catalogo"

# altere your_app_id e your_app_secret pelas chaves do google
OAUTH_CREDENTIALS = {
        "google": {
            "id": "your_app_id",
            "secret": "your_app_secret"
        }
    }
```

_Adicione o endereço http://<ip_server>.xip.io nas configurações de JavaScript Origins e dominios liberados no Console de desenvolvedor do Google_

- Instale o pip e o virtualenv

```
sudo apt-get install python3-pip
sudo python3 -m pip install --upgrade pip
sudo pip3 install virtualenv
sudo virtualenv venv
source venv/bin/activate
```

- Instale as dependências do projeto executando o comando `sudo venv/bin/pip3 install --upgrade -r requirements.txt`
- Instale psycopg2

```
sudo apt-get install libpq-dev python3-dev
sudo venv/bin/pip3 install psycopg2
```

- Configure a variável de ambiente do flask

```
echo "export FLASK_APP=run.py" >> ~/.profile
```

### Passo 5 - Configurando e habilitando Virtual Host

- Crie o arquivo catalogo.conf com o seguinte comando:

```
sudo nano /etc/apache2/sites-available/catalogo.conf
```

- Adicione a configuração abaixo para configurar o Virtual Host

```
<VirtualHost *:80>
	ServerName <ip_server>
    ServerAlias <ip_server>.xip.io
	WSGIScriptAlias / /var/www/FlaskApp/catalogo.wsgi
	<Directory /var/www/FlaskApp/catalogo/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/FlaskApp/catalogo/app/static
	<Directory /var/www/FlaskApp/catalogo/app/>static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

- Habilite o Virtual Host executando `sudo a2ensite catalogo`

### Passo 6 - Criando o arquivo wsgi

```
cd /var/www/FlaskApp
sudo nano catalogo.wsgi
```

- Adicione o seguinte código:

```
#!/usr/bin/python3

import sys
import logging

logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/catalogo")

activate_this = '/var/www/FlaskApp/catalogo/venv/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

from run import app as application
```

- Reinicie o apache executando `sudo service apache2 restart`
- Para monitorar o log do apache execute `sudo tail -f /var/log/apache2/error.log`

### Passo 7 - Testando a aplicação

Abra o navegador e acesse o endpoint http://<ip_server>.xip.io/api/categorias/all, o retorno deve ser igual o JSON abaixo

```
{
  "categoria": [
    {
      "id": 1,
      "nome": "Soccer"
    },
    ...
    {
      "id": 8,
      "nome": "Skating"
    },
    {
      "id": 9,
      "nome": "Hockey"
    }
  ]
}
```

Após testar o endpoint acesse http://<ip_server>.xip.io para ter acesso a aplicação e cadastrar novos itens depois de logado.

## Referências

- https://help.ubuntu.com/community/AutomaticSecurityUpdates
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
- https://sempreupdate.com.br/como-conceder-e-remover-privilegios-sudo-no-ubuntu/
- https://realpython.com/flask-by-example-part-2-postgres-sqlalchemy-and-alembic/
- https://askubuntu.com/questions/27559/how-do-i-disable-remote-ssh-login-as-root-from-a-server
- https://github.com/andrevst/fsnd-p6-linux-server-configuration
- https://github.com/adityamehra/udacity-linux-server-configuration
- https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-xvii-deployment-on-linux
- https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-iv-database

# Licensa

Este projeto foi desenvolvido durante o Nanodegree Desenvolvedor Web Full-Stack oferecido pela Udacity.
