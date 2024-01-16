<div align="center">

![ArgoCD](./Images/argocd.svg) </center>

</div>

[![Github Badge](https://img.shields.io/badge/-Github-000?style=flat-square&logo=Github&logoColor=white&link=https://github.com/emanuelfds)](https://github.com/emanuelfds)
[![Linkedin Badge](https://img.shields.io/badge/-LinkedIn-blue?style=flat-square&logo=Linkedin&logoColor=white&link=https://www.linkedin.com/in/emanuelfds/)](https://www.linkedin.com/in/emanuelfds/)

## O que é o ArgoCD?

O ArgoCD é uma poderoso ferramenta quando você pensa em GitOps ou Continous Delivery. O ArgoCD é um projeto open source, criado pela [Argo](https://argoproj.github.io/), que tem como objetivo facilitar a implantação e gerenciamento de aplicações em Kubernetes.

O ArgoCD foi escrito em Go e utiliza o [Kubernetes Operator Pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) para gerenciar os recursos do Kubernetes.

Dito isso, fica claro que quando você instala o ArgoCD, você está instalando um operador do Kubernetes, que está extendendo o Kubernetes, adicionando novos `Custom Resources` e novos `Controllers` para gerenciar esses `Custom Resources`.

Um forte característica do ArgoCD é a sua separação de responsabilidades quando estamos falando de CI/CD. Ele não se preocupa em ser um solução completa para a sua esteira de CI/CD, ele se preocupa em ser uma ferramenta que vai gerenciar as entregas contínuas no Kubernetes, ele se preocupa com a parte CD, e não com a parte CI.

## O que é GitOps?

O GitOps é um conceito que foi criado pela [Weaveworks](https://www.weave.works/), e que tem como objetivo facilitar a entrega de aplicações no Kubernetes, utilizando o Git como fonte de verdade. O Git é a fonte de verdade, e o Git é o único lugar onde você vai encontrar a verdade sobre o estado da sua aplicação.

Se lá é a fonte da verdade, vale a pena falar que quando estamos falando de GitOps, estamos falando sobre modo declarativo de gerenciar as aplicações no Kubernetes. Quando falamos em declarativo, estamos falando que o estado que das suas aplicação no Kubernetes, é o mesmo que está no Git, que é o mesmo que você deseja que esteja no Kubernetes.

O GitOps simplifica a propagação de alterações de configuração de infraestrutura e aplicativos em vários clusters, definindo suas definições de infraestrutura e aplicativos como “código”.

Vamos imaginar que você tenha o seguinte arquivo no Git:

```yaml
apiVersion: apps/v1 # versão da API
kind: Deployment # tipo de recurso, no caso, um Deployment
metadata: # metadados do recurso 
  name: nginx-server # nome do recurso
spec: # especificação do recurso
  selector: # seletor para identificar os pods que serão gerenciados pelo deployment
    matchLabels: # labels que identificam os pods que serão gerenciados pelo deployment
      app: nginx # label que identifica o app que será gerenciado pelo deployment
  replicas: 2 # quantidade de réplicas do deployment
  template: # template do deployment
    metadata: # metadados do template
      labels: # labels do template
        app: nginx # label que identifica o app
      annotations: # annotations do template
        prometheus.io/scrape: 'true' # habilita o scraping do Prometheus
        prometheus.io/port: '9113' # porta do target
    spec: # especificação do template
      containers: # containers do template 
        - name: nginx # nome do container
          image: nginx # imagem do container do Nginx
          ports: # portas do container
            - containerPort: 80 # porta do container
              name: http # nome da porta
          volumeMounts: # volumes que serão montados no container
            - name: nginx-config # nome do volume
              mountPath: /etc/nginx/conf.d/default.conf # caminho de montagem do volume
              subPath: nginx.conf # subpath do volume
        - name: nginx-exporter # nome do container que será o exporter
          image: 'nginx/nginx-prometheus-exporter:0.11.0' # imagem do container do exporter
          args: # argumentos do container
            - '-nginx.scrape-uri=http://localhost/metrics' # argumento para definir a URI de scraping
          resources: # recursos do container
            limits: # limites de recursos
              memory: 128Mi # limite de memória
              cpu: 0.3 # limite de CPU
          ports: # portas do container
            - containerPort: 9113 # porta do container que será exposta
              name: metrics # nome da porta
      volumes: # volumes do template
        - configMap: # configmap do volume, nós iremos criar esse volume através de um configmap
            defaultMode: 420 # modo padrão do volume
            name: nginx-config # nome do configmap
          name: nginx-config # nome do volume
```

Vamos dar o nome para esse arquivo de `nginx-deployment.yaml`.

Esse arquivo é somente um manifesto do Kubernetes, onde estamos especificando um `Deployment` do Nginx, com 2 réplicas, onde o Nginx está exposto na porta 80, e o Prometheus está fazendo o scraping da porta 9113.

Perceba, nesse arquivo estamos falando para o Kubernetes, que queremos que o Nginx esteja rodando com 2 réplicas, e que o Prometheus está fazendo o scraping da porta 9113, estamos declarando o estado desejado da nossa aplicação.

Para aplicar esse arquivo no Kubernetes, basta executar o seguinte comando:

```bash
kubectl apply -f nginx-deployment.yaml
```

Vamos imaginar que por algum motivo precisamos mudar a quantidade de réplicas do Nginx para 3. Nós desejamos declarar o estado da nossa aplicação para 3 réplicas. 

Para isso, basta declarar, alterar a quantidade de réplicas do Nginx para 3 no arquivo `nginx-deployment.yaml`:

```yaml
apiVersion: apps/v1 # versão da API
kind: Deployment # tipo de recurso, no caso, um Deployment
metadata: # metadados do recurso 
  name: nginx-server # nome do recurso
spec: # especificação do recurso
  selector: # seletor para identificar os pods que serão gerenciados pelo deployment
    matchLabels: # labels que identificam os pods que serão gerenciados pelo deployment
      app: nginx # label que identifica o app que será gerenciado pelo deployment
  replicas: 3 # quantidade de réplicas do deployment
  template: # template do deployment
    metadata: # metadados do template
      labels: # labels do template
        app: nginx # label que identifica o app
      annotations: # annotations do template
        prometheus.io/scrape: 'true' # habilita o scraping do Prometheus
        prometheus.io/port: '9113' # porta do target
    spec: # especificação do template
      containers: # containers do template 
        - name: nginx # nome do container
          image: nginx # imagem do container do Nginx
          ports: # portas do container
            - containerPort: 80 # porta do container
              name: http # nome da porta
          volumeMounts: # volumes que serão montados no container
            - name: nginx-config # nome do volume
              mountPath: /etc/nginx/conf.d/default.conf # caminho de montagem do volume
              subPath: nginx.conf # subpath do volume
        - name: nginx-exporter # nome do container que será o exporter
          image: 'nginx/nginx-prometheus-exporter:0.11.0' # imagem do container do exporter
          args: # argumentos do container
            - '-nginx.scrape-uri=http://localhost/metrics' # argumento para definir a URI de scraping
          resources: # recursos do container
            limits: # limites de recursos
              memory: 128Mi # limite de memória
              cpu: 0.3 # limite de CPU
          ports: # portas do container
            - containerPort: 9113 # porta do container que será exposta
              name: metrics # nome da porta
      volumes: # volumes do template
        - configMap: # configmap do volume, nós iremos criar esse volume através de um configmap
            defaultMode: 420 # modo padrão do volume
            name: nginx-config # nome do configmap
          name: nginx-config # nome do volume
```

Para que sua vonta seja aplicada, basta executar o seguinte comando:

```bash
kubectl apply -f nginx-deployment.yaml
```

Pronto, agora o Kubernetes sabe que você deseja que o Nginx esteja rodando com 3 réplicas e assim o fez.

Isso é o que chamamos de **estado desejado**, estado que declaramos para o Kubernetes, e o Kubernetes mudou o estado atual para o estado desejado.

Acredito que agora você tenha entendido um conceito muito importante do Kubernetes e no GitOps, que é o **estado desejado**.

Se é importante para o GitOps, é importante para o ArgoCD, e é por isso que o ArgoCD trabalha com o conceito de **estado desejado**.

Basicamente o GitOps é uma metodologia de gerenciamento de configurações, onde o Git é a única fonte de verdade, e o Git é o único responsável por declarar o estado desejado da aplicação.

## Pré-requisitos

Para que possamos continuar daqui para frente, precisamos ter o seguinte instalado:

- Um cluster Kubernetes
- kubectl instalado

## Instalando o ArgoCD

Aqui precisamos dividir essa instalação em duas partes, a instalação do ArgoCD como um operador no Kubernetes, e a instalação do ArgoCD como CLI, para que você possa utilizar o ArgoCD no seu dia a dia. Ele possui ainda uma interface gráfica, que é o ArgoCD UI.

### Instalando o ArgoCD como um operador no Kubernetes

Para instalar o ArgoCD como um operador no Kubernetes, antes precisamos criar uma namespace chamada `argocd`, e para isso basta executar o seguinte comando:

```bash
kubectl create namespace argocd
```

A saída desse comando será algo parecido com isso:

```bash
namespace/argocd created
```

Agora vamos instalar o ArgoCD como um operador no Kubernetes:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

A saída desse comando será algo parecido com isso:

```bash
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/applicationsets.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
serviceaccount/argocd-application-controller created
serviceaccount/argocd-applicationset-controller created
serviceaccount/argocd-dex-server created
serviceaccount/argocd-notifications-controller created
serviceaccount/argocd-redis created
serviceaccount/argocd-repo-server created
serviceaccount/argocd-server created
role.rbac.authorization.k8s.io/argocd-application-controller created
role.rbac.authorization.k8s.io/argocd-applicationset-controller created
role.rbac.authorization.k8s.io/argocd-dex-server created
role.rbac.authorization.k8s.io/argocd-notifications-controller created
role.rbac.authorization.k8s.io/argocd-server created
clusterrole.rbac.authorization.k8s.io/argocd-application-controller created
clusterrole.rbac.authorization.k8s.io/argocd-server created
rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
rolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
rolebinding.rbac.authorization.k8s.io/argocd-notifications-controller created
rolebinding.rbac.authorization.k8s.io/argocd-redis created
rolebinding.rbac.authorization.k8s.io/argocd-server created
clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-server created
configmap/argocd-cm created
configmap/argocd-cmd-params-cm created
configmap/argocd-gpg-keys-cm created
configmap/argocd-notifications-cm created
configmap/argocd-rbac-cm created
configmap/argocd-ssh-known-hosts-cm created
configmap/argocd-tls-certs-cm created
secret/argocd-notifications-secret created
secret/argocd-secret created
service/argocd-applicationset-controller created
service/argocd-dex-server created
service/argocd-metrics created
service/argocd-notifications-controller-metrics created
service/argocd-redis created
service/argocd-repo-server created
service/argocd-server created
service/argocd-server-metrics created
deployment.apps/argocd-applicationset-controller created
deployment.apps/argocd-dex-server created
deployment.apps/argocd-notifications-controller created
deployment.apps/argocd-redis created
deployment.apps/argocd-repo-server created
deployment.apps/argocd-server created
statefulset.apps/argocd-application-controller created
networkpolicy.networking.k8s.io/argocd-application-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-applicationset-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-dex-server-network-policy created
networkpolicy.networking.k8s.io/argocd-notifications-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-redis-network-policy created
networkpolicy.networking.k8s.io/argocd-repo-server-network-policy created
networkpolicy.networking.k8s.io/argocd-server-network-policy created
```

Como você pode ver, com o comando acima configuramos o ArgoCD através da criação de vários objetos no Kubernetes, como por exemplo, um `deployment` para o `argocd-server`, um `service` para o `argocd-server`, um `configmap` para o `argocd-cm`, e por aí vai.

Caso você queira conhecer mais sobre o projeto, vá até o [repositório oficial](https://github.com/argoproj/argo-cd)

Vamos ver se os pods do ArgoCD foram criados com sucesso:

```bash
kubectl get pods -n argocd
```

A saída desse comando será algo parecido com isso:

```bash
NAME                                                READY   STATUS    RESTARTS   AGE
argocd-applicationset-controller-57db5f5c7d-z6l5z   1/1     Running   0          100s
argocd-notifications-controller-7cddc64d84-b7xfp    1/1     Running   0          100s
argocd-redis-6b7c6f67db-r47p2                       1/1     Running   0          100s
argocd-application-controller-0                     1/1     Running   0          100s
argocd-dex-server-c4b8545d-225mm                    1/1     Running   0          100s
argocd-repo-server-6867ddcc74-wcszg                 1/1     Running   0          100s
argocd-server-64957744c9-h6vzm                      1/1     Running   0          100s
```

Onde temos os seguintes pods:

* argocd-application-controller-0 - Responsável por gerenciar os recursos do Kubernetes
* argocd-applicationset-controller-57db5f5c7d-z6l5z - Controller responsável por gerenciar os `ApplicationSets`
* argocd-dex-server-c4b8545d-225mm - Responsável por gerenciar a autenticação
* argocd-notifications-controller-7cddc64d84-b7xfp - Responsável por gerenciar as notificações, como por exemplo, quando um `Application` é atualizado
* argocd-redis-6b7c6f67db-r47p2 - Responsável por armazenar os dados do ArgoCD
* argocd-repo-server-6867ddcc74-wcszg - Responsável por gerenciar os repositórios
* argocd-server-64957744c9-h6vzm - Responsável por expor a interface gráfica do ArgoCD

Todos os nossos podes estão com o status `Running`, o que significa que eles estão funcionando corretamente.

## Instalando o ArgoCD CLI

O ArgoCD possui uma interface gráfica, mas também é possível interagir com ele através de comandos. Para isso, precisamos instalar o `argocd` CLI.

Para instalar o `argocd` CLI no Linux, basta executar o seguinte comando:

```bash
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd

rm argocd-linux-amd64
```

Com o comando acima fizemos o download do binário do `argocd` CLI, e o instalamos no diretório `/usr/local/bin/argocd`, para fazer a instalação utilizamos o comando `install` do Linux, que é um comando que faz a instalação de arquivos e diretórios. Passamos os parâmetros `-m 555` para definir as permissões do arquivo, e o nome do arquivo que queremos instalar.

Pronto! O nosso `argocd` CLI está instalado.

Vamos ver se ele está funcionando corretamente:

```bash
argocd version
```

A saída desse comando será algo parecido com isso:

```bash
argocd: v2.6.7+5bcd846
  BuildDate: 2023-03-23T15:24:49Z
  GitCommit: 5bcd846fa16e4b19d8f477de7da50ec0aef320e5
  GitTreeState: clean
  GoVersion: go1.18.10
  Compiler: gc
  Platform: linux/amd64
FATA[0000] Argo CD server address unspecified
```

## Configurando o ArgoCD com `Application` CRD

Agora que já temos o ArgoCD instalado, tanto o CLI quanto o operador, precisamos fazer a autenticação no ArgoCD para que possamos dar os primeiros passos.

Antes de mais nada, precisamos saber qual o endereço do ArgoCD. Para isso, vamos executar o seguinte comando:

```bash
kubectl get svc -n argocd
```

A saída desse comando será algo parecido com isso:

```bash
NAME                                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
argocd-applicationset-controller          ClusterIP   10.96.25.215    <none>        7000/TCP,8080/TCP            10m
argocd-dex-server                         ClusterIP   10.108.186.66   <none>        5556/TCP,5557/TCP,5558/TCP   10m
argocd-metrics                            ClusterIP   10.96.90.231    <none>        8082/TCP                     10m
argocd-notifications-controller-metrics   ClusterIP   10.102.27.1     <none>        9001/TCP                     10m
argocd-redis                              ClusterIP   10.96.170.218   <none>        6379/TCP                     10m
argocd-repo-server                        ClusterIP   10.111.224.12   <none>        8081/TCP,8084/TCP            10m
argocd-server                             ClusterIP   10.97.138.104   <none>        80/TCP,443/TCP               10m
argocd-server-metrics                     ClusterIP   10.103.236.94   <none>        8083/TCP                     10m
```

O service que precisamos por agora do ArgoCD é o `argocd-server`, e o endereço completo é `argocd-server.argocd.svc.cluster.local`.

Vamos fazer o port-forward para acessar o ArgoCD sem precisar expor:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Pronto, agora podemos acessar o ArgoCD através do endereço `localhost:8080`, tanto pelo navegador quanto pelo CLI.

Vamos continuar com a nossa saga utilizando o CLI, então vamos fazer a autenticação no ArgoCD.

Para fazer a autenticação no ArgoCD, precisamos executar o seguinte comando:

```bash
argocd login localhost:8080
```

Perceba que ele irá pedir o usuário e a senha, mas não se preocupe, pois o usuário padrão do ArgoCD é o `admin`, e a senha inicial está armazenada em um secret, então vamos executar o seguinte comando para pegar a senha:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

Extrair a senha de outra forma:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
```

A saída desse comando será algo parecido com isso:

```bash
apiVersion: v1
data:
  password: ZEluSXZua2gtbGpUODhEZQ==
kind: Secret
metadata:
  creationTimestamp: "2023-04-26T13:38:37Z"
  name: argocd-initial-admin-secret
  namespace: argocd
  resourceVersion: "113413"
  uid: c67e091f-76da-4e7e-be8e-2dfab37e2261
type: Opaque
```
Agora vamos decodificar o `password`

```bash
echo  'password' | base64 --decode
```

A saída será a sua senha inicial. Agora você já pode realizar o login.

> **Nota** caso ao decodificar a senha apresente o caractere `%`, desconsidere.




##
A saída será a sua senha inicial, copie ela para que possamos utilizar no próximo comando:

```bash
argocd login localhost:8080
WARNING: server certificate had error: x509: certificate signed by unknown authority. Proceed insecurely (y/n)? y
Username: admin           
Password: 
'admin:login' logged in successfully
Context 'localhost:8080' updated
```

Pronto, estamos autenticados no ArgoCD. Agora vamos adicionar o nosso cluster Kubernetes ao ArgoCD.


Para isso, vamos ver qual o contexto do nosso cluster Kubernetes:

```bash
kubectl config current-context
```

A saída será algo parecido com isso (neste caso, estamos utulizando um cluster local com o [K0S](https://docs.k0sproject.io/v1.21.0+k0s.0/)):

```bash
Default
```

Isso no meu caso que somente estou utilizando um cluster e é um EKS, lá da AWS.

Agora vamos adicionar o nosso cluster ao ArgoCD:

```bash
argocd cluster add O_NOME_DO_SEU_CONTEXT
```

No meu caso:
    
```bash
argocd cluster add Default
``` 

A saída será algo parecido com isso:

```bash
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `Default` with full cluster level privileges. Do you want to continue [y/N]? y
INFO[0001] ServiceAccount "argocd-manager" already exists in namespace "kube-system" 
INFO[0001] ClusterRole "argocd-manager-role" updated    
INFO[0001] ClusterRoleBinding "argocd-manager-role-binding" updated 
Cluster 'https://192.168.1.9:6443' added
```

Caso esteja utilizando um cluster k8s no mesmo host em que está executando o kubectl, como é o que acontece quando usamos um cluster via kind ou minikube por exemplo, você pode ter o seguinte erro:

```bash
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `Default` with full cluster level privileges. Do you want to continue [y/N]? y
INFO[0007] ServiceAccount "argocd-manager" created in namespace "kube-system" 
INFO[0007] ClusterRole "argocd-manager-role" created    
INFO[0007] ClusterRoleBinding "argocd-manager-role-binding" created 
INFO[0012] Created bearer token secret for ServiceAccount "argocd-manager" 
FATA[0013] rpc error: code = Unknown desc = Get "https://localhost:6443/version?timeout=32s": dial tcp [::1]:6443: connect: connection refused
```

Para contornar esse erro execute o comando `kubectl get -n default endpoints`. A saída será algo parecido com isso:

```bash
NAME         ENDPOINTS          AGE
kubernetes   192.168.1.9:6443   21h
```

Agora copie o ip e porta que foi mostrado com a execução do comando anterior e altere somente o valor de endereço do server no seu arquivo `.kube/config`, como no exemplo abaixo onde o ip antigo foi comentado e o novo endereço foi configurado:

```yaml
apiVersion: v1
clusters:
- cluster:
    #server: https://localhost:6443
    server: https://192.168.1.9:6443
  name: local
```

Após essa modificação execute novamente o comando para adicionar o cluster ao ArgoCD

```bash
argocd cluster add O_NOME_DO_SEU_CONTEXT
```

E desta vez a saída sem erro será parecida com isso:

```bash
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `Default` with full cluster level privileges. Do you want to continue [y/N]? y
INFO[0001] ServiceAccount "argocd-manager" already exists in namespace "kube-system" 
INFO[0001] ClusterRole "argocd-manager-role" updated    
INFO[0001] ClusterRoleBinding "argocd-manager-role-binding" updated 
Cluster 'https://192.168.1.9:6443' added
```

Pronto, nosso cluster foi adicionado ao ArgoCD.

Vamos confirmar se o nosso cluster foi adicionado ao ArgoCD:

```bash
argocd cluster list
```

A saída será algo parecido com isso:

```bash
SERVER                          NAME        VERSION  STATUS   MESSAGE                                                  PROJECT
https://192.168.1.9:6443        Default              Unknown  Cluster has no applications and is not being monitored.  
https://kubernetes.default.svc  in-cluster           Unknown  Cluster has no applications and is not being monitored. 
```

Na saída, temos o nosso cluster adicionado, e o cluster local, que é o `in-cluster`, que vem por padrão.

Pronto, já temos onde a nossa aplicação vai ser implantada, agora vamos criar a nossa aplicação para o ArgoCD.
###



## Criando a aplicação no ArgoCD

Agora que já temos o nosso cluster adicionado ao ArgoCD, vamos criar a nossa aplicação. Para isso, temos que ter um repositório Git com o nosso código, e o ArgoCD vai monitorar esse repositório e vai fazer o deploy da nossa aplicação sempre que tiver uma alteração.

### Criando a app no ArgoCD usando o ArgoCD CLI

Já sabemos o que queremos ter em nosso cluster, agora bora criar a nossa aplicação no ArgoCD com o seguinte comando:

```bash
argocd app create wiki.js-app --repo https://github.com/emanuelfds/wiki.js-in-k0s.git --path ./Configs/Wiki.js --dest-server https://192.168.1.9:6443  --dest-namespace default
argocd app create postgre-app --repo https://github.com/emanuelfds/wiki.js-in-k0s.git --path ./Configs/PostgreSQL --dest-server https://192.168.1.9:6443  --dest-namespace wikijs```

  Onde:
* `wiki.js-app` é o nome da nossa aplicação
* `repo` é o repo onde está o nosso código
* `path` é o caminho onde está o nosso código
* `dest-server` é o cluster onde queremos fazer o deploy
* `dest-namespace` é o namespace onde queremos fazer o deploy

A saída do comando será algo como:

```bash
Application 'wiki.js-app' created
Application 'postgre-app' created
```

>**Note**
> Para este exemplo foi utilizado o repositório [Wiki.Js](https://github.com/emanuelfds/wiki.js-in-k0s.git).
> Atentar para a criação do namespace `wikijs` antes de criar as aplicações no ArgoCD (Wiki.js e PostgreSQL). 

### Primeiros passos com o ArgoCD e nossa app

Agora vamos ver se o nosso `Application` foi criado com sucesso:

```bash
argocd app list
```

A saída do comando será algo como:

```bash
NAME                CLUSTER                   NAMESPACE  PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                              PATH                  TARGET
argocd/postgre-app  https://192.168.1.9:6443  wikijs     default  Synced  Healthy  <none>      <none>      https://github.com/emanuelfds/wiki.js-in-k0s.git  ./Configs/PostgreSQL  
argocd/wiki.js-app  https://192.168.1.9:6443  wikijs     default  Synced  Healthy  <none>      <none>      https://github.com/emanuelfds/wiki.js-in-k0s.git  ./Configs/Wiki.js 
```

Agora vamos ver o que está acontecendo com o nosso `Application`:

```bash
argocd app get wikijs-app
```

Ele irá retornar algo como:

```bash
Name:               argocd/wiki.js-app
Project:            default
Server:             https://192.168.1.9:6443
Namespace:          default
URL:                https://localhost:8081/applications/wiki.js-app
Repo:               https://github.com/emanuelfds/wiki.js-in-k0s.git
Target:             
Path:               ./Configs/Wiki.js
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        OutOfSync from  (70bc8ad)
Health Status:      Missing

GROUP  KIND        NAMESPACE  NAME             STATUS     HEALTH   HOOK  MESSAGE
       ConfigMap   wikijs     postgres-config  OutOfSync  Missing        
       Secret      wikijs     postgres-secret  OutOfSync  Missing        
       Service     wikijs     wikijs           OutOfSync  Missing        
apps   Deployment  wikijs     wikijs           OutOfSync  Missing 
```

Duas informações importantes aqui:

* O `Application` está com o status `OutOfSync` e o `Sync Status` também está `OutOfSync`;
* O `Application` está com o status `Missing` e o `Health Status` também está `Missing`.

Precisa de mais alguma informação? Vamos ver o que o ArgoCD está tentando fazer:

```bash
argocd app logs wiki.js-app
```

Ainda não temos nada, pois o ArgoCD ainda não fez nada, pois o nosso `Application` está com o status `OutOfSync` e o `Sync Status` também está `OutOfSync`, então precisamos fazer o sync do nosso `Application` para que o ArgoCD possa fazer o deploy do nosso `Deployment` e `Service`:

>**Note**
> O padrão de sincronização do ArgoCD é de 3 minutos.

```bash
argocd app sync wiki.js-app
```

A saída do comando será algo como:

```bash
TIMESTAMP                  GROUP        KIND   NAMESPACE                  NAME    STATUS    HEALTH        HOOK  MESSAGE
2023-04-15T14:58:57-03:00            Service      wikijs                wikijs  OutOfSync  Missing              
2023-04-15T14:58:57-03:00   apps  Deployment      wikijs                wikijs  OutOfSync  Missing              
2023-04-15T14:58:57-03:00          ConfigMap      wikijs       postgres-config  OutOfSync  Missing              
2023-04-15T14:58:57-03:00             Secret      wikijs       postgres-secret  OutOfSync  Missing              
2023-04-15T14:58:57-03:00             Secret      wikijs       postgres-secret    Synced  Missing              
2023-04-15T14:58:57-03:00          ConfigMap      wikijs       postgres-config    Synced  Missing              
2023-04-15T14:58:57-03:00            Service      wikijs                wikijs    Synced  Healthy              
2023-04-15T14:58:57-03:00             Secret      wikijs       postgres-secret    Synced   Missing              secret/postgres-secret created
2023-04-15T14:58:57-03:00          ConfigMap      wikijs       postgres-config    Synced   Missing              configmap/postgres-config created
2023-04-15T14:58:57-03:00            Service      wikijs                wikijs    Synced   Healthy              service/wikijs created
2023-04-15T14:58:57-03:00   apps  Deployment      wikijs                wikijs  OutOfSync  Missing              deployment.apps/wikijs created
2023-04-15T14:58:57-03:00   apps  Deployment      wikijs                wikijs    Synced  Progressing              deployment.apps/wikijs created

Name:               argocd/wiki.js-app
Project:            default
Server:             https://192.168.1.9:6443
Namespace:          wikijs
URL:                https://localhost:8081/applications/wiki.js-app
Repo:               https://github.com/emanuelfds/wiki.js-in-k0s.git
Target:             
Path:               ./Configs/Wiki.js
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to  (70bc8ad)
Health Status:      Progressing

Operation:          Sync
Sync Revision:      70bc8ad4d925395204b57c54d8a1fee73c8427f6
Phase:              Succeeded
Start:              2023-04-15 14:58:56 -0300 -03
Finished:           2023-04-15 14:58:57 -0300 -03
Duration:           1s
Message:            successfully synced (all tasks run)

GROUP  KIND        NAMESPACE  NAME             STATUS  HEALTH       HOOK  MESSAGE
       Secret      wikijs     postgres-secret  Synced                     secret/postgres-secret created
       ConfigMap   wikijs     postgres-config  Synced                     configmap/postgres-config created
       Service     wikijs     wikijs           Synced  Healthy            service/wikijs created
apps   Deployment  wikijs     wikijs           Synced  Progressing        deployment.apps/wikijs created
```

Aqui ele está informa que o `ConfigMap` foi criado, o `Service` foi criado, o `Secret` foi criado e o `Deployment` foi criado. Parece que está tudo certo, certo? 

Vamos ver se ele criou algo no Kubernetes:

```bash
kubectl get pods -n wikijs
```

Pods criados

```bash
NAME                       READY   STATUS    RESTARTS      AGE
wikijs-7ff8c4b87-qqwp2     1/1     Running   0             18m
postgres-bcc5c45b8-6ssxm   1/1     Running   0             18m
```

Agora vamos ver se o `Service` está funcionando:

```bash
kubectl get svc -n wikijs
```

```bash
NAME       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
wikijs     ClusterIP   10.104.58.175    <none>        3000/TCP   18m
postgres   ClusterIP   10.102.224.130   <none>        5432/TCP   18m
```

Somente falta ver se o `ConfigMap` foi criado:

```bash
kubectl get cm -n wikijs
```

```bash
NAME               DATA   AGE
kube-root-ca.crt   1      18m
postgres-config    4      18m
```
Pronto, tudo funcionando.

Vamos ver novamente o status do nosso `Application`:

```bash
argocd app get wiki.js-app
```

```bash
Name:               argocd/wiki.js-app
Project:            default
Server:             https://192.168.1.9:6443
Namespace:          wikijs
URL:                https://localhost:8081/applications/wiki.js-app
Repo:               https://github.com/emanuelfds/wiki.js-in-k0s.git
Target:             
Path:               ./Configs/Wiki.js
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to  (70bc8ad)
Health Status:      Healthy

GROUP  KIND        NAMESPACE  NAME             STATUS  HEALTH   HOOK  MESSAGE
       Secret      wikijs     postgres-secret  Synced                 secret/postgres-secret created
       ConfigMap   wikijs     postgres-config  Synced                 configmap/postgres-config created
       Service     wikijs     wikijs           Synced  Healthy        service/wikijs created
apps   Deployment  wikijs     wikijs           Synced  Healthy        deployment.apps/wikijs created
```

Tudo está `Synced` e `Healthy`.

E como está os logs?

```bash
argocd app logs wiki.js-app --container wikijs
```

```bash
Loading configuration from /wiki/config.yml... OK
2023-04-16T12:29:26.116Z [MASTER] info: =======================================
2023-04-16T12:29:26.153Z [MASTER] info: = Wiki.js 2.5.298 =====================
2023-04-16T12:29:26.154Z [MASTER] info: =======================================
2023-04-16T12:29:26.155Z [MASTER] info: Initializing...
2023-04-16T12:29:28.907Z [MASTER] info: Using database driver pg for postgres [ OK ]
2023-04-16T12:29:28.913Z [MASTER] info: Connecting to database...
2023-04-16T12:29:33.959Z [MASTER] error: Database Connection Error: EAI_AGAIN undefined:undefined
2023-04-16T12:29:33.960Z [MASTER] warn: Will retry in 3 seconds... [Attempt 1 of 10]
2023-04-16T12:29:36.962Z [MASTER] info: Connecting to database...
2023-04-16T12:29:37.005Z [MASTER] info: Database Connection Successful [ OK ]
2023-04-16T12:29:38.458Z [MASTER] warn: Mail is not setup! Please set the configuration in the administration area!
2023-04-16T12:29:38.655Z [MASTER] info: Loading GraphQL Schema...
2023-04-16T12:29:40.758Z [MASTER] info: GraphQL Schema: [ OK ]
2023-04-16T12:29:41.582Z [MASTER] info: HTTP Server on port: [ 3000 ]
```

O que ele faz é trazer o log do container `wikijs` do pod `wikijs-pod` que foi criado pelo `Deployment` `wikijs-deployment`, mesma coisa que se você tivesse feito:

```bash
kubectl logs -f wikijs-pod -c wikijs
```

Agora vamos relembrar os comandos do ArgoCD:

```bash
argocd login localhost:8080 # Faz o login no ArgoCD
argocd add cluster NOME_DO_SEU_CONTEXT # Adiciona um cluster ao ArgoCD
argocd app create wiki.js-app --repo https://github.com/emanuelfds/wiki.js-in-k0s.git --path ./Configs/Wiki.js --dest-server https://192.168.1.9:6443  --dest-namespace wikijs # Cria o aplicativo
argocd app list # Lista os aplicativos
argocd app get wiki.js-app # Mostra o status do aplicativo
argocd app logs wiki.js-app --container nginx # Mostra os logs do aplicativo
argocd app sync wiki.js-app # Sincroniza o aplicativo com o repositório
```

Ahh.. caso você queira remover a aplicação, basta executar o comando abaixo:

```bash
argocd app delete wiki.js-app
```

A saída do comando será algo como:

```bash
Are you sure you want to delete 'wiki.js-app' and all its resources? [y/n] y
application 'wiki.js-app' deleted
```

E caso você queira remover o cluster, basta executar o comando abaixo:

```bash
argocd cluster rm Default        
```

A saída do comando será algo como:

```bash
Are you sure you want to remove 'Default'? Any Apps deploying to this cluster will go to health status Unknown.[y/n] y
Cluster 'Default' removed
INFO[0005] ClusterRoleBinding "argocd-manager-role-binding" deleted 
INFO[0005] ClusterRole "argocd-manager-role" deleted    
INFO[0005] ServiceAccount "argocd-manager" deleted
```

Validando..

```bash
argocd cluster list
```

```bash
SERVER                          NAME        VERSION  STATUS   MESSAGE                                                  PROJECT
https://kubernetes.default.svc  in-cluster           Unknown  Cluster has no applications and is not being monitored.  
```

Para remover o ArgoCD:

```bash
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

A saída desse comando será algo parecido com isso:

```bash
customresourcedefinition.apiextensions.k8s.io "applications.argoproj.io" deleted
customresourcedefinition.apiextensions.k8s.io "applicationsets.argoproj.io" deleted
customresourcedefinition.apiextensions.k8s.io "appprojects.argoproj.io" deleted
serviceaccount "argocd-application-controller" deleted
serviceaccount "argocd-applicationset-controller" deleted
serviceaccount "argocd-dex-server" deleted
serviceaccount "argocd-notifications-controller" deleted
serviceaccount "argocd-redis" deleted
serviceaccount "argocd-repo-server" deleted
serviceaccount "argocd-server" deleted
role.rbac.authorization.k8s.io "argocd-application-controller" deleted
role.rbac.authorization.k8s.io "argocd-applicationset-controller" deleted
role.rbac.authorization.k8s.io "argocd-dex-server" deleted
role.rbac.authorization.k8s.io "argocd-notifications-controller" deleted
role.rbac.authorization.k8s.io "argocd-server" deleted
clusterrole.rbac.authorization.k8s.io "argocd-application-controller" deleted
clusterrole.rbac.authorization.k8s.io "argocd-server" deleted
rolebinding.rbac.authorization.k8s.io "argocd-application-controller" deleted
rolebinding.rbac.authorization.k8s.io "argocd-applicationset-controller" deleted
rolebinding.rbac.authorization.k8s.io "argocd-dex-server" deleted
rolebinding.rbac.authorization.k8s.io "argocd-notifications-controller" deleted
rolebinding.rbac.authorization.k8s.io "argocd-redis" deleted
rolebinding.rbac.authorization.k8s.io "argocd-server" deleted
clusterrolebinding.rbac.authorization.k8s.io "argocd-application-controller" deleted
clusterrolebinding.rbac.authorization.k8s.io "argocd-server" deleted
configmap "argocd-cm" deleted
configmap "argocd-cmd-params-cm" deleted
configmap "argocd-gpg-keys-cm" deleted
configmap "argocd-notifications-cm" deleted
configmap "argocd-rbac-cm" deleted
configmap "argocd-ssh-known-hosts-cm" deleted
configmap "argocd-tls-certs-cm" deleted
secret "argocd-notifications-secret" deleted
secret "argocd-secret" deleted
service "argocd-applicationset-controller" deleted
service "argocd-dex-server" deleted
service "argocd-metrics" deleted
service "argocd-notifications-controller-metrics" deleted
service "argocd-redis" deleted
service "argocd-repo-server" deleted
service "argocd-server" deleted
service "argocd-server-metrics" deleted
deployment.apps "argocd-applicationset-controller" deleted
deployment.apps "argocd-dex-server" deleted
deployment.apps "argocd-notifications-controller" deleted
deployment.apps "argocd-redis" deleted
deployment.apps "argocd-repo-server" deleted
deployment.apps "argocd-server" deleted
statefulset.apps "argocd-application-controller" deleted
networkpolicy.networking.k8s.io "argocd-application-controller-network-policy" deleted
networkpolicy.networking.k8s.io "argocd-applicationset-controller-network-policy" deleted
networkpolicy.networking.k8s.io "argocd-dex-server-network-policy" deleted
networkpolicy.networking.k8s.io "argocd-notifications-controller-network-policy" deleted
networkpolicy.networking.k8s.io "argocd-redis-network-policy" deleted
networkpolicy.networking.k8s.io "argocd-repo-server-network-policy" deleted
networkpolicy.networking.k8s.io "argocd-server-network-policy" deleted
```

Remover o namespace do ArgoCD

```bash
kubectl delete namespace argocd
```

A saída desse comando será algo parecido com isso:

```bash
namespace "argocd" deleted
```