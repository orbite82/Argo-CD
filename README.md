# Argo-CD
https://learning.codefresh.io/


Deploy Argo CD to Kubernetes
Use the terminal to deploy Argo CD:

```
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/codefresh-contrib/gitops-certification-examples/main/argocd-noauth/install.yaml
```

# Resultado:

```
root@kubernetes-vm:~/workdir# kubectl create namespace argocd
namespace/argocd created
root@kubernetes-vm:~/workdir# kubectl apply -n argocd -f https://raw.githubusercontent.com/codefresh-contrib/gitops-certification-examples/main/argocd-noauth/install.yaml
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
serviceaccount/argocd-application-controller created
serviceaccount/argocd-dex-server created
serviceaccount/argocd-redis created
serviceaccount/argocd-server created
role.rbac.authorization.k8s.io/argocd-application-controller created
role.rbac.authorization.k8s.io/argocd-dex-server created
role.rbac.authorization.k8s.io/argocd-server created
clusterrole.rbac.authorization.k8s.io/argocd-application-controller created
clusterrole.rbac.authorization.k8s.io/argocd-server created
rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
rolebinding.rbac.authorization.k8s.io/argocd-redis created
rolebinding.rbac.authorization.k8s.io/argocd-server created
clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-server created
configmap/argocd-cm created
configmap/argocd-cmd-params-cm created
configmap/argocd-gpg-keys-cm created
configmap/argocd-rbac-cm created
configmap/argocd-ssh-known-hosts-cm created
configmap/argocd-tls-certs-cm created
secret/argocd-secret created
service/argocd-dex-server created
service/argocd-metrics created
service/argocd-redis created
service/argocd-repo-server created
service/argocd-server created
service/argocd-server-metrics created
deployment.apps/argocd-dex-server created
deployment.apps/argocd-redis created
deployment.apps/argocd-repo-server created
deployment.apps/argocd-server created
statefulset.apps/argocd-application-controller created
networkpolicy.networking.k8s.io/argocd-application-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-dex-server-network-policy created
networkpolicy.networking.k8s.io/argocd-redis-network-policy created
networkpolicy.networking.k8s.io/argocd-repo-server-network-policy created
networkpolicy.networking.k8s.io/argocd-server-network-policy created
root@kubernetes-vm:~/workdir#
```

Isso criará um novo namespace, argocd, onde os serviços do Argo CD e os recursos de aplicativos residirão. Para visualizar o deployment digite enter

```
kubectl get pods -n argocd
```

```
root@kubernetes-vm:~/workdir# kubectl get pods -n argocd
NAME                                 READY   STATUS    RESTARTS   AGE
argocd-redis-5b6967fdfc-htvzs        1/1     Running   0          2m36s
argocd-dex-server-5fc596bcdd-ctwwn   1/1     Running   0          2m36s
argocd-repo-server-98598b6c7-x7zwj   1/1     Running   0          2m36s
argocd-application-controller-0      1/1     Running   0          2m35s
argocd-server-5b4b7b868b-9kz54       1/1     Running   0          2m36s
```
Observe que este é um método de instalação simples, sem qualquer autenticação. Em uma configuração real, você provavelmente configuraria o SSO com sua instância ArgoCD.

Por padrão, o Argo CD só é acessível de dentro do cluster. Para expor a interface do usuário aos usuários, você pode utilizar qualquer um dos mecanismos de rede padrão do Kubernetes, como

Entrada (recomendado para produção)
Balanceador de carga (afeta o custo da nuvem)
NodePort (simples, mas não muito flexível)
Para este exercício vamos fazer da maneira mais simples e usar um serviço Nodeport. Em um ambiente de produção, você deve usar um Ingress.

# Expose the Argo CD UI

Sinta-se à vontade para ver o arquivo service.yml na guia Editor. É um recurso de serviço NodePort padrão.

Use o terminal para criá-lo no cluster:

```
kubectl apply -f service.yml
```

```
root@kubernetes-vm:~/workdir# kubectl apply -f service.yml
service/argocd-server-nodeport created
```

Após a criação do serviço, verifique a guia "Argo CD UI" para acessar a interface da Web do Argo CD.

O Argo CD tem um modelo de usuário flexível que também suporta SSO com provedores de identificação populares. Para este exemplo simples, usaremos a conta de administrador padrão.

