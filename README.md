
# Prerequisitos

- 4 máquinas virtuais com 2/4 processadores e 6/8 gb de memória ram e 30Gb de HD
- 1 domínio
- Sistema operacional Ubuntu 16.04 LTS
- Domínio a ser utilizado: rcic.com.br

[Kubernetes Projeto](https://github.com/robertocorreajr/Kubernetes-Projeto)


Teremos 1 máquina que será o Rancher Server e outras 3 máquinas que serão os Kubernetes de produção.
![](2021-11-01-14-11-23.png)

Vou configurar no domínio para apontar para o DNS do Google que será onde eu vou criar as máquinas virtuais e lá no DNS do google eu vou criar entradas do qual o endereço rancher.rci.com.br vai apontar para a máquina quer será o Rancher Server.

Também vou criar um balanceamento com Wildcard ( * ) que conterá os 3 iPs dos outros servidores.



# Domínio

Após dominio criado modificar as entradas DNS para apontar para o DNS do Google ou o provedor escolhido, como por exemplo os DNS da AWS. Para que possamos fazer essa configuração, vamos acessar o GCP e acessar o serviço Cloud DNS para criar uma nova zona que vai receber o domínio.

![](2021-11-01-14-19-47.png)

![](2021-11-01-14-21-41.png)

Expandir a entrada do registro NS para poder pegar os endereços DNS do google que serão preenchidos no site do Registro Br.
![](2021-11-01-14-25-04.png)
![](2021-11-01-14-31-50.png)

Acessar o site do Registro Br e adicionar as entradas DNS.

![](2021-11-01-14-15-12.png)

# Criação do ambiente

Agora vamos acessar Compute Engine e criar as máquinas virtuais. "VM Instance".
![](2021-11-01-15-13-43.png)

### Configurações da instância.
Do o nome da instância e na guia de configuração da máquina, vou na opção personalisada e escolho 2 à 4 processadores com no minimo 6Gb de memória.
![](2021-11-01-15-20-58.png)

### Boot disk
Também é necessário escolher em Boot disk a imagem do Ubuntu 16.04 TLS e o tamanho do HD. "30Gb"

![](2021-11-01-15-26-04.png)

### Script de criação das instâncias.
Abaixo, segue script para a criação de todas as máquinas pela CLI com 4 processadores, 6Gb de RAM e 30Gb de HD. 
```sh
gcloud compute instances create rancher-kubernetes --project=ordinal-link-321822 --zone=us-west1-b --machine-type=e2-custom-4-6144 --network-interface=network-tier=PREMIUM,subnet=default --maintenance-policy=MIGRATE --service-account=1005677283607-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=rancher-kubernetes,image=projects/ubuntu-os-cloud/global/images/ubuntu-1604-xenial-v20210928,mode=rw,size=30,type=projects/ordinal-link-321822/zones/us-west1-b/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any

gcloud compute instances create k8s-1 --project=ordinal-link-321822 --zone=us-west1-b --machine-type=e2-custom-4-6144 --network-interface=network-tier=PREMIUM,subnet=default --maintenance-policy=MIGRATE --service-account=1005677283607-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=rancher-kubernetes,image=projects/ubuntu-os-cloud/global/images/ubuntu-1604-xenial-v20210928,mode=rw,size=30,type=projects/ordinal-link-321822/zones/us-west1-b/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any

gcloud compute instances create k8s-2 --project=ordinal-link-321822 --zone=us-west1-b --machine-type=e2-custom-4-6144 --network-interface=network-tier=PREMIUM,subnet=default --maintenance-policy=MIGRATE --service-account=1005677283607-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=rancher-kubernetes,image=projects/ubuntu-os-cloud/global/images/ubuntu-1604-xenial-v20210928,mode=rw,size=30,type=projects/ordinal-link-321822/zones/us-west1-b/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any

gcloud compute instances create k8s-3 --project=ordinal-link-321822 --zone=us-west1-b --machine-type=e2-custom-4-6144 --network-interface=network-tier=PREMIUM,subnet=default --maintenance-policy=MIGRATE --service-account=1005677283607-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=rancher-kubernetes,image=projects/ubuntu-os-cloud/global/images/ubuntu-1604-xenial-v20210928,mode=rw,size=30,type=projects/ordinal-link-321822/zones/us-west1-b/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```

### Instâncias criadas:
![](2021-11-01-15-40-50.png)

### Instalação do Docker:

```sh
ssh casa@34.105.100.171     - rancher
ssh casa@35.197.59.9        - k8s-1
ssh casa@34.82.244.41       - k8s-2
ssh casa@35.233.199.77      - k8s-3
```
Vou acessar cada uma das máquinas e instalar o docker e também vou adicionar o usuário ubuntu no groupo docker, após instalado podemos testar executando o comando docker ps.
```sh
sudo su
curl https://releases.rancher.com/install-docker/19.03.sh | sh
usermod -aG docker ubuntu

docker ps
```
![](2021-11-01-15-49-26.png)

### Cloud DNS:
Vou configurar agora as entradas do DNS que vão apontar para o rancher e um balanceador que será configurado com *Wildcard ( * )*

Acessar a zona criada.

![](2021-11-01-15-54-10.png)

### Script de criação das entradas DNS:
```sh
gcloud beta dns --project=ordinal-link-321822 record-sets transaction start --zone="rcic" && gcloud beta dns --project=ordinal-link-321822 record-sets transaction add 34.105.100.171 --name="rancher.rcic.com.br." --ttl="300" --type="A" --zone="rcic" && gcloud beta dns --project=ordinal-link-321822 record-sets transaction execute --zone="rcic"

gcloud beta dns --project=ordinal-link-321822 record-sets transaction start --zone="rcic" && gcloud beta dns --project=ordinal-link-321822 record-sets transaction add 35.197.59.9 34.82.244.41 35.233.199.77 --name="*.rancher.rcic.com.br." --ttl="300" --type="A" --zone="rcic" && gcloud beta dns --project=ordinal-link-321822 record-sets transaction execute --zone="rcic"

```
![](2021-11-01-16-04-08.png)

# Git / Docker Compose / Images builds

### Instalando o GIT
```sh
sudo su
apt-get install git -y
```

### O que é e faz o docker-compose?
Docker Compose é uma ferramenta que foi desenvolvida para ajudar a definir e compartilhar aplicativos de vários contêineres. Com o Compose, você pode criar um arquivo YAML para definir os serviços e, com um único comando, pode girar tudo ou destruir tudo.

A grande vantagem de usar o Compose é que você pode definir sua pilha de aplicativos em um arquivo, mantê-la na raiz do seu repo do projeto (agora ele é controlado por versão) e permitir facilmente que outra pessoa contribua com seu projeto. Alguém só precisaria clonar seu repo e iniciar o aplicativo de composição. Na verdade, você pode ver alguns projetos no GitHub/GitLab fazendo exatamente isso agora.

[Documentação oficial](https://docs.docker.com/compose/install/)

### Baixando o Docker Compose:
```sh
curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

### Aplicando permissão de execução ao binário:
```sh
chmod +x /usr/local/bin/docker-compose
```

### Criando um link simbolico para o executavel.
*( Para windows Users, seria como se criasse um atalho. "Shortcut" )*
```sh
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

Com os pacotes já instalados vou clonar o repositório para o meu servidor.
```sh
cd /home/ubuntu
git clone https://github.com/robertocorreajr/Kubernetes-Projeto
cd devops/exercicios/app
```
<br>
<br>

### Fazendo build de um container REDIS
```sh
cd redis
docker build -t robertocorrea/redis:devops .
docker run -d --name redis -p 6379:6379 robertocorrea/redis:devops
docker ps
docker logs redis
```
Com isso temos o container do Redis rodando na porta 6379.
<br>
<br>

### Fazendo build de um container NODE.
Criando a imagem
```sh
cd ../node
docker build -t robertocorrea/node:devops .
```

Agora iremos rodar a imagem do node, fazendo a ligação dela com o container do Redis.
```sh
docker run -d --name node -p 8080:8080 --link redis robertocorrea/node:devops
docker ps 
docker logs node
```
Com isso temos nossa aplicação rodando, e conectada no Redis. A api para verificação pode ser acessada em /redis.
<br>
<br>

### Container=NGINX
Iremos fazer o build do container do nginx, que será nosso balanceador de carga.
```sh
cd ../nginx
docker build -t robertocorrea/nginx:devops .
```
Criando o container do nginx a partir da imagem e fazendo a ligação com o container do Node
```sh
docker run -d --name nginx -p 80:80 --link node robertocorrea/nginx:devops
docker ps
```
Podemos acessar então nossa aplicação nas portas 80 e 8080 no ip da nossa instância.

![](2021-11-01-23-49-23.png)

Iremos acessar a api em /redis para nos certificar que está tudo ok, e depois iremos limpar todos os containers e volumes.
```sh
docker rm -f $(docker ps -a -q)
docker volume rm $(docker volume ls)
```
<br>
<br>

### Docker Compose

Até agora foi feito um exercício onde foi colocado alguns containers para rodar e eles foram interligando. Foi possivel ver como funciona a aplicação que possui um contador de acessos, mas essa aplicação estava rodando apenas na máquina principal que estava destinada ao rancher. Então uma das ultimas tarefas foi remover os containers que utilizamos e seus volumes. 

Agora vamos preparar para *"buildar"* 

Nesse exercício que fizemos agora, colocamos os containers para rodar, e interligando eles, foi possível observar  como funciona nossa aplicação que tem um contador de acessos.
Para rodar nosso docker-compose, precisamos remover todos os containers que estão rodando e ir na raiz do diretório para rodar.

É preciso editar o arquivo docker-compose.yml, onde estão os nomes das imagens e colocar o seu nome do repositório que você criou.

- Linha 8   = \<dockerhub-user>/nginx:devops
- Linha 16  = image: \<dockerhub-user>/redis:devops
- Linha 33  = image: \<dockerhub-user>/node:devops

<br>
*Abaixo o arquivo já editado*
<br>

```yml
# Versão 2 do Docker-Compose
version: '2'

services:
    
    nginx:
        restart: "always"
        image: robertocorrea/nginx:devops
        ports:
            - "80:80"
        links:
            - node
            
    redis:
        restart: "always"
        image: robertocorrea/redis:devops
        ports:
            - 6379

    mysql:
        restart: "always"
        image: mysql
        ports:
            - 3306
        environment:
            MYSQL_ROOT_PASSWORD: 123
            MYSQL_DATABASE: books
            MYSQL_USER: apitreinamento
            MYSQL_PASSWORD: 123 

    node:
        restart: "always"
        image: robertocorrea/node:devops
        links:
            - redis
            - mysql
        ports:
            - 8080
        volumes:
            -  volumeteste:/tmp/volumeteste

# Mapeamento dos volumes
volumes:
    volumeteste:
        external: false
```
<br>

**OBS:** *Note que quando removemos os container, mantivemos a imagem dos container e será essas imagens que vamos utilizar nesse build.*
<br>

![](2021-11-01-23-43-33.png)

Após alterar e colocar o nome correto das imagens, rodar o comando de *up -d* para subir toda a stack.

```sh
cd ..
vi docker-compose.yml
docker-compose -f docker-compose.yml up -d
curl rancher.rcic.com.br:80
```

```sh
curl rancher.rcic.com.br:80/redis
	----------------------------------
	This page has been viewed 29 times
	----------------------------------
```

Se acessarmos o IP na porta 80 vamos ver a aplicação rodando e também podemos acessar o /redis para ver o contador de acesso à essa página incrementando a cada acesso.
![](2021-11-01-23-52-14.png)

Explicação: Toda essa aplicação foi *"deployada"*  de uma só vez com um unico arquivo docker *docker file*. Ele criou todo um deploy de aplicação, totalmente padronizado e com todas as suas dependências satisfeitas.

Agora para finalizar vamos terminar a nossa aplicação temos que rodar o comando do docker-compose abaixo:
```sh
docker-compose down
```
![](2021-11-01-23-59-09.png)