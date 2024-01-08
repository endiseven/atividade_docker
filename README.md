# atividade_docker

Documentação da segunda atividade - Docker - AWS - PB COMPASS

## Criar VPC - AWS
- Criar uma VPC selecionando a opção "VPC e muito mais" para a configuração automática de toda a rede.
- Selecionar pelo menos 2 zonas de disponibilidade.
  
## Criar EFS
- No serviço EFS, Selecionar "Criar sistema de arquivos"
  
## Criar instancia EC2 para configuração inicial
- Adicionar as Tags do recurso.
- Configurar o EFS automáticamente no lançamento da instância
	- Em "configurar armazenamento" clicar em "avançado".
 	- Selecionar "Adicionar sistema de arquivos compartilhado" e definir o ponto de montagem.
  	- Mantenha as opções "Criar e anexar grupos de segurança automaticamente" e "Montar automaticamente o sistema de arquivos..." selecionadas.
- Configurar o Docker e o docker-compose no user data    		 
- Editar "User data" para utilizar dois tipos de script, cloud-config e bash
	- Copiar o script de "cloud-config" e formatar de acordo com o modelo abaixo. 
```
Content-Type: multipart/mixed; boundary="//"
MIME-Version: 1.0

--//
Content-Type: text/cloud-config; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="cloud-config.txt"

INSERIR O CONTEÚDO DE DADOS DO USUÁRIO AQUI

--//
Content-Type: text/x-shellscript; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="userdata.txt"

#!/bin/bash
yum update -y
yum install -y docker
service docker start
usermod -a -G docker ec2-user
systemctl enable docker

curl -L https://github.com/docker/compose/releases/download/1.21.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
--//--

```

- Após instancia ser lançada verificar se o EFS foi montado corretamente e se o docker foi instalado com sucesso
  
## Criar RDS
- Configurar manualmente, selecionar instancia para conectar (security groups serão criados)
- Criar database inicial e nomea-la "wordpress"
- Copiar "endpoint" para configuração do container wordpress
  
## Na Instância EC2:
- Criar pasta "wordpress" no EFS
- Escrever o arquivo docker-compose.yml para subir o container de Wordpress e configurar:
	- Enviroment: Com as informações do host mysql criado com o RDS
	- Volumes: com a localização no ponto de montagem do EFS na instancia.
```
version: '3.1'

services:

  wordpress:
    image: wordpress
    restart: always
    ports:
      - 80:80
    environment:
      WORDPRESS_DB_HOST: wordpress.xxxxxxxxx.us-east-1.rds.amazonaws.com
      WORDPRESS_DB_USER: admin
      WORDPRESS_DB_PASSWORD: xxxxxxxx
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - /mnt/efs/fs1/wordpress:/var/www/html
```

- docker-compose up -d
- Verificar instalação
  
## Load Balancer
- Configurar load balancer e o target group
  
## Configuração wordpress:
- Acessar o DNS do load balancer
- Realizar configuração inicial do Wordpress
  
## Criar AMI da instancia
## Criar modelo de execução usando a AMI
## Auto-Scaling
- configurar o auto scaling group 
- Definir load balancer e target groups
- Min 2 instancias, duas zonas de disponibilidade

