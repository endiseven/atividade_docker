# atividade_docker

Documentação da segunda atividade - Docker - AWS - PB COMPASS


## Criar VPC - AWS
- Criar uma VPC selecionando a opção "VPC e muito mais" para a configuração automática de toda a rede.
- Selecionar pelo menos 2 zonas de disponibilidade.

## Grupos de Segurança
- Um grupo de segurança padrão "default" foi configurado para garantir a comunicação entre os diversos serviços (RDS, EFS, LoadBalancer e EC2).
	- Abrir Porta ```22``` para acesso SSH e porta ```80``` para HTTP.
 	- Liberar Todo tráfego em todas as portas de qualquer protocolo entre os membros do grupo de segurança.  	  
  
## Criar EFS
- No serviço EFS, Selecionar "Criar sistema de arquivos"
  
## Criar instancia EC2 para configuração inicial
- Adicionar as Tags do recurso.
- Configurar o EFS automaticamente no lançamento da instância
	- Em "configurar armazenamento" clicar em "avançado".
 	- Selecionar "Adicionar sistema de arquivos compartilhado" e definir o ponto de montagem:
    ```/mnt/efs/fs1```
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

- Após instância ser lançada verificar se o EFS foi montado corretamente.

```
df - T
```

 - Verificar se o docker e o docker-compose foi instalado com sucesso.
```
docker --version
docker-compose --version
```
  
## Criar RDS
- Configurar manualmente, selecionar uma instância para conectar (security groups necessários serão criados automaticamente).
- Criar banco de dados inicial e nomea-lo "wordpress".
- Após a criação do RDS, copiar o "endpoint" para configuração do container Wordpress.
  
## Na Instância EC2:
- Criar pasta "wordpress" no EFS.
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
- Iniciar container docker com comando:
```  
docker-compose up -d
```
- Verificar instalação:

```
docker ps -a
```
  
## Load Balancer
- Criar e Configurar load balancer e o target group.
	- Tipo Aplication Load Balancer
	- Esquema: Voltado para internet
 	- Grupo de segurança: "Default", configurado no início da atividade. 		 	
	- Registrar a instância como destino no Target Group.
  
## Configuração wordpress
- Acessar o DNS do load balancer:
```
http://lbwordpress2-1659449724.us-east-1.elb.amazonaws.com/
```
- Realizar configuração inicial do Wordpress.
  
## Criar AMI da instancia

## Criar modelo de execução usando a AMI
- Lembrar das Tags de recurso.
- Manter os grupos de segurança.
  
## Auto-Scaling
- Selecionar o modelo de execução.
- Configurar o auto scaling group.
- Definir load balancer e target groups.
- Min 2 instancias, duas zonas de disponibilidade

## Acesso ao Wordpress

- A página de login do wordpress pode ser acessada pelo DNS do load balancer acrescido de ```/login/``` ou ```/wp-admin/```.
```
http://lbwordpress2-1659449724.us-east-1.elb.amazonaws.com/login/
```

