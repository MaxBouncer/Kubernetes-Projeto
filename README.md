
# Prerequisitos

- 4 máquinas virtuais com 2/4 processadores e 6/8 gb de memória ram e 30Gb de HD
- 1 domínio
- Sistema operacional Ubuntu 16.04 LTS
- Domínio a ser utilizado: rcic.com.br

[Kubernetes Projeto](https://github.com/robertocorreajr/Kubernetes-Projeto)


Teremos 1 máquina que será o Rancher Server e outras 3 máquinas que serão os Kubernetes de produção.
![](img/2021-11-01-14-11-23.png)

Vou configurar no domínio para apontar para o DNS do Google que será onde eu vou criar as máquinas virtuais e lá no DNS do google eu vou criar entradas do qual o endereço rancher.rci.com.br vai apontar para a máquina quer será o Rancher Server.

Também vou criar um balanceamento com Wildcard ( * ) que conterá os 3 iPs dos outros servidores.



# Domínio

Após dominio criado modificar as entradas DNS para apontar para o DNS do Google ou o provedor escolhido, como por exemplo os DNS da AWS. Para que possamos fazer essa configuração, vamos acessar o GCP e acessar o serviço Cloud DNS para criar uma nova zona que vai receber o domínio.

![](img/2021-11-01-14-19-47.png)

![](img/2021-11-01-14-21-41.png)

Expandir a entrada do registro NS para poder pegar os endereços DNS do google que serão preenchidos no site do Registro Br.
![](img/2021-11-01-14-25-04.png)
![](img/2021-11-01-14-31-50.png)

Acessar o site do Registro Br e adicionar as entradas DNS.

![](img/2021-11-01-14-15-12.png)

# Criação do ambiente

Agora vamos acessar Compute Engine e criar as máquinas virtuais. "VM Instance".
![](img/2021-11-01-15-13-43.png)

### Configurações da instância.
Do o nome da instância e na guia de configuração da máquina, vou na opção personalisada e escolho 2 à 4 processadores com no minimo 6Gb de memória.
![](img/2021-11-01-15-20-58.png)

### Boot disk
Também é necessário escolher em Boot disk a imagem do Ubuntu 16.04 TLS e o tamanho do HD. "30Gb"

![](img/2021-11-01-15-26-04.png)

### Script de criação das instâncias.
Abaixo, segue script para a criação de todas as máquinas pela CLI com 4 processadores, 6Gb de RAM e 30Gb de HD. 
```sh
gcloud compute instances create rancher-kubernetes --project=ordinal-link-321822 --zone=us-west1-b --machine-type=e2-custom-4-6144 --network-interface=network-tier=PREMIUM,subnet=default --maintenance-policy=MIGRATE --service-account=1005677283607-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=rancher-kubernetes,image=projects/ubuntu-os-cloud/global/images/ubuntu-1604-xenial-v20210928,mode=rw,size=30,type=projects/ordinal-link-321822/zones/us-west1-b/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any

gcloud compute instances create k8s-1 --project=ordinal-link-321822 --zone=us-west1-b --machine-type=e2-custom-4-6144 --network-interface=network-tier=PREMIUM,subnet=default --maintenance-policy=MIGRATE --service-account=1005677283607-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=rancher-kubernetes,image=projects/ubuntu-os-cloud/global/images/ubuntu-1604-xenial-v20210928,mode=rw,size=30,type=projects/ordinal-link-321822/zones/us-west1-b/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any

gcloud compute instances create k8s-2 --project=ordinal-link-321822 --zone=us-west1-b --machine-type=e2-custom-4-6144 --network-interface=network-tier=PREMIUM,subnet=default --maintenance-policy=MIGRATE --service-account=1005677283607-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=rancher-kubernetes,image=projects/ubuntu-os-cloud/global/images/ubuntu-1604-xenial-v20210928,mode=rw,size=30,type=projects/ordinal-link-321822/zones/us-west1-b/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any

gcloud compute instances create k8s-3 --project=ordinal-link-321822 --zone=us-west1-b --machine-type=e2-custom-4-6144 --network-interface=network-tier=PREMIUM,subnet=default --maintenance-policy=MIGRATE --service-account=1005677283607-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=rancher-kubernetes,image=projects/ubuntu-os-cloud/global/images/ubuntu-1604-xenial-v20210928,mode=rw,size=30,type=projects/ordinal-link-321822/zones/us-west1-b/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```

### Instâncias criadas:
![](img/2021-11-01-15-40-50.png)

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
![](img/2021-11-01-15-49-26.png)

### Cloud DNS:
Vou configurar agora as entradas do DNS que vão apontar para o rancher e um balanceador que será configurado com *Wildcard ( * )*

Acessar a zona criada.

![](img/2021-11-01-15-54-10.png)

### Script de criação das entradas DNS:
```sh
gcloud beta dns --project=ordinal-link-321822 record-sets transaction start --zone="rcic" && gcloud beta dns --project=ordinal-link-321822 record-sets transaction add 34.105.100.171 --name="rancher.rcic.com.br." --ttl="300" --type="A" --zone="rcic" && gcloud beta dns --project=ordinal-link-321822 record-sets transaction execute --zone="rcic"

gcloud beta dns --project=ordinal-link-321822 record-sets transaction start --zone="rcic" && gcloud beta dns --project=ordinal-link-321822 record-sets transaction add 35.197.59.9 34.82.244.41 35.233.199.77 --name="*.rancher.rcic.com.br." --ttl="300" --type="A" --zone="rcic" && gcloud beta dns --project=ordinal-link-321822 record-sets transaction execute --zone="rcic"

```
![](img/2021-11-01-16-04-08.png)

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

![](img/2021-11-01-23-49-23.png)

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

![](img/2021-11-01-23-43-33.png)

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
![](img/2021-11-01-23-52-14.png)

Explicação: Toda essa aplicação foi *"deployada"*  de uma só vez com um unico arquivo docker *docker file*. Ele criou todo um deploy de aplicação, totalmente padronizado e com todas as suas dependências satisfeitas.

Agora para finalizar vamos terminar a nossa aplicação temos que rodar o comando do docker-compose abaixo:
```sh
docker-compose down
```
![](img/2021-11-01-23-59-09.png)

<br>
<br>

# Rancher - Single Node
[Rancher](https://rancher.com/)


O Rancher permite que tenhamos vários cluster Kubernetes gerenciados pelo Rancher, como por exemplo um Cluster Kubernets para Desenvolvimento, outro para Produção, Testes etc...
Por padrão o Kubernetes não tem a autenticação LDAP para os usuários, então o Rancher pode cuidar dessa autenticação e não precisaremos mais nos preocupar com essa integração do Kubernetes. Podemos configurar o Rancher para rodar inclusive Multi-Cloud.

![](img/2021-11-02-00-19-29.png)

![](img/2021-11-02-00-32-21.png)

https://www.delltechnologies.com/asset/nl-nl/products/converged-infrastructure/industry-market/rancher-cluster-with-vxflex-csi-000066.pdf

Vamos iniciar realizando a instalação do Rancher Single Node.
Nesse exemplo será instalado o Rancher 2.4.3. Primeiramente vou acessar a máquina que será utilizada para o Rancher Server e executar o comando abaixo:
```sh
docker run -d --name rancher --restart=unless-stopped -v /opt/rancher:/var/lib/rancher  -p 80:80 -p 443:443 rancher/rancher:v2.4.3
```
![](img/2021-11-02-01-05-10.png)

> Observe que no comando temos a opção `-v /opt/rancher:/var/lib/rancher` que está criando um volume `/opt/rancher` que é no próprio disco da máquina e dentro do container ele será mapeando para o diretório `/var/lib/rancher`.

Dessa forma se o container morrer, ao ser recriado ele voltará ao normal sem nenhum problema, já que seus dados estarão salvos em outro local.

*Abaixo deixo guardado o usuário e senha dessa instalação. Provavelmente se você estiver lendo esse arquivo e tentar utilizar essas credenciais elas não funcionarão mais ou talvez nem o ambiente esteja mais online, visto que eu o criei apenas para fins de estudos.*
```sh
usuário: admin
password: YdsLeqxFGk5Dud9
```

> Um adendo que quero fazer é sobre a parte de segurança e autenticação. No Rancher temos já integrado vários tipos de autenticação disponíveis. Então não precisariamos lidar com esse tipo de problema no Kubernetes uma vez que isso será feito pelo Rancher.
> ![](img/2021-11-02-01-12-28.png)
<br>
<br>

# Instalação do Cluster Kubernetes
Para criar o cluster, vamos acessar o Rancer e em *Add Cluster*.
![](img/2021-11-02-01-17-47.png)

Vamos escolher a opção *From existing nodes (Custom)*
![](img/2021-11-02-01-19-12.png)

Em nosso exemplo, vamos mudar poucas coisas. 
Primeiramente vamos dar um nome, no meu caso eu dei o nome de *padawan*.
![](img/2021-11-02-01-23-17.png)

Mais abaixo tem uma opção importante que não pode deixar de ser marcada ou nesse caso desabilitada. Que é o Nginx Ingress. 
> Por padrão o Rancher já instala o Nginx Ingress por padrão, mas vamos deixar desabilitado nesse exemplo para podermos utilizar o Traefic e mais para frente vamos executar utilizando o Nginx Ingress portanto, calma jovem Padawan.
![](img/2021-11-02-01-23-56.png)

Após clicar em Next seremos direcionados para a próxima tela de configuração onde vamos ter o script que executaremos nas 3 máquinas que vão compor o Cluster Kubernetes. Para que elas tenham as mesmas funções vamos selecionar as 3 opções etcd, Control Plane e Worker. Também vou expandir a guia advanced options para poder escolher um node name que vou chamar o primeiro de k8s-1, vou copiar o script e alterar para na segunda máquina executar como k8s-2 e na terceira de k8s3. Observe também que essas opções foram adicionadas ao final da linha de comando.
![](img/2021-11-02-01-37-33.png)

Segue abaixo o script gerado:

Servidor 1
```sh
sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.4.3 --server https://rancher.rcic.com.br --token tgcjqs772cgg5kk7fzwq59dsffdxnxrnng7c2hg6g2r7d4b2zfcrm6 --ca-checksum e622c086cf27d07232f97ab963a09e37bded73fbb33abd7000029ede94e50550 --node-name k8s-1 --etcd --controlplane --worker
```

Servidor 2
```sh
sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.4.3 --server https://rancher.rcic.com.br --token tgcjqs772cgg5kk7fzwq59dsffdxnxrnng7c2hg6g2r7d4b2zfcrm6 --ca-checksum e622c086cf27d07232f97ab963a09e37bded73fbb33abd7000029ede94e50550 --node-name k8s-2 --etcd --controlplane --worker
```

Servidor 3
```sh
sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.4.3 --server https://rancher.rcic.com.br --token tgcjqs772cgg5kk7fzwq59dsffdxnxrnng7c2hg6g2r7d4b2zfcrm6 --ca-checksum e622c086cf27d07232f97ab963a09e37bded73fbb33abd7000029ede94e50550 --node-name k8s-3 --etcd --controlplane --worker
```
Vamos entrar em cada uma das máquinas destinadas à membros do cluster e realizar o deploy do Kubernetes, cada um com o seu respectivo script.

Após a execução do Script o Rancher vai se encarregar de fazer todo o deploy nas máquinas. Podemos observer o status indo na interface do rancher e acessar o pelo nome do cluster e posteriormente nodes.
![](img/2021-11-02-01-44-56.png)
![](img/2021-11-02-01-45-26.png)

Devemos aguardar até o termino de todo o deploy, uma vez que ele da algumas mensagens de erro e volta a tentar realizar o deploy novamente até que tudo esteja realmente instalado e finalizado.
![](img/2021-11-02-01-52-13.png)
<br>
<br>

# Instalação do Kubectl
O que é o kubctl?
É a **`CLI`** - *`"comand line interface"`* do Kubernetes e é atravez dela que vamos interagir com o Cluster. 
Vamos precisar instalar ela apenas no servidor do Rancher Server, para isso vou acessar a máquina via SSH e executar a instalação abaixo:
> OBS: executar cada linha separadamente.
> ```sh
> apt-get update && apt-get install -y apt-transport-https gnupg2
> ```
> ```sh
> curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
> ```
> ```sh
> echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | tee -a /etc/apt/sources.list.d/kubernetes.list
> ```
> ```sh
> apt-get update
> ```
> ```sh
> apt-get install -y kubectl
> ```

Com essa instalação ao executar comandos pelo Kubectl ele vai disparar os comando para ele mesmo (Rancher) e ele vai encaminhar os comandos para o Cluster Kubernetes que estão agregados. Então agora todas as execuções serão feitas por arquivos *`yml`*

Bom após a instalação do Kubectl vamos acessar novamente à página do rancher e pegar o arquivo Kubeconfig File para rodar no servidor Rancher do Cluster.
![](img/2021-11-02-02-21-25.png)

Vai abrir um script que já informa que ele deve ser salvo no caminho: ~/.kube/config
![](img/2021-11-02-02-22-39.png)

Como estamos utilizando o usuário ubuntu para rodar o Rancher, será necessário utilizar esse usuário para criar o arquivo ou criar como root e mudar o dono do arquivo para o ubuntu. No meu caso eu vou utilizar diretamente o usuário ubuntu.

Primeiro eu vou criar o diretório ~/.kube e depois o arquivo config para aí colar o script abaixo:
```yml
apiVersion: v1
kind: Config
clusters:
- name: "padawan"
  cluster:
    server: "https://rancher.rcic.com.br/k8s/clusters/c-89zvp"
    certificate-authority-data: "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJpVENDQ\
      VM2Z0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQTdNUnd3R2dZRFZRUUtFeE5rZVc1aGJXbGoKY\
      kdsemRHVnVaWEl0YjNKbk1Sc3dHUVlEVlFRREV4SmtlVzVoYldsamJHbHpkR1Z1WlhJdFkyRXdIa\
      GNOTWpFeApNVEF5TURRd05EUXhXaGNOTXpFeE1ETXhNRFF3TkRReFdqQTdNUnd3R2dZRFZRUUtFe\
      E5rZVc1aGJXbGpiR2x6CmRHVnVaWEl0YjNKbk1Sc3dHUVlEVlFRREV4SmtlVzVoYldsamJHbHpkR\
      1Z1WlhJdFkyRXdXVEFUQmdjcWhrak8KUFFJQkJnZ3Foa2pPUFFNQkJ3TkNBQVJicC9GQ0xNSHpUN\
      zNtWVJCc0xCUFBiT2wrVUw2ZGk5YlBEM0orQW5WVApjMWNST3hhVE91NE03S3kyMTdiQUYrblJhL\
      zFLakdEblFaMG4vcmFVQ2hKQ295TXdJVEFPQmdOVkhROEJBZjhFCkJBTUNBcVF3RHdZRFZSMFRBU\
      UgvQkFVd0F3RUIvekFLQmdncWhrak9QUVFEQWdOSkFEQkdBaUVBdHNEZDByR1cKb1NHWlFFR050L\
      zhzWHFtZEhOOUJWNm9Pa1FTaFZtUTFvWTBDSVFDM3IvUWxtckNuR2RQTEFQU212UlFOelpCTAphU\
      Eg3b0VwKzl6UHNPTEJ1N1E9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0t"
- name: "padawan-k8s-1"
  cluster:
    server: "https://10.138.0.3:6443"
    certificate-authority-data: "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN3akNDQ\
      WFxZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFTTVJBd0RnWURWUVFERXdkcmRXSmwKT\
      FdOaE1CNFhEVEl4TVRFd01qQTBOREV4TUZvWERUTXhNVEF6TVRBME5ERXhNRm93RWpFUU1BNEdBM\
      VVFQXhNSAphM1ZpWlMxallUQ0NBU0l3RFFZSktvWklodmNOQVFFQkJRQURnZ0VQQURDQ0FRb0NnZ\
      0VCQVBBQ1o2SW4xMW5sCmFUQjBwTEtHSlZkV3BrUDFyN3dSbG9VbENhR3ZvOGZic1g3cVMrL05jT\
      0RzbnBNQ1RDMGtzTGpkam42UTVQMU8KMXF0Ui8rT2hzVHdaUTAwYzJUNm9jSUtNNmlUSnUzTzQxe\
      TlKN2MvWXV1VmlNdWlkYXJZNWx0WUpzRER4dEh4dAoyUndzeFh3ZjcvSEt3RXBpQXo3WS83MGF6d\
      TVYRVJ4Y3ZmMjJ1SGJURnJ3REdnYllRTnhsZVREWngwVjc2eHd6CkxpTVlqU2JwT0l6eThwSTFFV\
      zZMSUtYNU84WEVGbndUZlBKaGJ1OHF2NjVYS3ZTQytCQjBXWk5vS3lEUFJBcEMKOGNpbytINGt3R\
      XJTYjZIcENjaVBmWWl2bzJYUGQxSkRtMXladHdMcXZOTVlXaWZydm4rNzdCN2xmaEVLRlpQWAptK\
      09nTEg0U2M2TUNBd0VBQWFNak1DRXdEZ1lEVlIwUEFRSC9CQVFEQWdLa01BOEdBMVVkRXdFQi93U\
      UZNQU1CCkFmOHdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBQXZZZG1pYnozRkpSVzBwdmxYWEJxe\
      WJ5ZjVMRXpGNytKTEoKQWsvZjYzSTdxMEdudzB0cjVEL0QxT3BJQjFkdDJsNnQwNGU5cHhiR0dXd\
      mJpbzJnUHdyK2ZXZ0hrbTZEbkt6Ywp4dHRDaHNlNnRrL3Zubng2V1VyZlIwVEtkZzB2QldBUXBoc\
      m5KTHB6SjJZQjJlKzRJU3hMc3cyU25hWW11Vzk4CnpJb2E1bG5zaksyczlIYlFwRVBibjNLMHNia\
      1ZsWFFieEZESkRLb0c2czFWV3FrcGJrVDFJbU8xNHdRUnA4MGIKSEY1QUpJZS9NOTBSejlqUVJrV\
      HF2RHhtcmpSdmxKNEgwVTM4UE41M28xVExsK1VuWlZ5U0VCTVRmemUwcE9jWApwbm9xNHE5V3ZpU\
      FlmM1FPbVV1NDdaelZERVBBc3JBTDdHWnkxL0dIWUJPWW0zOG4wN2s9Ci0tLS0tRU5EIENFUlRJR\
      klDQVRFLS0tLS0K"
- name: "padawan-k8s-3"
  cluster:
    server: "https://10.138.0.5:6443"
    certificate-authority-data: "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN3akNDQ\
      WFxZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFTTVJBd0RnWURWUVFERXdkcmRXSmwKT\
      FdOaE1CNFhEVEl4TVRFd01qQTBOREV4TUZvWERUTXhNVEF6TVRBME5ERXhNRm93RWpFUU1BNEdBM\
      VVFQXhNSAphM1ZpWlMxallUQ0NBU0l3RFFZSktvWklodmNOQVFFQkJRQURnZ0VQQURDQ0FRb0NnZ\
      0VCQVBBQ1o2SW4xMW5sCmFUQjBwTEtHSlZkV3BrUDFyN3dSbG9VbENhR3ZvOGZic1g3cVMrL05jT\
      0RzbnBNQ1RDMGtzTGpkam42UTVQMU8KMXF0Ui8rT2hzVHdaUTAwYzJUNm9jSUtNNmlUSnUzTzQxe\
      TlKN2MvWXV1VmlNdWlkYXJZNWx0WUpzRER4dEh4dAoyUndzeFh3ZjcvSEt3RXBpQXo3WS83MGF6d\
      TVYRVJ4Y3ZmMjJ1SGJURnJ3REdnYllRTnhsZVREWngwVjc2eHd6CkxpTVlqU2JwT0l6eThwSTFFV\
      zZMSUtYNU84WEVGbndUZlBKaGJ1OHF2NjVYS3ZTQytCQjBXWk5vS3lEUFJBcEMKOGNpbytINGt3R\
      XJTYjZIcENjaVBmWWl2bzJYUGQxSkRtMXladHdMcXZOTVlXaWZydm4rNzdCN2xmaEVLRlpQWAptK\
      09nTEg0U2M2TUNBd0VBQWFNak1DRXdEZ1lEVlIwUEFRSC9CQVFEQWdLa01BOEdBMVVkRXdFQi93U\
      UZNQU1CCkFmOHdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBQXZZZG1pYnozRkpSVzBwdmxYWEJxe\
      WJ5ZjVMRXpGNytKTEoKQWsvZjYzSTdxMEdudzB0cjVEL0QxT3BJQjFkdDJsNnQwNGU5cHhiR0dXd\
      mJpbzJnUHdyK2ZXZ0hrbTZEbkt6Ywp4dHRDaHNlNnRrL3Zubng2V1VyZlIwVEtkZzB2QldBUXBoc\
      m5KTHB6SjJZQjJlKzRJU3hMc3cyU25hWW11Vzk4CnpJb2E1bG5zaksyczlIYlFwRVBibjNLMHNia\
      1ZsWFFieEZESkRLb0c2czFWV3FrcGJrVDFJbU8xNHdRUnA4MGIKSEY1QUpJZS9NOTBSejlqUVJrV\
      HF2RHhtcmpSdmxKNEgwVTM4UE41M28xVExsK1VuWlZ5U0VCTVRmemUwcE9jWApwbm9xNHE5V3ZpU\
      FlmM1FPbVV1NDdaelZERVBBc3JBTDdHWnkxL0dIWUJPWW0zOG4wN2s9Ci0tLS0tRU5EIENFUlRJR\
      klDQVRFLS0tLS0K"
- name: "padawan-k8s-2"
  cluster:
    server: "https://10.138.0.4:6443"
    certificate-authority-data: "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN3akNDQ\
      WFxZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFTTVJBd0RnWURWUVFERXdkcmRXSmwKT\
      FdOaE1CNFhEVEl4TVRFd01qQTBOREV4TUZvWERUTXhNVEF6TVRBME5ERXhNRm93RWpFUU1BNEdBM\
      VVFQXhNSAphM1ZpWlMxallUQ0NBU0l3RFFZSktvWklodmNOQVFFQkJRQURnZ0VQQURDQ0FRb0NnZ\
      0VCQVBBQ1o2SW4xMW5sCmFUQjBwTEtHSlZkV3BrUDFyN3dSbG9VbENhR3ZvOGZic1g3cVMrL05jT\
      0RzbnBNQ1RDMGtzTGpkam42UTVQMU8KMXF0Ui8rT2hzVHdaUTAwYzJUNm9jSUtNNmlUSnUzTzQxe\
      TlKN2MvWXV1VmlNdWlkYXJZNWx0WUpzRER4dEh4dAoyUndzeFh3ZjcvSEt3RXBpQXo3WS83MGF6d\
      TVYRVJ4Y3ZmMjJ1SGJURnJ3REdnYllRTnhsZVREWngwVjc2eHd6CkxpTVlqU2JwT0l6eThwSTFFV\
      zZMSUtYNU84WEVGbndUZlBKaGJ1OHF2NjVYS3ZTQytCQjBXWk5vS3lEUFJBcEMKOGNpbytINGt3R\
      XJTYjZIcENjaVBmWWl2bzJYUGQxSkRtMXladHdMcXZOTVlXaWZydm4rNzdCN2xmaEVLRlpQWAptK\
      09nTEg0U2M2TUNBd0VBQWFNak1DRXdEZ1lEVlIwUEFRSC9CQVFEQWdLa01BOEdBMVVkRXdFQi93U\
      UZNQU1CCkFmOHdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBQXZZZG1pYnozRkpSVzBwdmxYWEJxe\
      WJ5ZjVMRXpGNytKTEoKQWsvZjYzSTdxMEdudzB0cjVEL0QxT3BJQjFkdDJsNnQwNGU5cHhiR0dXd\
      mJpbzJnUHdyK2ZXZ0hrbTZEbkt6Ywp4dHRDaHNlNnRrL3Zubng2V1VyZlIwVEtkZzB2QldBUXBoc\
      m5KTHB6SjJZQjJlKzRJU3hMc3cyU25hWW11Vzk4CnpJb2E1bG5zaksyczlIYlFwRVBibjNLMHNia\
      1ZsWFFieEZESkRLb0c2czFWV3FrcGJrVDFJbU8xNHdRUnA4MGIKSEY1QUpJZS9NOTBSejlqUVJrV\
      HF2RHhtcmpSdmxKNEgwVTM4UE41M28xVExsK1VuWlZ5U0VCTVRmemUwcE9jWApwbm9xNHE5V3ZpU\
      FlmM1FPbVV1NDdaelZERVBBc3JBTDdHWnkxL0dIWUJPWW0zOG4wN2s9Ci0tLS0tRU5EIENFUlRJR\
      klDQVRFLS0tLS0K"

users:
- name: "padawan"
  user:
    token: "kubeconfig-user-hng6b.c-89zvp:cptnqnfmwjghll44qkfbfnprs7w9bdwtv7rph8jjft52s7zmljjhb5"

contexts:
- name: "padawan"
  context:
    user: "padawan"
    cluster: "padawan"
- name: "padawan-k8s-1"
  context:
    user: "padawan"
    cluster: "padawan-k8s-1"
- name: "padawan-k8s-3"
  context:
    user: "padawan"
    cluster: "padawan-k8s-3"
- name: "padawan-k8s-2"
  context:
    user: "padawan"
    cluster: "padawan-k8s-2"

current-context: "padawan"

```
![](img/2021-11-02-02-26-50.png)

Agora para checar se está tudo funcionando vou executar o comando abaixo que vai se conectar ao Cluster e retornar os nós que estão rodando *`nodes`*

```sh
kubectl get nodes
```

![](img/2021-11-02-02-29-07.png)

E para pegar todos os pods que estão rodando no namespace kube-system
```sh
kubectl get pods -n kube-system
```
![](img/2021-11-02-02-31-48.png)

# Instalação do Traefik **( DNS )**

## Traefik - DNS
Antes era muito utilizado o Nginx, HProxi e agora será utilizada uma nova ferramenta que é o Traefik. Cada um dos *"`nós`"* nodes do Cluster terá um container Traefik executando, ele vai ficar ouvindo quando a requisição chegar. Quando a requisição chega ela informa qual aplicação e porta que deseja chegar e o Traefik vai saber que qual é e onde internamente ela está, dessa forma temos um bind entre a aplicação e seu pod de execução interno.
![](img/2021-11-02-09-26-36.png)

```sh
kubectl apply -f https://raw.githubusercontent.com/containous/traefik/v1.7/examples/k8s/traefik-rbac.yaml
kubectl apply -f https://raw.githubusercontent.com/containous/traefik/v1.7/examples/k8s/traefik-ds.yaml
```

Exibindo os pods do traefik rodando. 
```sh
kubectl --namespace=kube-system get pods
```
![](img/2021-11-02-09-42-25.png)

No rancher podemos acessar o nome projeto e o system

![](img/2021-11-02-09-44-53.png)

E mais abaixo no Namespace kube-system podemos ver os pods do traefik executando.
![](img/2021-11-02-09-46-32.png)


Agora vamos expor o traefik. Toda vez que eu quiser criar um Ingress eu posso criar um arquivo yml como o abaixo para que ele faça esse trabalho.
```sh
cd exercicios
vi ui.yml
```

código do arquivo ui.yml 

```yml
---
apiVersion: v1
kind: Service
metadata:
  name: traefik-web-ui
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
  - name: web
    port: 80
    targetPort: 8080
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-web-ui
  namespace: kube-system
spec:
  rules:
  - host: traefik.rancher.rcic.com.br
    http:
      paths:
      - path: /
        backend:
          serviceName: traefik-web-ui
          servicePort: web
```

Após ter o arquivo criado vou executar com o comando abaixo:
```sh
kubectl apply -f ui.yml
```
![](img/2021-11-02-09-58-10.png)

Se eu acessar o endereço do Traefik ele vai me abrir a interface web onde teremos um dashboard.
![](img/2021-11-02-09-59-45.png)

Todas as vezes que for criado um Ingress ele aparecerá na guia a esqueda de "Frontend" e teremos o seu correspondente especifico para o "Backend".
![](img/2021-11-02-10-02-51.png)

Eu não consigo explicar melhor como é o funcionamento do Traefik. Mas talvez esse pensamento seja de ajuda. 
O Traefik é um DNS que recebe a requisição atravez do cluster e o direciona internamente para onde a aplicação está rodando. Ele é um DNS interno para o Kubernetes.

# Volumes

Os contêineres são imutáveis ​​significa que eles não gravam dados permanentemente em nenhum local de armazenamento, o que significa que, quando um contêiner é excluído, todos os dados gerados durante sua vida útil também são excluídos. Isso dá origem a dois problemas. Uma perda de arquivos quando o contêiner trava e os segundos arquivos não podem ser compartilhados entre os contêineres. É aí que o volume entra em cena. Basicamente, um volume é apenas um diretório, possivelmente com alguns dados, que pode ser acessado pelos contêineres de um pod. Ele resolve esses dois problemas.

Diferença entre o volume K8s e o volume persistente: o ciclo de vida do volume está vinculado a um pod. Ele é excluído quando o pod é excluído. Enquanto o volume persistente tem um ciclo de vida independente. Ele pode existir além da vida útil de um pod.

![](img/2021-11-02-10-20-29.png)

Quando trabalhamos com vários containers e precisamos gerenciar vários volumes, precisamos de um sistema de gerenciamento para esses volumes e aí entra em cena o Longhorn.
Nesse exemplo ele vai utilizar o próprio disco do host e criar um sistema de storage em cima, replicando com os armazenamentos internos dos hosts

![](img/2021-11-02-10-27-26.png)

No exemplo vamos fazer o deploy de uma aplicação MySQL/MariaDb então vamos precisar criar o volume para esse banco de dados.

Vamos acessar o projeto na guia default e selecionar APPs

![](img/2021-11-02-11-16-53.png)

Ir em Launch e buscar pela aplicação do Longhorn.

![](img/2021-11-02-11-18-38.png)
![](img/2021-11-02-11-19-51.png)

A única mudança que vou fazer no deploy do Longhorn é na sua versão que vou utilizar uma versão mais antiga para manter uma compatibilidade com o projeto.
![](img/2021-11-02-11-21-53.png)

Após clicar em Launch vamos aguardar o deploy.
![](img/2021-11-02-11-24-48.png)
![](img/2021-11-02-11-24-58.png)

Após finalizar o deploy podemos acessar o Longhorn acessando a guia Apps da area Default do projeto e clicando em index.html.
![](img/2021-11-02-11-28-13.png)

Essa é a tela de gerenciamento do Longhorn:
![](img/2021-11-02-11-29-23.png)

Em Node podemos ver quantos nós temos no cluster
![](img/2021-11-02-11-30-37.png)

Volumes criados:
![](img/2021-11-02-11-31-34.png)

Agora vamos criar um arquivo *`yml`* com as configurações do volume que queremos.
```sh
vi mariadb-longhorn-volume.yml
```

conteúdo do arquivo *`yml`*
```yml
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  ports:
    - port: 3306
  selector:
    app: mysql
  clusterIP: None

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: longhorn
  resources:
    requests:
      storage: 1Gi

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mariadb:10.4
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: changeme
        ports:
        - containerPort: 3306
          name: mysql
        
        volumeMounts:
        - name: mysql-volume
          mountPath: /var/lib/mysql
        
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "rancher"
      
      volumes:
      - name: mysql-volume
        persistentVolumeClaim:
          claimName: mysql-pvc
```


> Observe o trecho abaixo do arquivo:
> ```yml
> apiVersion: v1
> kind: PersistentVolumeClaim
> metadata:
>   name: mysql-pvc
> spec:
>   accessModes:
>     - ReadWriteOnce
>   storageClassName: longhorn
>   resources:
>     requests:
>       storage: 1Gi
> ```
> Estamos criando um volume no Longhorn com o nome de *`mysql-pvc`* com o modo de acesso *`ReadWriteOnce`* e com *`1Gi`* de armazenamento.
> <br>
> <br>
>  
> No trecho da criação do deployment do mysql temos a informação de qual volume será utilizado e o nome que será dado à esse volumen dentro do pod/container, no exemplo ele está ganhando o nome de *`mysql-volume`*.
> ```yml
>       volumes:
>       - name: mysql-volume
>         persistentVolumeClaim:
>           claimName: mysql-pvc
> ```
> <br>
> 
> E no mesmo arquivo de deployment do mysql, um pouco acima do anterior, estamos montando ele em *`/var/lib/mysql`*
> ```yml
>         volumeMounts:
>         - name: mysql-volume
>           mountPath: /var/lib/mysql
> ```

Visto isso vamos acessar o servidor do Rancher e executar o script.
```sh
kubectl apply -f mariadb-longhorn-volume.yml
```
![](img/2021-11-02-11-54-42.png)

Ao acessar o Longhorn podemos ver o volume criado:
![](img/2021-11-02-11-55-48.png)

E se acessarmos o volume vamos ver as 3 replicas criadas que estão rodando em cada um dos hosts:
![](img/2021-11-02-11-57-52.png)


Agora vamos acessar o shell do pod mysql para ver o ponto de montagem como ficou:

![](img/2021-11-02-12-03-11.png)
![](img/2021-11-02-12-02-35.png)

Tambem podemos acessar o shell indo pela máquina onde o pod está rodando. No meu caso ele está rodando no servidor ks8-2
![](img/2021-11-02-12-11-10.png)

Vou acessar por ssh esse servidor e checar com os mesmos comandos, inclusive salvando um arquivo lá para posteriormente quando ele for montado em outro servidor possamos ver esse arquivo tendo sido persistido.

Mas antes precisamos acessar o pod do mysql que esta rodando nesse servidor, então vou começar dando o comando *`docker ps`* para conseguir ver o *`container id`* do servidor e depois vou utilizar esse id para executar o shell dele
![](img/2021-11-02-12-27-26.png)

No meu exemplo o container tinha o id: *`48b647e1df3e`* então executei o comando como ilustrado abaixo:

```sh
docker exec -it 48b647e1df3e /bin/bash
```
![](img/2021-11-02-12-32-17.png)

Criando um arquivo no volume:

![](img/2021-11-02-12-35-54.png)


Agora vou simular uma falha deletando o pod pelo rancher. Assim que ele for deletado ele vai ser re-criado automanticamente e assim que for recriado vou checar se o arquivo continua lá.

![](img/2021-11-02-12-38-11.png)
![](img/2021-11-02-12-39-49.png)

No meu exemplo ele acabou criando no mesmo host, mas não tem problema, podemos checar do mesmo jeito.
Observe também que ele criou um adicional antes de excluir o anterior.

![](img/2021-11-02-12-41-11.png)

O *`container id`* mudou e vamos acessar ele por esse novo id
![](img/2021-11-02-12-43-21.png)

![](img/2021-11-02-12-54-08.png)

# LOGS
Quando temos diversos pods rodando e gerando infinitos logs precisamos centralizar essa informação para não ficarmos perdidos nos processos e possamos acompanhar o desenvolvimento das aplicações e pods. 
Para que isso seja possivel vamos realizar a instalação de uma ferramenta de log no cluster. Essa ferramenta será a Graylog.

## Deployment Graylog
O arquivo de deployment do graylog é muito grande e por esse motivo ele ficará dentro da pasta exercicios com o nome de? adivinha... *`graylog.yml`*.
Será necessário editar o arquivo e modificar para seu dominio especifico.

Abaixo as linhas que devem ser alteradas:
```sh
Linha 265 - value: http://graylog.rancher.rcic.com.br/api
Linha 341 - host: graylog.rancher.rcic.com.br
```

Observe que no arquivo ele está fazendo diversas coisas e vou pegar alguns exemplos e tentar dar maiores informações abaixo:

Criando o namespace *`graylog`*
```yml
apiVersion: v1
kind: Namespace
metadata: 
  name: graylog
```

O script *`graylog.yml`* cria diversos volumes, abaixo listo um deles que está com o nome *`mongodb-pvc`* e terá *`6Gi`* de armazenamento:
```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
  namespace: graylog
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: longhorn
  resources:
    requests:
      storage: 6Gi
```

Exemplo de role sendo criada:
```yml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: graylog
  namespace: graylog
rules:
- apiGroups:
  - ""
  resources:
  - nodes
  - nodes/proxy
  - services
  - endpoints
  - pods
  verbs:
  - get
  - list
  - watch
  verbs:
  - get
```
Alem disso tem regras de deploy do pod e serviços a ele relacionados.
<br>
<br>

## O Stack é composto de:
*`Elasticsearch, MongoDb, Graylog, FluentD`* esse ultimo estará em todos os hosts, porque ele será o responsável por capturar os logs para o *`Graylog`*.

Ok. Agora vamos finalmente rodar o script para criar todo esse Stack!
```sh
kubectl apply -f graylog.yml
```
![](img/2021-11-02-13-44-37.png)

Posso acompanhar o Deploy sendo executado se acessar o Rancher. No Rancher ele cria um namespace chamado graylog e temos que mover ele para o namespace default.
![](img/2021-11-02-13-48-34.png)
![](img/2021-11-02-13-48-43.png)
![](img/2021-11-02-13-48-52.png)

Agora que o *`graylog`* foi movido podemos ver ele e suas dependências na guia default. Observe que o *`fluentd`* está com 3 pods em execução.
![](img/2021-11-02-13-53-02.png)

Agora podemos acessar o Graylog.<br>
![](img/2021-11-02-13-59-02.png)

Segue a credencial padrão do Graylog no primeiro acesso:
Usuário: admin
Password: admin
![](img/2021-11-02-14-01-05.png)

Assim que logar vamos acessar a guia System > Inputs <br>
![](img/2021-11-02-14-03-09.png)

Vamos escolher um do tipo *`GELF UDP`* e clicar em *`Launch new input`* <br>
![](img/2021-11-02-14-05-41.png)

Vamos selecionar Global e dar um nome para ele, no meu exemplo eu dei o nome de k8s. <br>
![](img/2021-11-02-14-06-31.png)

Assim que adicionarmos ele vai começar a receber logs do *`fluentd`*, então vamos preparar alguns filtros.
![](img/2021-11-02-14-08-16.png)

Para isso vou acessar Manage Extractor e configur/adicionar um extrator para retirar do log só o que interessa.<br>
![](img/2021-11-02-14-09-33.png)

Em Add extractor clique em Get started depois em Load Message e em kubernetes acesse e escolha JSON
![](img/2021-11-02-14-14-15.png)

Na próxima guia apenas adicione um prefix e um title onde informado e clique em create extractor.
![](img/2021-11-02-14-18-09.png)

Agora o Graylog está rodando e capturando as informações necessárias.
![](img/2021-11-02-14-24-35.png)
![](img/2021-11-02-14-24-43.png)

> Considerações: Com o *`Graylog`* podemos capturar os logs da aplicação e dar acesso ao desenvolvedor para que ele possa acessar os logs de sua aplicação sem que seja necessário eu dar acesso ao Cluster à ele, além de ter uma ferramenta onde podemos centralizar todos os logs e acompanha-los. O *`Graylog`* também é capaz de pegar os logs do LDAP.

<br>

# Monitoramento

O Grafana/Prometheus é a stack que iremos usar para monitoramento. O Deployment dela será feito pelo Catálogo de Apps.

## Habilitando o monitoramento.
Para habilitar o monitoramento, devemos acessar o rancher e ir na aba tools > monitoring.
![](img/2021-11-02-18-24-52.png)

Lá devemos habilitar o monitoramento clicando em Enable.
> Aqui quero fazer uma ressalva. Em um ambiente real de produção, temos que aumentar os valores de CPU e Memória e quem sabe também habilitar a persistência de dados. Em um ambiente real de produção precisamos que a máquina tenha mais poder de *`"fogo"`* para conseguir processar todas as informações.

Foi criado um Namespace chamado cattle-prometheus, o print abaixo está em fase de deploying
![](img/2021-11-02-18-28-40.png)

E agora essa é a nova tela com o monitoramento ativo.
![](img/2021-11-02-18-30-36.png)

Também ganhamos uma área de metricas.
![](img/2021-11-02-18-31-14.png)

E se em qualquer lugar da tela eu clicar no icone do Grafana ele vai abrir o Grafana e exibir tudo que está sendo monitorado por ele no momento.
![](img/2021-11-02-18-33-36.png)


# Cronjobs

O tipo de serviço como CronJob é um serviço igual a uma cron, porém é executado no cluster kubernetes. Você agenda um pod que irá rodar em uma frequência determinada de tempo. Pode ser usado para diversas funções, como executar backup's dos bancos de dados.

Nesse exemplo, iremos executar um pod, com um comando para retornar uma mensagem de tempos em tempos, a mensagem é "Hello from the Kubernetes cluster"

Conteúdo do arquivo *`conjob.yml`*:
```yml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config2
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm

---
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
  namespace: default
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: test-container
              image: k8s.gcr.io/busybox
              command: [ "/bin/sh", "-c", "env" ]
              envFrom:
              - configMapRef:
                  name: special-config2
          # restartPolicy: Never
          restartPolicy: OnFailure 
```

```sh
kubectl apply -f cronjob.yml
```
> configmap/special-config2 created
> cronjob.batch/hello created

Depois de criada a cron, pegamos o estado dela usando:
```sh
kubectl get cronjob hello
```
  | NAME  |  SCHEDULE   | SUSPEND | ACTIVE | LAST SCHEDULE |  AGE  |
  | :---: | :---------: | :-----: | :----: | :-----------: | :---: |
  | hello | */1 * * * * |  False  |   0    |    \<none>    |  29s  |

Ainda não existe um job ativo, e nenhum agendado também.
Vamos esperar por 1 minutos ate o job ser criado:
```sh
kubectl  get jobs --watch
```
![](img/2021-11-02-19-07-34.png)

Entrar no Rancher para ver os logs e a sequencia de execucao.
![](img/2021-11-02-19-05-46.png)
![](img/2021-11-02-19-06-03.png)
<br>

# ConfigMap

O ConfigMap é um tipo de componente muito usado, principalmente quando precisamos colocar configurações dos nossos serviços externas aos contâiners que estão rodando a aplicação. 

Nesse exemplo, iremos criar um ConfigMap, e iremos acessar as informações dentro do container que está a aplicação.

Código do arquivo configmap.yml
```yml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm

---
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - configMapRef:
          name: special-config
  restartPolicy: Never
```

```sh
kubectl apply -f configmap.yml
```
Agora iremos entrar dentro do container e verificar as configurações definidas no ConfigMap.
