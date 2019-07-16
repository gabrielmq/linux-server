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
sudo mkdir /home/grader/.ssh
sudo chown grader:grader /home/grader/.ssh
sudo touch /home/grader/.ssh/authorized_keys
sudo nano /home/grader/.ssh/authorized_keys #cole o valor da chave.pub
sudo chmod 700 /home/grader/.ssh
sudo chmod 644 /home/grader/.ssh/authorized_keys
```

- Agora já é possível utilizar o comando `ssh -i ~/.ssh/grader_key grader@18.207.168.11` para acessar o servidor com o novo usuário.

# Licensa

Este projeto foi desenvolvido durante o Nanodegree Desenvolvedor Web Full-Stack oferecido pela Udacity.
