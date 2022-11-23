# Documentação
O objetivo desta documentação é exibir o passo a passo para subir uma aplicação do Wordpress em um cluster com Kubernetes, sendo gerenciado pelo Rancher.


## :wrench: Pré requisitos do sistema operacional
### **Para usuários do Windows**
* Instalar o [Docker Desktop](https://docs.docker.com/desktop/install/windows-install/)
* Habilitar o Hyper-V
  
  Para verificar se o Hyper-V está habilitado, abra o prompt de comando como administrador e execute o comando `bcdedit`. Se o Hyper-V estiver desabilitado, execute `bcdedit /set hypervisorlaunchtype auto` no prompt de comando aberto como administrador, e reinicie seu computador.

### **Para usuários de distribuições Linux baseadas no Debian**
* Instalar o [Docker e o containerd](https://docs.docker.com/engine/install/debian/)
  
  > O containerd é uma alternativa ao _engine_ do Docker, e é necessário por estabelecer comunicação entre os containers do Docker e o Kubernetes
* Instalar o [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
  
  > Ferramenta de linha de comando do Kubernetes
* Instalar o [Minikube](https://minikube.sigs.k8s.io/docs/start/)
  
  > Ferramenta que permite executar o Kubernetes localmente
* Definir o Docker como o executor de containers do Minikube
  > Para isso, execute `minikube config set driver docker`


## Iniciando o cluster com Kubernetes
### Minikube
No terminal, inicie um novo cluster com o comando `minikube start`.
> Caso seja solicitado, execute `minikube delete` antes de iniciar o cluster.

### Docker Desktop
No Docker Desktop, acesse Setting>Kubernetes>Enable Kubernetes.
Após isso, selecione "Apply & Restart"

##  :hammer: Implantando a infraestrutura da aplicação
Nesta etapa, você irá utilizar o **kubectl** e o conteúdo deste repositório.

Para executar comandos com o kubectl, você pode utilizar o prompt de comando do Windows, o terminal do Linux ou qualquer IDE de sua preferência que forneça algum terminal, como o Visual Studio Code.

Neste repositório, você encontra dois diretórios principais: `docker-dektop/` e `minikube/`. Eles se destinam a usuários do Docker Desktop no Windows e do Minikube no Linux, respectivamente. Acesse o que lhe diz respeito antes de passar para os próximos passos.


### :one: Primeiro passo: Namespace
Inicialmente, você deve criar um namespace para implantar os objetos da aplicação dentro dele. Ele funciona como um cluster virtual, onde você pode isolar um ambiente de trabalho para objetivos específicos.

Crie o namespace `labwordpress` a partir do arquivo YAML que se encontra no diretório `namespace/`. Para isso, execute o comando:

`kubectl create -f namespace/labwordpress.yml`


### :two: Segundo passo: Secrets
Após o namespace `labwordpress` ter sido criado, você precisará editar os templates disponibilizados no diretório `templates/`. Eles são arquivos YAML responsáveis pela criação das variáveis de ambiente que serão utilizadas pelo Wordpress e pelo MySQL. Escolhemos o tipo "Secret" pois ele criptografa o valor das variáveis de ambiente, caso sejam visualizadas por um `kubectl get secret`.

O arquivo `wordpress.yml` refere-se às variáveis de ambiente do Wordpress, ao passo que o arquivo `mysql.yml` refere-se às variáveis do MySQL Server. Defina os valores delas nos arquivos e, caso queira versionar seu ambiente, retire os Secrets do diretório versionado ou adicione o nome deles no .gitignore (ou correspondente), para não vazar informações sensíveis da sua aplicação.

Após editar os templates e resguardá-los, crie os dois Secrets no seu cluster Kubernetes a partir dos dois arquivos YAML, executando os seguintes comandos:

`kubectl create -f [DIRETÓRIO DO SECRET]/wordpress.yml`

`kubectl create -f [DIRETÓRIO DO SECRET]/mysql.yml`


### :three: Terceiro passo: PV e PVC
Agora, é preciso criar volumes para persistência dos dados dos containers da aplicação e do banco de dados. Uma possível solução é criar Persistent Volumes (PVs), que são diretórios no cluster sincronizados com diretórios dentro dos containers, e também criar Persistent Volume Claims (PVCs), que são resquisições para esses volumes.

Os PVs estão no diretório `persistentVolumes/`, ao passo que os PVCs se encontram em `persistentVolumeClaims/`. Para criá-los, execute os comandos:

`kubectl create -f persistentVolumes/wordpress.yml`

`kubectl create -f persistentVolumes/mysql.yml`

`kubectl create -f persistentVolumeClaims/wordpress.yml`

`kubectl create -f persistentVolumeClaims/mysql.yml`


### :four: Quarto passo: Services
Para permitir acesso entre PODs no cluster, usa-se serviços. O ClusterIP é um tipo de serviço do Kubernetes que opera dentro do cluster, liberando a comunicação interna entre seus objetos. Os arquivos do CLusterIP referentes aos pods do Wordpress e do MySQL estão no diretório `services/`. Para criar os serviços a partir deles, execute os comandos:

`kubectl create -f services/wordpress.yml`

`kubectl create -f services/mysql.yml`


### :five: Quinto passo: Deployments
Configurados os volumes, serviços e o namespace onde será implantada a aplicação no cluster, crie os deployments do Wordpress e do MySQL, que irão criar os containers e garantir a sua disponibilidade utilizando ReplicaSets (replicadores do Kubernetes). Execute os seguintes comandos:

`kubectl create -f deployments/mysql.yml`
> Recomendamos criar o deployment do MySQL primeiro, para depois criar o da aplicação, pois esta precisará utilizar um banco de dados do MySQL Server


`kubectl create -f deployments/wordpress.yml`

### :six: Sexto passo: Ingress

#### **Para usuários do Windows**

Para acessarmos a aplicação do Wordpress a partir de um navegador (externo ao cluster) é preciso viabilizar que POD do Wordpress responda a requisições externas, e uma maneira de fazer isso é utilizar um Ingress. No diretório corrente (`docker-dektop/`), existe um arquivo YAML que define um Ingress, que é o `ingress.yml`. Neste arquivo, encontra-se uma URL a partir da qual o acesso ao Wordpress poderá ser feito. Você pode editá-la!

Para criar o Ingress, execute o comando `kubectl create -f ingress.yml`.

#### **Para usuários de distribuições Linux baseadas no Debian**

Para acessarmos a aplicação do Wordpress a partir de um navegador (externo ao cluster) é preciso viabilizar que POD do Wordpress responda a requisições externas, e uma maneira de fazer isso é utilizar um Ingress. Usuários do Minikube precisam habilitar o NGINX Ingress Controller no seu Minikube. Isso pode ser feito com o comando `minikube addons enable ingress`. 

Logo após, execute `kubectl get pods -n ingress-nginx`. A saída deste útlimo comando deve conter pelo menos um pod no estado **running** com o início do nome igual a **ingress-nginx-controller** ou **nginx-ingress-controller**.

Feitas as configurações no Minikube, crie o Ingress a partir do arquivo `ingress.yml` disponível no diretório corrente (`minikube/`), executando o comando `kubectl create -f ingress.yml`.

## :key: Acessando a aplicação

Após constuir a infraestrutura do cluster, é hora de acessar a interface do Wordpress. Para isso, execute o comando `kubectl get ingress` para visualizar o IP pelo qual você poderá acessar a aplicação. Caso queira utilizar a URL em vez do endereço IP, você precisará editar o arquivo hosts do seu sistema operacional, inserindo o IP do Ingress e a URL que você definiu no YAML dele.

> No Linux/Debian, você precisará editar o arquivo /etc/hosts com permissão de usuário root!

Feito isso, abra algum navegador e digite na barra de endereços a URL que se encontra no arquivo do seu Ingress, ou o endereço IP gerado por ele. E pronto! Agora você consegue utilizar sua aplicação do Wordpress tranquilamente!

## :cow2: Gerenciando cluster pelo Rancher
Uma maneira de administrar seu cluster kubernetes é utilizando uma ferramenta opensource chamada [Rancher](https://www.rancher.com/why-rancher).  
Para fazer isso, é necessário subir um container contendo o Rancher e acessá-lo pelo navegador. Para isso, execute `docker run --privileged -d --restart=unless-stopped -p 8080:80 -p 8443:443 rancher/rancher:stable`. Após isso, você já pode acessar o Rancher pelo *localhost:8443* .  
Com o comando `docker ps`, obtêm-se o id do container, que é necessário para encontrar a primeira senha de acesso no Rancher. Para isso:
```
# Linux
docker logs [CONTAINER_ID] 2>&1 | grep "Bootstrap Password:"

# Windows
docker logs [CONTAINER_ID] 2>&1 | findstr "Bootstrap Password:"
```

Após esses passos, crie a senha para o usuário admin, e daí sua ferramenta de gerenciamento de clusters já estará disponível para uso!
