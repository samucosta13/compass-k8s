# Documentação
Implantação de uma aplicação do Wordpress em um cluster com Kubernetes

## Pré requisitos do sistema operacional
### **Para usuários do Windows**
* [Instalar o Docker Desktop](https://docs.docker.com/desktop/install/windows-install/)
* Habilitar o Hyper-V
  
  Para verificar se o Hyper-V está habilitado, abra o prompt de comando como administrador e execute o comando `bcdedit`. Se o Hyper-V estiver desabilitado, execute `bcdedit /set hypervisorlaunchtype auto` no prompt de comando aberto como administrador, e reinicie seu computador.

### **Para usuários de distribuições Linux baseadas no Debian**
* [Instalar o Docker e o containerd](https://docs.docker.com/engine/install/debian/)
  
  > O containerd é uma alternativa ao _engine_ do Docker, e é necessário por estabelecer comunicação entre os containers do Docker e o Kubernetes
* [Instalar o Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
  
  > Ferramenta de linha de comando do Kubernetes
* [Instalar o Minikube](https://minikube.sigs.k8s.io/docs/start/)
  
  > Ferramenta que permite executar o Kubernetes localmente
* Definir o Docker como o executor de containers do Minikube
  > Para isso, execute `minikube config set driver docker`


## Iniciando o cluster com Kubernetes
### Minikube
No terminal, inicie um novo cluster com o comando `minikube start`.
> Caso seja solicitado, execute `minikube delete` antes de iniciar o cluster.

### Docker Desktop
...

## Implantando a infraestrutura da aplicação
Nesta etapa, você irá utilizar o **kubectl** e o conteúdo deste repositório.

Para executar comandos com o kubectl, você pode utilizar o prompt de comando do Windows, o terminal do Linux ou qualquer IDE de sua preferência que forneça algum terminal, como o Visual Studio Code.


### Primeiro passo: Namespace
Inicialmente, você deve criar um namespace para implantar os objetos da aplicação dentro dele. Ele funciona como um cluster virtual, onde você pode isolar um ambiente de trabalho para objetivos específicos.

Crie o namespace `labwordpress` a partir do arquivo YAML que se encontra no diretório `namespace/`. Para isso, execute o comando:

`kubectl create -f namespace/labwordpress.yml`


### Segundo passo: Secrets
Após o namespace `labwordpress` ter sido criado, você precisará editar os templates disponibilizados no diretório `templates/`. Eles são arquivos YAML responsáveis pela criação das variáveis de ambiente que serão utilizadas pelo Wordpress e pelo MySQL. Escolhemos o tipo "Secret" pois ele criptografa o valor das variáveis de ambiente, caso sejam visualizadas por um `kubectl get secret`.

O arquivo `wordpress.yml` refere-se às variáveis de ambiente do Wordpress, ao passo que o arquivo `mysql.yml` refere-se às variáveis do MySQL Server. Defina os valores delas nos arquivos e, caso queira versionar seu ambiente, retire os Secrets do diretório versionado ou adicione o nome deles no .gitignore (ou correspondente), para não vazar informações sensíveis da sua aplicação.

Após editar os templates e resguardá-los, crie os dois Secrets no seu cluster Kubernetes a partir dos dois arquivos YAML, executando os seguintes comandos:

`kubectl create -f [DIRETÓRIO DO SECRET]/wordpress.yml`

`kubectl create -f [DIRETÓRIO DO SECRET]/mysql.yml`


### Terceiro passo: PV e PVC
Agora, é preciso criar volumes para persistência dos dados dos containers da aplicação e do banco de dados. Uma possível solução é criar Persistent Volumes (PVs), que são diretórios no cluster sincronizados com diretórios dentro dos containers, e também criar Persistent Volume Claims (PVCs), que são resquisições para esses volumes.

Os PVs estão no diretório `persistentVolumes/`, ao passo que os PVCs se encontram em `persistentVolumeClaims/`. Para criá-los, execute os comandos:

`kubectl create -f persistentVolumes/wordpress.yml`

`kubectl create -f persistentVolumes/mysql.yml`

`kubectl create -f persistentVolumeClaims/wordpress.yml`

`kubectl create -f persistentVolumeClaims/mysql.yml`


### Quarto passo: Services
Para permitir acesso entre objetos do Kubernetes, e também acessos de origem externa a eles, usa-se serviços. O ClusterIP opera dentro do cluster, liberando a comunicação interna entre seus objetos. O Ingress possibilita o acesso a objetos do cluster a partir de urls ou endereçamento IP.

O arquivo YAML do Ingress é o `ingress.yml`. Para criá-lo, execute o comando `kubectl create -f ingress.yml`.

Os arquivos do CLusterIP referentes aos pods do Wordpress e do MySQL estão no diretório `services/`. Para criar os serviços a partir deles, execute os comandos:

`kubectl create -f services/wordpress.yml`

`kubectl create -f services/mysql.yml`


### Quinto passo: Deployments
Configurados os volumes, serviços e o namespace onde será implantada a aplicação no cluster, crie os deployments do Wordpress e do MySQL, que irão criar os containers e garantir a sua disponibilidade utilizando ReplicaSets (replicadores do Kubernetes). Execute os seguintes comandos:

`kubectl create -f deployments/mysql.yml`
> Recomendamos criar o deployment do MySQL primeiro, para depois criar o da aplicação, pois esta precisará utilizar um banco de dados do MySQL Server


`kubectl create -f deployments/wordpress.yml`

### Sexto passo: Ingress

## Acessando a aplicação
...
