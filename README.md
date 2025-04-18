# Implementação de Microsserviços com Docker Swarm, MySQL e Load Balancer com proxy Nginx
Criação de uma infraestrutura de microsserviços utilizando Docker Swarm, MySQL e um balanceador de carga com proxy Nginx.  
Aqui estão os principais passos:

1. Provisionamento de VMs
2. Instalação do Docker
3. Configuração do MySQL: Volumes são criados e um container MySQL é executado, configurado com um banco de dados específico.
4. Conexão ao Banco de Dados: Ferramentas como MySQL Workbench são usadas para conectar e gerenciar o banco de dados.
5. Criação de Tabelas: Uma tabela chamada dados é criada no banco de dados.
6. Instalação do Webserver: Um container Apache com PHP é configurado para servir a aplicação web.
7. Teste da Aplicação: A aplicação é acessada via navegador para verificar seu funcionamento.
8. Estresse do Container: Ferramentas como Loader.io são usadas para testar a capacidade de resposta do container.
9. Criação do Docker Swarm: Um cluster Docker Swarm é inicializado, com um nó mestre e nós trabalhadores.
10. Replicação do Webserver: Réplicas do webserver são criadas para garantir alta disponibilidade.
11. Montagem de Volumes: Volumes são montados no cluster para compartilhamento de dados entre containers.
12. Estresse do Container: Novamente, Loader.io é utilizado para testar a escalabilidade da aplicação.

A seguir o passo a passo detalhado.

