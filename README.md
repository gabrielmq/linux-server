# Configuração Servidor Linux

Este projeto é um tutorial para configurar um servidor remoto hospedado na AWS [Lightsail](https://aws.amazon.com/pt/lightsail/), fazendo com que este servidor remoto disponibilize uma aplicação [web](https://github.com/gabrielmq/catalogo-itens) desenvolvida em python.

## Passo 1 - Acessando o servidor via SSH

- Faça o download da chave SSH localizada nas configurações de conta da Amazon Lightsail.
- Após realizar o download, mova o arquivo para o diretório `.ssh`.
  - Exemplo: `sudo mv ~/Downloads/lightsail-key.pem ~/.ssh`.
- No prompt de comandos execute o comando `sudo chmod 600 ~/.ssh/lightsail-key.pem`
- Ainda no prompt de comandos execute o comando `ssh -i ~/.ssh/lightsail-key.pem ubuntu@18.212.212.133` para se conectar ao servidor.

## Passo 2 - Atualizando os pacotes do servidor e configurando time zone UTC

```
   sudo apt-get update
   sudo apt-get upgrade
   sudo apt-get dist-upgrade
   sudo timedatectl set-timezone UTC
```

## Passo 3 - Configurando usuário grader

- Execute os comandos abaixo para criar o usuário grader

```
sudo adduser grader
sudo usermod -a -G sudo grader
```

## Passo 4 - Configurando chaves SSH para usuário grader

- Gere as chaves ssh localmente executando o comando `ssh-keygen` e salve no diretório `~/.ssh`
- Configure a chave pública no servidor remoto

```
su - grader # para mudar de usuário
sudo mkdir .ssh/
sudo chown grader:grader .ssh/
sudo touch .ssh/authorized_keys
sudo nano .ssh/authorized_keys #cole o valor da chave.pub
sudo chmod 700 .ssh
sudo chmod 644 .ssh/authorized_keys
```

- Agora já é possível utilizar o comando `ssh -i ~/.ssh/grader_key grader@18.207.168.11` para acessar o servidor com o novo usuário.

## Passo 5 - Removendo acesso para usuário root

- Execute o comando `sudo nano /etc/ssh/sshd_config`
- Altere a linha `PermitRootLogin prohibit-password` para `PermitRootLogin no`
- Altere a linha `PasswordAuthentication yes` para  `PasswordAuthentication no`
- Adicione `DenyUsers root`

## Passo 6 - Alterar porta SSH de 22 para 2200  

- Nas configurações de rede do servidor na AWS LightSail, configure uma porta personalizada no Firewall como: `Custom/TCP 2200`
- Remova a configuração padrão de SSH
- Conectado no servidor remoto pelo prompt execute o comando abaixo e altere a porta de 22 para 2200

```
sudo nano /etc/ssh/sshd_config
```

- Execute o comando `sudo service ssh restart` para reinicializar o serviço.

## Passo 7 - Configurando Firewall

```
sudo ufw status # verifica o estado do firewall
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www 
sudo ufw allow ntp
sudo ufw enable # habilita o firewall
sudo ufw status
```

# Licensa

Este projeto foi desenvolvido durante o Nanodegree Desenvolvedor Web Full-Stack oferecido pela Udacity.