# Log in the Argo CD UI

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d > admin-pass.txt
```

Apenas para simplificar, desabilitamos a autenticação na interface do usuário do ArgoCD.

Basta usar a guia "ArgoCD UI" para ver a interface Tudo está vazio agora.

Além da interface gráfica do usuário, o Argo CD também vem com uma interface de linha de comando (CLI) que pode ser usada para operações comuns.

A CLI é ótima para trabalho de administração, enquanto a GUI é melhor para inspecionar aplicativos e configurações.

# Argo CD CLI

Step 1
To install the CLI

```
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.1.5/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
```

Step 2
Test that the CLI works by typing

```
argocd help
```

Antes de podermos usar sua CLI, precisamos autenticar em nossa própria instância do Argo CD.

Em um ambiente de produção, você pode ter diferentes opções de autenticação e também alterar a senha de administrador padrão (ou remover completamente a conta de administrador).

# Login with the CLI

Step 1
To authenticate

```
argocd login localhost:30443 --insecure
```

```
root@kubernetes-vm:~/workdir# ls
admin-pass.txt  service.yml
root@kubernetes-vm:~/workdir# cat admin-pass.txt
akYb6ZFoEKUlI2I7root@kubernetes-vm:~/workdir#
```
Usamos a opção insegura porque o Argo CD possui um certificado autoassinado. Em uma instalação de produção você teria um certificado real e esta opção não deveria ser usada.

Faça login como admin e use a senha armazenada em admin-pass.txt. Você pode ver o valor na guia do editor.

Observe que, por motivos de segurança, a senha que você cola não é mostrada no terminal (nem mesmo com asteriscos). Basta colar o valor da senha e pressionar Enter.

```
root@kubernetes-vm:~/workdir# argocd login localhost:30443 --insecure
Username: admin
Password: 
'admin:login' logged in successfully
Context 'localhost:30443' updated
```

Step 2
Run some example commands

```
argocd version
argocd app list
```

```
root@kubernetes-vm:~/workdir# argocd version
argocd: v2.1.5+a8a6fc8
  BuildDate: 2021-10-20T15:16:40Z
  GitCommit: a8a6fc8dda0e26bb1e0b893e270c1128038f5b0f
  GitTreeState: clean
  GoVersion: go1.16.5
  Compiler: gc
  Platform: linux/amd64
argocd-server: v2.1.5+a8a6fc8
  BuildDate: 2021-10-20T15:16:40Z
  GitCommit: a8a6fc8dda0e26bb1e0b893e270c1128038f5b0f
  GitTreeState: clean
  GoVersion: go1.16.5
  Compiler: gc
  Platform: linux/amd64
  Ksonnet Version: v0.13.1
  Kustomize Version: v4.2.0 2021-06-30T22:49:26Z
  Helm Version: v3.6.0+g7f2df64
  Kubectl Version: v0.21.0
  Jsonnet Version: v0.17.0

root@kubernetes-vm:~/workdir# argocd app list
NAME  CLUSTER  NAMESPACE  PROJECT  STATUS  HEALTH  SYNCPOLICY  CONDITIONS  REPO  PATH  TARGET
```

# 02 Create GitOps application

Como o GitOps tem tudo a ver com Git, usaremos o Github para este exercício.

O aplicativo de exemplo está em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/simple-app

Objetivos
Nesta faixa, isso é o que você aprenderá:

Como funciona a interface do usuário do Argo CD
Como funciona a CLI do Argo CD
Como implantar e sincronizar aplicativos
Aguarde enquanto inicializamos a VM para você e iniciamos o Kubernetes.

Bem-vindo
Nosso aplicativo de exemplo pode ser encontrado em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/simple-app

Dê uma olhada nos manifestos do Kubernetes para entender o que vamos implantar. É um aplicativo muito simples com uma implantação e um serviço

# Argo CD User interface

Instalamos o Argo CD para você e você pode fazer login na guia UI.

A interface do usuário começa vazia porque nada é implantado em nosso cluster. Clique no botão "New app" no canto superior esquerdo e preencha os seguintes detalhes:

application name : demo
project: default
repository URL: https://github.com/codefresh-contrib/gitops-certification-examples
path: ./simple-app
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
(este é o mesmo cluster onde o ArgoCD está instalado)
Namespace: default
Deixe todos os outros valores vazios ou com seleções padrão. Por fim, clique no botão Criar. A entrada do aplicativo aparecerá no painel principal. Clique nisso.

Parabéns! Você configurou seu primeiro aplicativo com GitOps.

No entanto, se você verificar seu cluster com:

```
kubectl get deployments
```

Você verá que nada foi implantado ainda. O cluster ainda está vazio. A mensagem "Out of sync"  significa essencialmente que

O cluster está vazio
O repositório Git tem um aplicativo
Portanto, o estado do Git e o estado do cluster são diferentes.

```
root@kubernetes-vm:~/workdir# kubectl get deployments
No resources found in default namespace.
```

```
root@kubernetes-vm:~/workdir# kubectl get deployments
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
simple-deployment   1/1     1            1           13s
```

# Syncing an app

Criamos nosso aplicativo no Argo CD, mas ele ainda não foi implantado, pois só existe no Git no momento. Precisamos dizer ao Argo CD para tornar o estado do Git igual ao estado do cluster.

Visite a guia Argo CD UI e clique em seu aplicativo para ver todos os detalhes. Clique no botão "Sync button" e deixe todas as opções padrão em todas as opções. Por fim, clique no botão "Synchronize" na parte superior.

O ArgoCD vê que o cluster está vazio e irá implantar sua aplicação para que o cluster tenha o mesmo estado que está no Git

nosso aplicativo agora está implantado. Você pode verificar com:

```
kubectl get deployments
```

```
root@kubernetes-vm:~/workdir# kubectl get deployments
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
simple-deployment   1/1     1            1           6m19s
```

e também visite a guia "Deployed App".

```
I am an application running in Kubernetes. Now at Version 1.0
```
