

# Documentação da criação de um Ambiente de Aplicação em Docker usando AWS

Instruções detalhadas para configurar um ambiente de aplicação em Docker na AWS, incluindo a criação de uma instância EC2, configuração do Docker, criação de um banco de dados RDS, configuração de um sistema de arquivos elástico (EFS), configuração do Docker Compose, montagem do EFS e configuração do Auto Scaling Group com um Load Balancer.

## Passo 1: Criação da Instância EC2

1. Acesse o Console AWS em [https://aws.amazon.com/console/](https://aws.amazon.com/console/).
2. Vá para o serviço EC2.
3. Clique em "Launch Instance" para criar uma nova instância EC2.
4. Siga o assistente de criação de instâncias, escolhendo uma AMI e tipo de instância.
5. Em "Configure Instance Details", configure as opções de instância conforme necessário.
6. Em "Add Storage", defina o tamanho do disco e outras configurações de armazenamento.
7. Em "Add Tags", adicione tags para facilitar a organização.
8. Em "Configure Security Group", permita o acesso aos portos necessários (por exemplo, 22 para SSH, 80 para HTTP).
9. Revise as configurações e clique em "Launch".
10. Selecione ou crie uma chave de segurança para acessar a instância.

## Passo 2: Instalação do Docker e Docker Compose

1. Conecte-se à instância via SSH.
2. Instale o Docker:

   ```bash
   sudo yum install docker -y
   sudo service docker start
   sudo systemctl enable docker
   ```

3. Instale o Docker Compose:

   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   ```

## Passo 3: Criação do RDS e EFS

1. No Console AWS, vá para o serviço RDS e crie um banco de dados MySQL.
2. No Console AWS, vá para o serviço EFS e crie um sistema de arquivos elástico.

## Passo 4: Criação do Docker Compose

1. Crie um arquivo `docker-compose.yml` com as configurações para o WordPress e o RDS.

## Passo 5: Configuração do WordPress

1. Use o Docker Compose para iniciar o WordPress e conectar ao RDS:

   ```bash
   docker-compose up -d
   ```

## Passo 6: Montagem do EFS

1. Monte o EFS na pasta `/efs`:

   ```bash
   sudo mount -t efs fs-12345678:/ /efs
   ```

## Passo 7: Criação de um Script para User Data

1. Crie um script com os comandos acima (por exemplo, `setup.sh`).
2. Exemplo
   ```bash
      #!/bin/bash
 
      sudo yum update -y
      sudo yum install docker -y
      sudo systemctl start docker.service && systemctl enable docker.service
      usermod -aG docker ec2-user
 
      curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
      chmod +x /usr/local/bin/docker-compose
 
      sudo yum install -y nfs-utils
      sudo mkdir /mnt/efs
      echo "fs-0c681f4886dda0762.efs.us-east-1.amazonaws.com:/ /mnt/efs nfs defaults 0 0" >> /etc/fstab
      mount -a
 
      sudo yum install -y mysql
 
      sudo mkdir /Docker
 
      sudo tee /Docker/docker-compose.yml <<EOF
      version: '3.7'
 
      services:
        wordpress:
          image: wordpress
          restart: always
          ports:
            - 80:80
          environment:
            WORDPRESS_DB_HOST: docker-wordpress.cxgvimz3trpy.us-east-1.rds.amazonaws.com
            WORDPRESS_DB_NAME: docker-wordpress
            WORDPRESS_DB_USER: admin
            WORDPRESS_DB_PASSWORD: admin1234
          volumes: 
            - /mnt/efs:/var/www/html
      EOF
 
      cd /Docker
 
      docker-compose up -d
      ```

## Passo 8: Criação de um Launch Configuration

1. No Console AWS, vá para o serviço EC2 e clique em "Launch Configurations".
2. Crie um novo Launch Configuration e adicione o script de User Data.

## Passo 9: Configuração do Auto Scaling Group

1. No Console AWS, vá para o serviço EC2 e clique em "Auto Scaling Groups".
2. Crie um novo Auto Scaling Group usando o Launch Configuration.

## Passo 10: Configuração do Load Balancer

1. No Console AWS, vá para o serviço EC2 e clique em "Load Balancers".
2. Crie um novo Classic Load Balancer para distribuir o tráfego.

## Pasoo 11: Configuração Banco de Dados

1. Entre na instância.
2. Conecte-se ao banco de dados
   ```bash
   mysql -h (end point do deu RDS) -u root -p
   ```
3. Execute esse comando para visualizar o endereço do Wordpress
   ```bash
   SELECT * FROM wp_options WHERE option_name = 'siteurl' OR option_name = 'home';
   ```
4. Altere o endereço para o DNS do load balancer com esse comando
   ```bash
   UPDATE wpoptions SET option_value = 'http://(DNS do Load Balncer)' WHERE option_name = 'siteurl' OR option_name = 'home'; = 'home';
   ```

## Passo 12: Teste e Verificação

1. Verifique se o ambiente está funcionando corretamente, acessando o DNS público do Load Balancer.

---