01- PROVISIONAR 3 VMs AWS EC2 free tier (t2.micro/ubuntu)
---------------------------------------------------------
Três instâncias EC2 na AWS são configuradas com Ubuntu, liberando portas essenciais no Security Group.  
![image](https://github.com/user-attachments/assets/2a00134a-8b0b-4613-8725-098d43fb1916)

A fim de que todo o processo funcione é preciso liberar as seguintes portas no Secury Group através de suas inbound rules:
- 3306
- 80
- 22
- 2377
- 4500
- All ports for the Security Group

![image](https://github.com/user-attachments/assets/22c68c64-33ff-4522-8555-9d63cf84f8b7)

02- INSTALAR DOCKER NAS 3 VMs EC2
---------------------------------
Docker é instalado nas três VMs para permitir a execução de containers.  
~~~
curl -fsSL https://get.docker.com -o get-docker.sh  
sudo sh ./get-docker.sh  
chmod 666 /var/run/docker.sock  
~~~

03- CRIAR VOLUMES APP e DATA, E EXECUTAR CONTAINER MYSQL
--------------------------------------------------------
Volumes são criados e um container MySQL é executado, configurado com um banco de dados específico.  
~~~
docker volume create app  
volume create data  
docker run -e MYSQL_ROOT_PASSWORD=Senha123 -e MYSQL_DATABASE=meubanco --name mysql-A -d -p 3306:3306 --mount type=volume,src=data,dst=/var/lib/mysql/ mysql:5.7  
~~~
04- CONECTAR AO BD meubanco
---------------------------
Ferramentas como MySQL Workbench são usadas para conectar e gerenciar o banco de dados.  

![image](https://github.com/user-attachments/assets/68840bb0-83e4-4f97-8186-226d47e5fc59)

05- CRIAR TABELA dados
----------------------
Uma tabela chamada dados é criada no banco de dados.  
~~~
CREATE TABLE dados (  
    AlunoID int,  
    Nome varchar(50),  
    Sobrenome varchar(50),  
    Endereco varchar(150),  
    Cidade varchar(50),  
    Host varchar(50)  
);  
~~~
![image](https://github.com/user-attachments/assets/c0028a93-994e-4245-98a0-4a7fdc05b747)

06- INSTALAR CONTAINER WEBSERVER APACHE COM PHP
-----------------------------------------------
Um container Apache com PHP é configurado para servir a aplicação web.  
~~~
docker run --name web-server -dt -p 80:80 --mount type=volume,src=app,dst=/app/ webdevops/php-apache:alpine-php7
~~~
![image](https://github.com/user-attachments/assets/9dee0e2b-5875-4f76-bc0d-7239f9f6c89e)

07- TESTAR A APLICAÇÃO
----------------------
A aplicação é acessada via navegador para verificar seu funcionamento.  
Acesse o browser com o endereço: `http://<PUBLIC IP>`

08- EXTRESSAR O CONTAINER
-------------------------
Ferramentas como loader.io são usadas para testar a capacidade de resposta do container.  
8.1- Acesse `https://loader.io`  
8.2- Defina o Target host  
8.3- Faça a Target verification  
![image](https://github.com/user-attachments/assets/285c72a7-05d6-47ca-9d5d-dec90cdf1af5)
8.4- Copie o verification token e use-o para criar o arquivo no diretório app
![image](https://github.com/user-attachments/assets/e4627356-d3a5-4fd3-b531-f4954868ad67)

09- CRIAR SWARM DOCKER
----------------------
Um cluster Docker Swarm é inicializado, com um nó mestre e nós worker.  
9.1- NO MASTER: `docker swarm init`
![image](https://github.com/user-attachments/assets/cd740f2e-bce8-41fb-addd-ab3f4f0cec23)

9.2- NOS WORKERS: comando `docker join` fornecido pelo `docker swarm init`
![image](https://github.com/user-attachments/assets/e1791d37-a62d-45b2-8308-86d1cd8909ac)

10- CRIAR RÉPLICAS DO WEBSERVER APACHE COM PHP
----------------------------------------------
Réplicas do webserver são criadas para garantir alta disponibilidade.  
NO MASTER:  
~~~
docker service create --name web-server --replicas 10 -dt -p 80:80 --mount type=volume,src=app,dst=/app/ webdevops/php-apache:alpine-php7
~~~
![image](https://github.com/user-attachments/assets/a99c86cc-ea08-45a8-af42-eafcc9981129)

11- MONTAR O VOLUME NO CLUSTER
------------------------------
Volumes são montados no cluster para compartilhamento de dados entre containers.  
11.1- NO MASTER:   `apt-get install nfs-server`
![image](https://github.com/user-attachments/assets/f82c7cd4-2558-4223-8990-28a963c3d0ce)

11.2- NOS WORKERS: `apt-get install nfs-common`
![image](https://github.com/user-attachments/assets/975331a8-954c-418c-8ad3-e46a5e16436a)

11.3- NO MASTER:   `nano /etc/exports`
		               `/var/lib/docker/volumes/app/ *(rw,sync,subtree_check)`
![image](https://github.com/user-attachments/assets/7b6a1931-2921-4cd2-8f8b-73c93e6965fb)

11.4- NO MASTER:   
~~~
exportfs -ar
showmount -e
~~~
11.5- NOS WORKERS: `mount -o v3 <IP LEADER>:/var/lib/docker/volumes/app/_data /var/lib/docker/volumes/app/_data`

12- EXTRESSAR O CONTAINER
-------------------------
Novamente, Loader.io é utilizado para testar a escalabilidade da aplicação.  
12.1- Acesse `https://loader.io`
12.2- Defina o Target host
12.3- Faça a Target verification
![image](https://github.com/user-attachments/assets/1b2a148f-12b7-4076-9b37-475d38f29db3)
12.4- Copie o verification token e use-o para criar o arquivo no diretório app
![image](https://github.com/user-attachments/assets/2b0b4239-5dc6-49de-bfc5-bd202a8e2f33)
12.5- Defina o teste
![image](https://github.com/user-attachments/assets/034e5ebc-b40e-407e-877b-6e866f9843fb)
![image](https://github.com/user-attachments/assets/fbee78cf-4d1e-4862-8fc2-06433577a044)


