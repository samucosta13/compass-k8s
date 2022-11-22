# Documentação
Implantação de uma aplicação do Wordpress em um cluster com Kubernetes

## Pré requisitos do sistema operacional
### **Para usuários do Windows**
* [Instalar o Docker Desktop](https://docs.docker.com/desktop/install/windows-install/)
* Habilitar o Hyper-V
  
  Para verificar se o Hyper-V está habilitado, abra o prompt de comando como administrador e execute o comando `bcdedit`. Se o Hyper-V estiver desabilitado, execute `bcdedit /set hypervisorlaunchtype auto` no prompt de comando aberto como administrador, e reinicie seu computador.

### **Para usuários de distribuições Linux baseadas no Debian**
* [Instalar o Docker e o containerd](https://docs.docker.com/engine/install/debian/)
  
  > O containerd é uma alternativa ao _engine_ do Docker, e é necessário por estabelecer comunicação entre o Docker e o Kubernetes
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

No diretório `namespace/` crie o namespace `labwordpress` a partir do arquivo YAML que o define (`labwordpress.yml`). Para isso, execute o comando:

`kubectl create -f labwordpress.yml`

### Segundo passo: Secrets
Após o namespace `labwordpress` ter sido criado, você precisará editar os templates disponibilizados no diretório `templates/`. Eles são arquivos YAML responsáveis pela criação das variáveis de ambiente que serão utilizadas pelo Wordpress e pelo MySQL. Escolhemos o tipo "Secret" pois ele criptografa o valor das variáveis de ambiente, caso sejam visualizadas por um `kubectl get secret`.

O arquivo `wordpress-secret.yml` refere-se às variáveis de ambiente do Wordpress, ao passo que o arquivo `mysql-secret.yml` refere-se às variáveis do MySQL Server. Defina os valores delas nos arquivos e, caso queira versionar seu ambiente, retire os Secrets do diretório versionado ou adicione o nome deles no .gitignore (ou correspondente), para não vazar informações sensíveis da sua aplicação.

Após editar os templates e resguardá-los, crie os dois Secrets no seu cluster Kubernetes:

`kubectl create -f wordpress-secret.yml`

`kubectl create -f mysql-secret.yml`

### Terceiro passo: PV e PVC

### Quarto passo: Services

### Quinto passo: Deployments

### Sexto passo: Ingress

## Acessando a aplicação
...
