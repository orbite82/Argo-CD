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

Isso criar?? um novo namespace, argocd, onde os servi??os do Argo CD e os recursos de aplicativos residir??o. Para visualizar o deployment digite enter

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
Observe que este ?? um m??todo de instala????o simples, sem qualquer autentica????o. Em uma configura????o real, voc?? provavelmente configuraria o SSO com sua inst??ncia ArgoCD.

Por padr??o, o Argo CD s?? ?? acess??vel de dentro do cluster. Para expor a interface do usu??rio aos usu??rios, voc?? pode utilizar qualquer um dos mecanismos de rede padr??o do Kubernetes, como

Entrada (recomendado para produ????o)
Balanceador de carga (afeta o custo da nuvem)
NodePort (simples, mas n??o muito flex??vel)
Para este exerc??cio vamos fazer da maneira mais simples e usar um servi??o Nodeport. Em um ambiente de produ????o, voc?? deve usar um Ingress.

# Expose the Argo CD UI

Sinta-se ?? vontade para ver o arquivo service.yml na guia Editor. ?? um recurso de servi??o NodePort padr??o.

Use o terminal para cri??-lo no cluster:

```
kubectl apply -f service.yml
```

```
root@kubernetes-vm:~/workdir# kubectl apply -f service.yml
service/argocd-server-nodeport created
```

Ap??s a cria????o do servi??o, verifique a guia "Argo CD UI" para acessar a interface da Web do Argo CD.

O Argo CD tem um modelo de usu??rio flex??vel que tamb??m suporta SSO com provedores de identifica????o populares. Para este exemplo simples, usaremos a conta de administrador padr??o.

# Log in the Argo CD UI

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d > admin-pass.txt
```

Apenas para simplificar, desabilitamos a autentica????o na interface do usu??rio do ArgoCD.

Basta usar a guia "ArgoCD UI" para ver a interface Tudo est?? vazio agora.

Al??m da interface gr??fica do usu??rio, o Argo CD tamb??m vem com uma interface de linha de comando (CLI) que pode ser usada para opera????es comuns.

A CLI ?? ??tima para trabalho de administra????o, enquanto a GUI ?? melhor para inspecionar aplicativos e configura????es.

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

Antes de podermos usar sua CLI, precisamos autenticar em nossa pr??pria inst??ncia do Argo CD.

Em um ambiente de produ????o, voc?? pode ter diferentes op????es de autentica????o e tamb??m alterar a senha de administrador padr??o (ou remover completamente a conta de administrador).

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
Usamos a op????o insegura porque o Argo CD possui um certificado autoassinado. Em uma instala????o de produ????o voc?? teria um certificado real e esta op????o n??o deveria ser usada.

Fa??a login como admin e use a senha armazenada em admin-pass.txt. Voc?? pode ver o valor na guia do editor.

Observe que, por motivos de seguran??a, a senha que voc?? cola n??o ?? mostrada no terminal (nem mesmo com asteriscos). Basta colar o valor da senha e pressionar Enter.

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

Como o GitOps tem tudo a ver com Git, usaremos o Github para este exerc??cio.

O aplicativo de exemplo est?? em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/simple-app

Objetivos
Nesta faixa, isso ?? o que voc?? aprender??:

Como funciona a interface do usu??rio do Argo CD
Como funciona a CLI do Argo CD
Como implantar e sincronizar aplicativos
Aguarde enquanto inicializamos a VM para voc?? e iniciamos o Kubernetes.

Bem-vindo
Nosso aplicativo de exemplo pode ser encontrado em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/simple-app

D?? uma olhada nos manifestos do Kubernetes para entender o que vamos implantar. ?? um aplicativo muito simples com uma implanta????o e um servi??o

# Argo CD User interface

Instalamos o Argo CD para voc?? e voc?? pode fazer login na guia UI.

A interface do usu??rio come??a vazia porque nada ?? implantado em nosso cluster. Clique no bot??o "New app" no canto superior esquerdo e preencha os seguintes detalhes:

application name : demo
project: default
repository URL: https://github.com/codefresh-contrib/gitops-certification-examples
path: ./simple-app
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
(este ?? o mesmo cluster onde o ArgoCD est?? instalado)
Namespace: default
Deixe todos os outros valores vazios ou com sele????es padr??o. Por fim, clique no bot??o Criar. A entrada do aplicativo aparecer?? no painel principal. Clique nisso.

Parab??ns! Voc?? configurou seu primeiro aplicativo com GitOps.

No entanto, se voc?? verificar seu cluster com:

```
kubectl get deployments
```

Voc?? ver?? que nada foi implantado ainda. O cluster ainda est?? vazio. A mensagem "Out of sync"  significa essencialmente que

O cluster est?? vazio
O reposit??rio Git tem um aplicativo
Portanto, o estado do Git e o estado do cluster s??o diferentes.

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

Criamos nosso aplicativo no Argo CD, mas ele ainda n??o foi implantado, pois s?? existe no Git no momento. Precisamos dizer ao Argo CD para tornar o estado do Git igual ao estado do cluster.

Visite a guia Argo CD UI e clique em seu aplicativo para ver todos os detalhes. Clique no bot??o "Sync button" e deixe todas as op????es padr??o em todas as op????es. Por fim, clique no bot??o "Synchronize" na parte superior.

O ArgoCD v?? que o cluster est?? vazio e ir?? implantar sua aplica????o para que o cluster tenha o mesmo estado que est?? no Git

nosso aplicativo agora est?? implantado. Voc?? pode verificar com:

```
kubectl get deployments
```

```
root@kubernetes-vm:~/workdir# kubectl get deployments
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
simple-deployment   1/1     1            1           6m19s
```

e tamb??m visite a guia "Deployed App".

```
I am an application running in Kubernetes. Now at Version 1.0
```

# Argo CD CLI

Passo 1
Al??m da interface do usu??rio, o ArgoCD tamb??m possui uma CLI. J?? instalamos o cli para voc?? e autenticamos na inst??ncia.

Tente os seguintes comandos

```
argocd app list
argocd app get demo
argocd app history demo
```

```
root@kubernetes-vm:~/workdir# argocd app list
NAME  CLUSTER                         NAMESPACE  PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                                PATH          TARGET
demo  https://kubernetes.default.svc  default    default  Synced  Healthy  <none>      <none>      https://github.com/codefresh-contrib/gitops-certification-examples  ./simple-app  HEAD
```

```
root@kubernetes-vm:~/workdir# argocd app get demo
Name:               demo
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                https://localhost:30443/applications/demo
Repo:               https://github.com/codefresh-contrib/gitops-certification-examples
Target:             HEAD
Path:               ./simple-app
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to HEAD (c89b02b)
Health Status:      Healthy

GROUP  KIND        NAMESPACE  NAME               STATUS  HEALTH   HOOK  MESSAGE
       Service     default    simple-service     Synced  Healthy        service/simple-service unchanged
apps   Deployment  default    simple-deployment  Synced  Healthy        deployment.apps/simple-deployment unchanged
```

```
root@kubernetes-vm:~/workdir# argocd app history demo
ID  DATE                           REVISION
0   2022-10-19 20:01:59 +0000 UTC  HEAD (c89b02b)
1   2022-10-19 20:06:12 +0000 UTC  HEAD (c89b02b)
2   2022-10-19 20:06:44 +0000 UTC  HEAD (c89b02b)
3   2022-10-19 20:07:02 +0000 UTC  HEAD (c89b02b)
```

Vamos excluir o aplicativo e implant??-lo novamente, mas desta vez a partir da CLI.

Primeiro exclua o aplicativo

```
argocd app delete demo
```

Confirme a exclus??o respondendo sim no terminal. O aplicativo desaparecer?? do painel do Argo CD ap??s alguns minutos.

Agora implante-o novamente.

```
argocd app create demo2 \
--project default \
--repo https://github.com/codefresh-contrib/gitops-certification-examples \
--path "./simple-app" \
--dest-namespace default \
--dest-server https://kubernetes.default.svc
```



```
root@kubernetes-vm:~/workdir# argocd app history demo
ID  DATE                           REVISION
0   2022-10-19 20:18:59 +0000 UTC  HEAD (c89b02b)
root@kubernetes-vm:~/workdir# argocd app delete demo
Are you sure you want to delete 'demo' and all its resources? [y/n]
y
root@kubernetes-vm:~/workdir# argocd app create demo2 \
> --project default \
> --repo https://github.com/codefresh-contrib/gitops-certification-examples \
> --path "./simple-app" \
> --dest-namespace default \
> --dest-server https://kubernetes.default.svc
application 'demo2' created
```

O aplicativo aparecer?? no painel do ArgoCD.

Agora sincronize com

```
root@kubernetes-vm:~/workdir# argocd app sync demo2
TIMESTAMP                  GROUP        KIND   NAMESPACE                  NAME    STATUS    HEALTH        HOOK  MESSAGE
2022-10-19T20:21:44+00:00            Service     default        simple-service  OutOfSync  Missing              
2022-10-19T20:21:44+00:00   apps  Deployment     default     simple-deployment  OutOfSync  Missing              
2022-10-19T20:21:45+00:00            Service     default        simple-service    Synced  Healthy              
2022-10-19T20:21:45+00:00            Service     default        simple-service    Synced   Healthy              service/simple-service created
2022-10-19T20:21:45+00:00   apps  Deployment     default     simple-deployment  OutOfSync  Missing              deployment.apps/simple-deployment created
2022-10-19T20:21:45+00:00   apps  Deployment     default     simple-deployment    Synced  Progressing              deployment.apps/simple-deployment created

Name:               demo2
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                https://localhost:30443/applications/demo2
Repo:               https://github.com/codefresh-contrib/gitops-certification-examples
Target:             
Path:               ./simple-app
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to  (c89b02b)
Health Status:      Progressing

Operation:          Sync
Sync Revision:      c89b02b34185626d631a1e91e888098e76c568ec
Phase:              Succeeded
Start:              2022-10-19 20:21:44 +0000 UTC
Finished:           2022-10-19 20:21:45 +0000 UTC
Duration:           1s
Message:            successfully synced (all tasks run)

GROUP  KIND        NAMESPACE  NAME               STATUS  HEALTH       HOOK  MESSAGE
       Service     default    simple-service     Synced  Healthy            service/simple-service created
apps   Deployment  default    simple-deployment  Synced  Progressing        deployment.apps/simple-deployment created
```

# 03 Sync both ways

Voc?? precisa ter uma conta do GitHub para este exerc??cio.

O aplicativo de exemplo est?? em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/simple-app. Fork este reposit??rio em sua pr??pria conta

Objetivos
Nesta faixa, isso ?? o que voc?? aprender??:

Como alterar o aplicativo no Git e reimplantar
Como o Argo CD detecta altera????es entre o cluster e o Git
Como fazer altera????es no cluster e ver o diff
Aguarde enquanto inicializamos a VM para voc?? e iniciamos o Kubernetes.

Bem-vindo
Nosso aplicativo de exemplo pode ser encontrado em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/simple-app

Certifique-se de bifurc??-lo para sua pr??pria conta e n??o para o URL. Deve ser algo como:

https://github.com/<seu usu??rio>/gitops-certification-examples/

D?? uma olhada nos manifestos do Kubernetes para entender o que vamos implantar. ?? um aplicativo muito simples com uma implanta????o e um servi??o

Quando estiver pronto para prosseguir, pressione Avan??ar.

https://github.com/orbite82/gitops-certification-examples

Primeira implanta????o
Instalamos o Argo CD para voc?? e voc?? pode fazer login na guia UI.

A interface do usu??rio come??a vazia porque nada ?? implantado em nosso cluster. Clique no bot??o "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes:

nome do aplicativo: demonstra????o
projeto: padr??o
URL do reposit??rio: https://github.com/<seu usu??rio>/gitops-certification-examples
caminho: ./simple-app
Cluster: https://kubernetes.default.svc (este ?? o mesmo cluster onde o ArgoCD est?? instalado)
Espa??o de nomes: padr??o
Deixe todos os outros valores vazios ou com sele????es padr??o. Por fim, clique no bot??o Criar. A entrada do aplicativo aparecer?? no painel principal. Clique nisso.

Voc?? deve ver o seguinte.

Agora sincronize o aplicativo para garantir que ele seja implantado. Voc?? deve ver o seguinte:

You can also verify the application from the command line with:

```
root@kubernetes-vm:~/workdir# kubectl get deployments
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
simple-deployment   1/1     1            1           26s
```

# Deploy a new version

Queremos implantar outra vers??o do nosso aplicativo. Vamos alterar o Git e ver como o Argo CD detecta a altera????o.

Execute um commit em seu arquivo simple-app/deployment.yml em sua pr??pria conta (voc?? pode usar o GitHub em outra guia para isso) e altere a tag do cont??iner na linha 18 de v1.0 para v2.0

Normalmente, o Argo CD verifica o estado entre o Git e o cluster a cada 3 minutos por conta pr??pria. Apenas para acelerar as coisas, voc?? deve clicar manualmente no aplicativo no painel do Argo CD e pressionar o bot??o "Atualizar"

Voc?? deve ver o seguinte.


Aqui o Argo CD nos diz que o estado do Git n??o ?? mais o mesmo que o estado do cluster. Nas setas amarelas, voc?? pode ver que de todos os componentes do Kubernetes a implanta????o agora ?? diferente. E assim toda a aplica????o ?? diferente.

Voc?? pode clicar no bot??o "App diff" e ver a mudan??a exata. Ative a caixa de sele????o "compact diff".

# Detecting cluster changes

Passo 1
Vamos trazer o cluster de volta ao mesmo estado do Git. Clique no bot??o Sincronizar na interface do usu??rio para sincronizar o aplicativo com a nova vers??o.

Voc?? tamb??m pode executar o seguinte na CLI:

??????
argocd app sync demo
argocd app wait demo
```

```
root@kubernetes-vm:~/workdir# argocd app sync demo
TIMESTAMP                  GROUP        KIND   NAMESPACE                  NAME    STATUS    HEALTH        HOOK  MESSAGE
2022-10-19T20:44:18+00:00            Service     default        simple-service    Synced   Healthy              
2022-10-19T20:44:18+00:00   apps  Deployment     default     simple-deployment  OutOfSync  Healthy              

2022-10-19T20:44:18+00:00            Service     default        simple-service    Synced   Healthy              service/simple-service unchanged
2022-10-19T20:44:18+00:00   apps  Deployment     default     simple-deployment  OutOfSync  Healthy              deployment.apps/simple-deployment unchanged

Name:               demo
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                https://localhost:30443/applications/demo
Repo:               https://github.com/orbite82/gitops-certification-examples
Target:             HEAD
Path:               ./simple-app
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to HEAD (4bf6b92)
Health Status:      Healthy

Operation:          Sync
Sync Revision:      4bf6b92050b78545104b2134bdf9b008a1e5ca89
Phase:              Succeeded
Start:              2022-10-19 20:44:18 +0000 UTC
Finished:           2022-10-19 20:44:18 +0000 UTC
Duration:           0s
Message:            successfully synced (all tasks run)

GROUP  KIND        NAMESPACE  NAME               STATUS  HEALTH   HOOK  MESSAGE
       Service     default    simple-service     Synced  Healthy        service/simple-service unchanged
apps   Deployment  default    simple-deployment  Synced  Healthy        deployment.apps/simple-deployment unchanged
root@kubernetes-vm:~/workdir# argocd app wait demo

Name:               demo
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                https://localhost:30443/applications/demo
Repo:               https://github.com/orbite82/gitops-certification-examples
Target:             HEAD
Path:               ./simple-app
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to HEAD (4bf6b92)
Health Status:      Healthy

Operation:          Sync
Sync Revision:      4bf6b92050b78545104b2134bdf9b008a1e5ca89
Phase:              Succeeded
Start:              2022-10-19 20:44:18 +0000 UTC
Finished:           2022-10-19 20:44:18 +0000 UTC
Duration:           0s
Message:            successfully synced (all tasks run)

GROUP  KIND        NAMESPACE  NAME               STATUS  HEALTH   HOOK  MESSAGE
       Service     default    simple-service     Synced  Healthy        service/simple-service unchanged
apps   Deployment  default    simple-deployment  Synced  Healthy        deployment.apps/simple-deployment unchanged

??????
Detectar mudan??as no Git e aplicar ?? um cen??rio bem conhecido. A grande for??a do GitOps ?? que o Argo CD tamb??m funciona na dire????o oposta. Se voc?? fizer alguma altera????o no cluster, o Argo CD a detectar?? e novamente informar?? que algo est?? diferente entre o Git e seu cluster.

Digamos que algu??m altere manualmente as r??plicas da implanta????o sem criar um Pull Request oficial (uma pr??tica ruim em geral).

Execute o seguinte

```
root@kubernetes-vm:~/workdir# kubectl scale --replicas=3 deployment simple-deployment
deployment.apps/simple-deployment scaled
```

Normalmente, o Argo CD verifica o estado entre o Git e o cluster a cada 3 minutos por conta pr??pria. Apenas para acelerar as coisas, voc?? deve clicar manualmente no aplicativo no painel do Argo CD e pressionar o bot??o "Atualizar". Clique no bot??o "App diff" e ative a caixa de sele????o "Compact Diff".

Voc?? deve ver o seguinte:

O Argo CD detecta novamente a mudan??a entre os dois estados. Esse recurso ?? muito poderoso e voc?? pode detectar facilmente o desvio de configura????o entre seus ambientes.

Terminar
Clique em Verificar para terminar esta faixa.

# 04 Sync strategies

Bem-vindo
Nosso aplicativo de exemplo pode ser encontrado em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/sync-strategies

Certifique-se de bifurc??-lo para sua pr??pria conta e anote o URL. Deve ser algo como:

https://github.com/<your user>/gitops-certification-examples/

D?? uma olhada nos manifestos do Kubernetes para entender o que vamos implantar. ?? um aplicativo muito simples com uma implanta????o e um servi??o

Quando estiver pronto para prosseguir, pressione Avan??ar.

# Auto-sync

Instalamos o Argo CD para voc?? e voc?? pode fazer login na guia UI.

A interface do usu??rio come??a vazia porque nada ?? implantado em nosso cluster. Clique no bot??o "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes:

application name : demo
project: default
SYNC POLICY: automatic
repository URL: https://github.com/<your user>/gitops-certification-examples
path: ./sync-strategies
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: default

Deixe todos os outros valores vazios ou com sele????es padr??o. Por fim, clique no bot??o Criar. A entrada do aplicativo aparecer?? no painel principal. Clique nisso.

Como selecionamos a estrat??gia de sincroniza????o autom??tica, o Argo CD implantar?? automaticamente o aplicativo imediatamente (porque o estado do cluster n??o ?? o mesmo que o estado de confirma????o)

Voc?? tamb??m pode verificar o aplicativo na linha de comando com:

```
kubectl get deployments
```

```
root@kubernetes-vm:~/workdir# kubectl get deployments
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
simple-deployment   1/1     1            1           65s
```

# Deploy a new version with AutoSync

Se voc?? olhar para a guia "Aplicativo implantado", ver?? que nosso aplicativo est?? na vers??o 1.0

Queremos implantar outra vers??o do nosso aplicativo. Vamos alterar o Git e ver como o Argo CD detecta a mudan??a e implanta automaticamente (j?? que definimos a estrat??gia de sincroniza????o como autom??tica).

Execute uma confirma????o em seu arquivo sync-strategies/deployment.yml em sua pr??pria conta (voc?? pode usar o GitHub em outra guia para isso) e altere a tag do cont??iner na linha 18 de v1.0 para v2.0

Normalmente, o Argo CD verifica o estado entre o Git e o cluster a cada 3 minutos por conta pr??pria. Apenas para acelerar as coisas, voc?? deve clicar manualmente no aplicativo no painel do Argo CD e pressionar o bot??o "Atualizar"

Voc?? deve ver o seguinte.

O aplicativo j?? est?? implantado. Se voc?? clicar no bot??o "atualizar" no canto superior direito da guia "Aplicativo implantado", ver?? que a vers??o 2 j?? est?? ativa.

Quando estiver pronto para prosseguir, pressione Verificar.

# Enabling self-heal

Passo 1
A sincroniza????o autom??tica implantar?? automaticamente seu aplicativo assim que o reposit??rio Git for alterado. Mas se algu??m fizer uma altera????o manual no estado do cluster, o Argo CD n??o far?? nada por padr??o (ele ainda marcar?? o aplicativo como fora de sincronia).

Voc?? pode habilitar a op????o de autorrecupera????o para informar ao Argo CD para descartar quaisquer altera????es feitas manualmente no cluster. Essa ?? uma grande vantagem, pois sempre torna seus ambientes ?? prova de balas contra altera????es ad-hoc via kubectl.

Execute o seguinte

```
kubectl scale --replicas=3 deployment simple-deployment
```

```
root@kubernetes-vm:~/workdir# kubectl scale --replicas=3 deployment simple-deployment
deployment.apps/simple-deployment scaled
```

Voc?? alterou manualmente o cluster. Agora v?? para o painel do Argo CD e clique no aplicativo. Na tela do aplicativo, clique no bot??o superior esquerdo "Detalhes do aplicativo".

Role para baixo e encontre a "se????o de pol??tica de sincroniza????o". Clique no bot??o "Self-heal" para habilit??-lo. Responda ok para a caixa de di??logo de confirma????o.

A altera????o manual ser?? descartada e as r??plicas voltar??o a ser uma.

Agora voc?? pode tentar alterar qualquer coisa no cluster e todas as altera????es sempre ser??o descartadas (pois n??o fazem parte do Git).

Tente novamente no terminal.

```
kubectl scale --replicas=3 deployment simple-deployment
kubectl get deployment simple-deployment
```

```
root@kubernetes-vm:~/workdir# kubectl scale --replicas=3 deployment simple-deployment
deployment.apps/simple-deployment scaled
root@kubernetes-vm:~/workdir# kubectl get deployment simple-deployment
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
simple-deployment   1/1     1            1           10m
```
Voc?? alterou manualmente o cluster. Agora v?? para o painel do Argo CD e clique no aplicativo. Na tela do aplicativo, clique no bot??o superior esquerdo "Detalhes do aplicativo".

Role para baixo e encontre a "se????o de pol??tica de sincroniza????o". Clique no bot??o "Self-heal" para habilit??-lo. Responda ok para a caixa de di??logo de confirma????o.

A altera????o manual ser?? descartada e as r??plicas voltar??o a ser uma.

Agora voc?? pode tentar alterar qualquer coisa no cluster e todas as altera????es sempre ser??o descartadas (pois n??o fazem parte do Git).

Tente novamente no terminal.

```
root@kubernetes-vm:~/workdir# kubectl scale --replicas=3 deployment simple-deployment
deployment.apps/simple-deployment scaled
root@kubernetes-vm:~/workdir# kubectl get deployment simple-deployment
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
simple-deployment   3/3     3            3           5m2s
```

Seu aplicativo sempre ter?? 1 pod implantado. Isso ?? o que o estado do Git diz e, portanto, o Argo CD automaticamente torna o estado do cluster o mesmo.

Terminar
Quando estiver pronto para prosseguir, pressione Verificar.

# Enable auto-prune

Mesmo depois de ativar a sincroniza????o autom??tica e a autorrecupera????o, o Argo CD nunca excluir?? recursos do cluster se voc?? remov??-los no Git.

Para habilitar esse comportamento, voc?? precisa habilitar a op????o de remo????o autom??tica.

Primeiro, fa??a um commit em seu reposit??rio Git excluindo o arquivo sync-strategies/deployment.yml.

Em seguida, clique no bot??o "Atualizar" no painel do ArgoCD. O ArgoCD detectar?? a altera????o e marcar?? o aplicativo como "Fora de sincronia". Mas a implanta????o ainda estar?? l??.

Voc?? deve ver o seguinte.

Voc?? ainda pode ver a implanta????o com

```
kubectl get deployment simple-deployment
```

```
root@kubernetes-vm:~/workdir# kubectl get deployment simple-deployment
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
simple-deployment   1/1     1            1           10m
```

V?? ao painel do Argo CD e clique no aplicativo. Na tela do aplicativo, clique no bot??o superior esquerdo "Detalhes do aplicativo".

Role para baixo e encontre a "se????o de pol??tica de sincroniza????o". Clique no bot??o "Remover recursos" para habilit??-lo. Responda ok para a caixa de di??logo de confirma????o.

Feche a janela de configura????es e clique novamente no bot??o "Atualizar" no aplicativo.

Agora sua implanta????o ser?? removida (j?? que ela n??o existe no Git).

Verifique novamente:

```
kubectl get deployment
```

```
root@kubernetes-vm:~/workdir# kubectl get deployment
No resources found in default namespace.
```

# 05 Secrets with GitOps

Bem-vindo
Nosso aplicativo de exemplo pode ser encontrado em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/secret-app

Certifique-se de bifurc??-lo para sua pr??pria conta e anote o URL. Deve ser algo como:

https://github.com/<seu usu??rio>/gitops-certification-examples/

D?? uma olhada nos manifestos do Kubernetes em secret-app/manifests/ para entender o que vamos implantar.

?? um aplicativo muito simples com uma implanta????o e um servi??o que tamb??m precisa de dois arquivos de segredos (para conex??o db e certificado paypal) que s??o passados ??????para o aplicativo como arquivos em /secrets/.

Os dois segredos s??o armazenados nos arquivos db-creds-encrypted.yaml e paypal-cert-encrypted.yaml Esses arquivos est??o vazios no reposit??rio Git porque queremos criptograf??-los primeiro.

Voc?? tamb??m pode ver o c??digo-fonte completo em source-code-secrets se quiser entender como o aplicativo est?? carregando segredos de arquivos.

Quando estiver pronto para prosseguir, pressione Avan??ar.

# Install the Sealed Secrets controller

Colocamos os segredos brutos/n??o criptografados em seu diret??rio de trabalho que s??o necess??rios para o aplicativo. Voc?? pode ver os arquivos na guia Editor.

Esses segredos s??o segredos simples do Kubernetes e n??o s??o criptografados de forma alguma. Vamos instalar o controlador de segredos selados da Bitnami para criptografar nossos segredos.

Instalamos o Argo CD para voc?? e voc?? pode fazer login na guia UI.

A interface do usu??rio come??a vazia porque nada ?? implantado em nosso cluster. Clique no bot??o "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes:

Deixe todos os outros valores vazios ou com sele????es padr??o. Por fim, clique no bot??o Criar. O controlador ser?? instalado no cluster.

Observe que est??o usando um namespace espec??fico para o controlador e n??o o padr??o. ?? imperativo entrar no sistema kube, caso contr??rio, o controlador de segredos n??o funcionar??.

Voc?? pode ver que o GitOps n??o ?? ??til apenas para seus pr??prios aplicativos, mas tamb??m para outros aplicativos de suporte.

Quando estiver pronto para prosseguir, pressione Verificar.

# Use Kubeseal to encrypt secretsb

O controlador Sealed Secrets est?? em execu????o agora no cluster e est?? pronto para descriptografar segredos.

Agora precisamos criptografar nossos segredos e envi??-los para o git. A criptografia acontece com o execut??vel kubeseal. Ele precisa ser instalado da mesma maneira que o kubectl. Ele reutiliza a autentica????o de cluster j?? usada pelo kubectl.

J?? instalamos o kubeseal para voc?? neste exerc??cio. Voc?? pode us??-lo imediatamente para criptografar seus segredos simples do Kubernetes e convert??-los em segredos criptografados

Execute o seguinte

```
kubeseal < unsealed_secrets/db-creds.yml > sealed_secrets/db-creds-encrypted.yaml -o yaml
kubeseal < unsealed_secrets/paypal-cert.yml > sealed_secrets/paypal-cert-encrypted.yaml -o yaml
```

Agora voc?? tem segredos criptografados. Abra os arquivos na guia "Editor" e copie o conte??do em sua ??rea de transfer??ncia.

Em seguida, v?? para a interface do usu??rio do Github em outra guia do navegador e confirme/envie seu conte??do em seu pr??prio fork do aplicativo, preenchendo os arquivos vazios em gitops-certification-examples/secret-app/manifests/db-creds-encrypted.yaml e gitops- certificate-examples/secret-app/manifests/paypal-cert-encrypted.yaml

Quando estiver pronto para prosseguir, pressione Verificar.

# Deploy secrets

Passo 1
Agora que todos os nossos segredos est??o no Git de forma criptografada, podemos implantar nosso aplicativo normalmente.

Fa??a login na interface do usu??rio do Argo CD.

Clique no bot??o "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes:

application name : demo
project: default
SYNC POLICY: automatic
repository URL: https://github.com/your_github_account/gitops-certification-examples
path: ./secret-app/manifests
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: default

# Bem-vindo

At?? agora, em todos os desafios anteriores, voc?? criou aplicativos por meio da interface do usu??rio do ArgoCD ou CLI. Um aplicativo ArgoCD ?? essencialmente uma combina????o de um reposit??rio git, um projeto Argo, v??rias op????es de sincroniza????o e outros valores.

Esta informa????o n??o precisa ser confinada no pr??prio Argo CD. Ele pode ser modelado em um recurso do Kubernetes para que possa ser armazenado no Git e gerenciado com o GitOps tamb??m.

D?? uma olhada em https://github.com/codefresh-contrib/gitops-certification-examples/blob/main/declarative/single-app/my-application.yml para um exemplo de aplicativo

Quando estiver pronto para prosseguir, pressione Avan??ar.

# ArgoCD declarative format

Como nosso aplicativo ArgoCD agora ?? um recurso do Kubernetes, podemos trat??-lo como qualquer outro recurso do Kubernetes.

Verificamos para voc?? localmente o arquivo my-application.yml Aplique-o com

```
kubectl apply -f my-application.yml
```

```
root@kubernetes-vm:~/workdir# kubectl apply -f my-application.yml
application.argoproj.io/demo created
```

Tamb??m instalamos o Argo CD para voc?? e voc?? o v?? na guia UI.

Agora voc?? deve ver o aplicativo j?? na interface do usu??rio do ArgoCD.

Quando estiver pronto para prosseguir, pressione Verificar.

# Use App of Apps

Podemos excluir o aplicativo como qualquer outro recurso do Kubernetes

```
kubectl delete -f my-application.yml
```

```
root@kubernetes-vm:~/workdir# kubectl delete -f my-application.yml
application.argoproj.io "demo" deleted
```

After a while the application should disappear from the ArgoCD UI.

Remember however that the whole point of GitOps is to avoid manual kubectl commands. We want to store this application manifest in Git. And if we store it in Git, we can handle it like another GitOps application!

ArgoCD can manage any kind of Kubernetes resources and this includes its own applications (inception).

So we can commit multiple applications in Git and pass them to ArgoCD like any other kind of manifest. This means that we can handle multiple applications as a single one.

We already have such example at https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/declarative/multiple-apps.

In the ArgoCD UI, click the "New app" button on the top left and fill the following details:

application name : 3-apps
project: default
SYNC POLICY: automatic
repository URL: https://github.com/codefresh-contrib/gitops-certification-examples
path: ./declarative/multiple-apps
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: argocd

Observe que o valor do namespace ?? o namespace no qual o aplicativo pai ?? implantado e n??o o namespace no qual os aplicativos individuais s??o implantados. Os aplicativos ArgoCD devem ser implantados no mesmo namespace que o pr??prio ArgoCD.

Deixe todos os outros valores vazios ou com sele????es padr??o. Por fim, clique no bot??o Criar.

O ArgoCD implantar?? o aplicativo pai e seus 3 filhos. Clique no aplicativo pai. Voc?? deve ver o seguinte.

Passe algum tempo na interface do usu??rio para entender onde cada aplicativo ?? implantado. Voc?? tamb??m pode usar a linha de comando

```
kubectl get all -n demo1
kubectl get all -n demo2
kubectl get all -n demo3
```

```
root@kubernetes-vm:~/workdir# kubectl get all -n demo1
NAME                                    READY   STATUS    RESTARTS   AGE
pod/simple-deployment-9d88c574d-sbcrx   1/1     Running   0          80s

NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/simple-service   ClusterIP   10.43.253.166   <none>        80/TCP    80s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/simple-deployment   1/1     1            1           80s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/simple-deployment-9d88c574d   1         1         1       80s
root@kubernetes-vm:~/workdir# kubectl get all -n demo2
NAME                                    READY   STATUS    RESTARTS   AGE
pod/simple-deployment-9d88c574d-q5xj6   1/1     Running   0          80s

NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/simple-service   ClusterIP   10.43.11.142   <none>        80/TCP    80s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/simple-deployment   1/1     1            1           80s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/simple-deployment-9d88c574d   1         1         1       80s
root@kubernetes-vm:~/workdir# kubectl get all -n demo3
NAME                                    READY   STATUS    RESTARTS   AGE
pod/simple-deployment-9d88c574d-hqhn4   1/1     Running   0          83s

NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/simple-service   ClusterIP   10.43.231.14   <none>        80/TCP    83s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/simple-deployment   1/1     1            1           83s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/simple-deployment-9d88c574d   1         1         1       83s
```

# 07 Using Helm with ArgoCD

Bem-vindo
Nosso aplicativo de exemplo pode ser encontrado em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/helm-app

D?? uma olhada nos gr??ficos do Helm para entender o que vamos implantar. ?? um aplicativo muito simples com uma implanta????o e um servi??o. O modelo do Ingress no gr??fico est?? desabilitado por padr??o.

Quando estiver pronto para prosseguir, pressione Avan??ar.

# Using the ArgoCD GUI

Instalamos o Argo CD para voc?? e voc?? pode fazer login na guia UI.

A interface do usu??rio come??a vazia porque nada ?? implantado em nosso cluster. Clique no bot??o "NOVO APP" no canto superior esquerdo e preencha os seguintes detalhes:

application name : helm-gitops-example
project: default
sync policy: automatic
repository URL: https://github.com/codefresh-contrib/gitops-certification-examples
path: ./helm-app/
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: default

Deixe todos os outros valores vazios ou com sele????es padr??o. Observe que, no caso de gr??ficos do Helm, voc?? tamb??m pode substituir valores espec??ficos na parte inferior da caixa de di??logo. Isso ?? totalmente opcional, pois nosso gr??fico Helm j?? possui o arquivo values.yaml correto.

Por fim, clique no bot??o Criar. A entrada do aplicativo aparecer?? no painel principal. Clique nisso.

Parab??ns! Voc?? configurou seu primeiro aplicativo Helm com o GitOps. Voc?? deve ver o seguinte:

Voc?? pode verificar a implanta????o com

```
kubectl get deployment
```

```
root@kubernetes-vm:~/workdir# kubectl get deployment
NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
helm-gitops-example-helm-example   1/1     1            1           39s
```

Observe que, se voc?? usar qualquer um dos comandos do leme, como a lista do leme, n??o ver?? nada. Um gr??fico do Helm implantado pelo ArgoCD n??o ?? mais registrado como uma implanta????o do Helm. Isso ocorre porque o ArgoCD n??o inclui as informa????es de carga ??til do Helm. Ao implantar um aplicativo Helm, o Argo CD executa o "modelo de leme" e implanta os manifestos resultantes.

# Using ArgoCD CLI

Passo 1
Al??m da interface do usu??rio, o ArgoCD tamb??m possui uma CLI. J?? instalamos o cli para voc?? e autenticamos na inst??ncia.

Tente os seguintes comandos

```
argocd app list
argocd app get helm-gitops-example
argocd app history helm-gitops-example
```

```
root@kubernetes-vm:~/workdir# argocd app list
NAME                 CLUSTER                         NAMESPACE  PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                                PATH         TARGET
helm-gitops-example  https://kubernetes.default.svc  default    default  Synced  Healthy  Auto        <none>      https://github.com/codefresh-contrib/gitops-certification-examples  ./helm-app/  HEAD

root@kubernetes-vm:~/workdir# argocd app get helm-gitops-example
Name:               helm-gitops-example
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                https://localhost:30443/applications/helm-gitops-example
Repo:               https://github.com/codefresh-contrib/gitops-certification-examples
Target:             HEAD
Path:               ./helm-app/
SyncWindow:         Sync Allowed
Sync Policy:        Automated
Sync Status:        Synced to HEAD (c89b02b)
Health Status:      Healthy

GROUP  KIND            NAMESPACE  NAME                              STATUS  HEALTH   HOOK  MESSAGE
       ServiceAccount  default    helm-gitops-example-helm-example  Synced                 serviceaccount/helm-gitops-example-helm-example created
       Service         default    helm-gitops-example-helm-example  Synced  Healthy        service/helm-gitops-example-helm-example created
apps   Deployment      default    helm-gitops-example-helm-example  Synced  Healthy        deployment.apps/helm-gitops-example-helm-example created

root@kubernetes-vm:~/workdir# argocd app history helm-gitops-example
ID  DATE                           REVISION
0   2022-10-20 19:12:50 +0000 UTC  HEAD (c89b02b)
```
Vamos excluir o aplicativo e implant??-lo novamente, mas desta vez a partir da CLI.

Primeiro exclua o aplicativo

```
argocd app delete helm-gitops-example
```

```
root@kubernetes-vm:~/workdir# argocd app delete helm-gitops-example
Are you sure you want to delete 'helm-gitops-example' and all its resources? [y/n]
y
```

Confirme a exclus??o respondendo sim no terminal. O aplicativo desaparecer?? do painel do Argo CD ap??s alguns minutos.

Agora implante-o novamente.

```
root@kubernetes-vm:~/workdir# argocd app create demo \
> --project default \
> --repo https://github.com/codefresh-contrib/gitops-certification-examples \
> --path "./helm-app/" \
> --sync-policy auto \
> --dest-namespace default \
> --dest-server https://kubernetes.default.svc
application 'demo' created
```

O aplicativo aparecer?? novamente no painel do ArgoCD.

Confirme a implanta????o

```
kubectl get all
```

```
root@kubernetes-vm:~/workdir# kubectl get all
NAME                                     READY   STATUS    RESTARTS   AGE
pod/demo-helm-example-5546f55d74-27gs9   1/1     Running   0          43s

NAME                        TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes          ClusterIP   10.43.0.1    <none>        443/TCP   10m
service/demo-helm-example   ClusterIP   10.43.23.0   <none>        80/TCP    43s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/demo-helm-example   1/1     1            1           43s

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/demo-helm-example-5546f55d74   1         1         1       43s
```

Terminar
Se voc?? confirmou a implanta????o, clique em Verificar para concluir esta faixa.


# 08 Using Kustomize with ArgoCD

Bem-vindo
Nosso aplicativo de exemplo pode ser encontrado em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/kustomize-app

D?? uma olhada nas pastas base e overlays para entender o que vamos implantar. ?? um aplicativo muito simples com 2 ambientes para implanta????o: prepara????o e produ????o.

Os ambientes diferem em v??rios aspectos como n??mero de r??plicas, banco de dados utilizado, rede etc.

Quando estiver pronto para prosseguir, pressione Avan??ar.

# Using the ArgoCD GUI

Instalamos o Argo CD para voc?? e voc?? pode fazer login na guia UI.

A interface do usu??rio come??a vazia porque nada ?? implantado em nosso cluster. Clique no bot??o "NOVO APP" no canto superior esquerdo e preencha os seguintes detalhes:
Abrir no Google T

application name : kustomize-gitops-example
project: default
sync policy: automatic
repository URL: https://github.com/codefresh-contrib/gitops-certification-examples
path: ./kustomize-app/overlays/staging
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: default

Deixe todos os outros valores vazios ou com sele????es padr??o. Por fim, clique no bot??o Criar. A entrada do aplicativo aparecer?? no painel principal. Clique nisso.

Parab??ns! Voc?? configurou seu primeiro aplicativo Kustomize com GitOps.

Voc?? deve ver o seguinte.

Aguarde algum tempo para o aplicativo iniciar e, em seguida, visite o "Aplicativo implantado" para v??-lo em execu????o. Voc?? ver?? que ele carregou o valor de teste para o banco de dados.

# Using ArgoCD CLI

Passo 1
Al??m da interface do usu??rio, o ArgoCD tamb??m possui uma CLI. J?? instalamos o cli para voc?? e autenticamos na inst??ncia.

Tente os seguintes comandos

```
argocd app list
argocd app get kustomize-gitops-example
argocd app history kustomize-gitops-example
```

```
root@kubernetes-vm:~/workdir# argocd app list
NAME                      CLUSTER                         NAMESPACE  PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                                PATH                              TARGET
kustomize-gitops-example  https://kubernetes.default.svc  default    default  Synced  Healthy  Auto        <none>      https://github.com/codefresh-contrib/gitops-certification-examples  ./kustomize-app/overlays/staging  HEAD
root@kubernetes-vm:~/workdir# argocd app get kustomize-gitops-example
Name:               kustomize-gitops-example
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                https://localhost:30443/applications/kustomize-gitops-example
Repo:               https://github.com/codefresh-contrib/gitops-certification-examples
Target:             HEAD
Path:               ./kustomize-app/overlays/staging
SyncWindow:         Sync Allowed
Sync Policy:        Automated
Sync Status:        Synced to HEAD (c89b02b)
Health Status:      Healthy

GROUP  KIND        NAMESPACE  NAME                    STATUS  HEALTH   HOOK  MESSAGE
       ConfigMap   default    staging-the-map         Synced                 configmap/staging-the-map created
       Service     default    staging-demo            Synced  Healthy        service/staging-demo created
apps   Deployment  default    staging-the-deployment  Synced  Healthy        deployment.apps/staging-the-deployment created
root@kubernetes-vm:~/workdir# argocd app history kustomize-gitops-example
ID  DATE                           REVISION
0   2022-10-20 20:16:08 +0000 UTC  HEAD (c89b02b)
```

Vamos excluir o aplicativo e implant??-lo novamente, mas desta vez a partir da CLI.

Primeiro exclua o aplicativo

```
argocd app delete kustomize-gitops-example
```

```
root@kubernetes-vm:~/workdir# argocd app delete kustomize-gitops-example
Are you sure you want to delete 'kustomize-gitops-example' and all its resources? [y/n]
y
```

Confirme a exclus??o respondendo sim no terminal. O aplicativo desaparecer?? do painel do Argo CD ap??s alguns minutos.

Agora implante-o novamente.

```
argocd app create demo \
--project default \
--repo https://github.com/codefresh-contrib/gitops-certification-examples \
--path ./kustomize-app/overlays/staging \
--sync-policy auto \
--dest-namespace default \
--dest-server https://kubernetes.default.svc
```

```
root@kubernetes-vm:~/workdir# argocd app create demo \
> --project default \
> --repo https://github.com/codefresh-contrib/gitops-certification-examples \
> --path ./kustomize-app/overlays/staging \
> --sync-policy auto \
> --dest-namespace default \
> --dest-server https://kubernetes.default.svc
application 'demo' created
```

O aplicativo aparecer?? no painel do ArgoCD.

Confirme a implanta????o

```
root@kubernetes-vm:~/workdir# kubectl get all
NAME                                          READY   STATUS    RESTARTS   AGE
pod/staging-the-deployment-85c5c7685f-qvg9v   1/1     Running   0          38s

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes     ClusterIP   10.43.0.1       <none>        443/TCP          11m
service/staging-demo   NodePort    10.43.220.101   <none>        8080:31000/TCP   38s

NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/staging-the-deployment   1/1     1            1           38s

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/staging-the-deployment-85c5c7685f   1         1         1       38s
```

Terminar
Se voc?? confirmou a implanta????o, clique em Verificar para concluir esta faixa.

# 09 Blue/Green deployments

Bem-vindo
Nosso aplicativo de exemplo pode ser encontrado em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/blue-green-app.

?? um aplicativo muito simples com uma implanta????o e um servi??o. Observe que a implanta????o foi convertida em um Rollout e tem uma se????o extra sobre as op????es de implanta????o azul/verde.

Voc?? tamb??m pode ver o c??digo-fonte completo em source-code-canary se quiser entender como o aplicativo est?? renderizando uma p??gina da Web.

Quando estiver pronto para prosseguir, pressione Avan??ar.

# Install the Argo Rollouts controller

Antes de come??armos com a entrega progressiva, precisamos instalar o controlador Argo Rollouts no cluster.

Instalamos o Argo CD para voc?? e voc?? pode fazer login na guia UI.

A interface do usu??rio come??a vazia porque nada ?? implantado em nosso cluster. Clique no bot??o "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes:

application name : argo-rollouts-controller
project: default
SYNC POLICY: automatic
AUTO-CREATE Namespace: enabled
repository URL: https://github.com/codefresh-contrib/gitops-certification-examples
path: ./argo-rollouts-controller
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: argo-rollouts

Deixe todos os outros valores vazios ou com sele????es padr??o. Por fim, clique no bot??o Criar. O controlador ser?? instalado no cluster.

Observe que n??o estamos usando o namespace padr??o, mas um novo. ?? imperativo que voc?? selecione a op????o "auto-criar namespace".

Se voc?? n??o selecionar essa op????o, o ArgoCD n??o encontrar?? o namespace e a implanta????o falhar??. Exclua o aplicativo e crie-o novamente se tiver um problema de implanta????o.

Voc?? tamb??m pode fazer manualmente se esqueceu na interface do usu??rio:

```
kubectl create namespace argo-rollouts
```

```
root@kubernetes-vm:~/workdir# kubectl create namespace argo-rollouts
namespace/argo-rollouts created
```
Tente sincronizar novamente o aplicativo.

Voc?? pode ver que o GitOps n??o ?? ??til apenas para seus pr??prios aplicativos, mas tamb??m para outros aplicativos de suporte.

Aguarde algum tempo at?? que o controlador esteja totalmente sincronizado e a implanta????o seja marcada como Saud??vel.

Quando estiver pronto para prosseguir, pressione Verificar.

# The first deployment

O controlador Argo Rollouts est?? sendo executado agora no cluster e est?? pronto para lidar com implanta????es.

Vamos implantar nosso aplicativo. Como esta ?? a primeira implanta????o, n??o h?? vers??o anterior e, portanto, ocorrer?? uma implanta????o normal.

Instalamos o Argo CD para voc?? e voc?? pode fazer login na guia UI.

Clique no bot??o "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes


application name : demo
project: default
SYNC POLICY: Manual
repository URL: https://github.com/codefresh-contrib/gitops-certification-examples
path: ./blue-green-app
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: default


Deixe todos os outros valores vazios ou com sele????es padr??o.

Por fim, clique no bot??o Criar. A entrada do aplicativo aparecer?? no painel principal. Clique nisso.

O aplicativo ser?? inicialmente "Fora de Sincroniza????o". Pressione o bot??o de sincroniza????o para sincroniz??-lo e aguarde at?? que esteja ??ntegro.

O aplicativo ?? implantado e voc?? pode v??-lo na guia "Tr??fego ao vivo". Voc?? deve ver o seguinte.

O Argo Rollouts tamb??m possui uma CLI opcional que pode ser usada para monitorar e promover implanta????es

J?? instalamos os lan??amentos do kubectl argo para voc?? neste exerc??cio. E podemos us??-lo para monitorar a primeira implanta????o

Execute o seguinte

```
kubectl argo rollouts list rollouts
kubectl argo rollouts status simple-rollout
kubectl argo rollouts get rollout simple-rollout
```

```
root@kubernetes-vm:~/workdir# kubectl argo rollouts list rollouts
NAME            STRATEGY   STATUS        STEP  SET-WEIGHT  READY  DESIRED  UP-TO-DATE  AVAILABLE
simple-rollout  BlueGreen  Healthy       -     -           2/2    2        2           2 

root@kubernetes-vm:~/workdir# kubectl argo rollouts status simple-rollout
Healthy

root@kubernetes-vm:~/workdir# kubectl argo rollouts get rollout simple-rollout
Name:            simple-rollout
Namespace:       default
Status:          ??? Healthy
Strategy:        BlueGreen
Images:          docker.io/kostiscodefresh/gitops-canary-app:v1.0 (stable, active)
Replicas:
  Desired:       2
  Current:       2
  Updated:       2
  Ready:         2
  Available:     2

NAME                                       KIND        STATUS     AGE  INFO
??? simple-rollout                           Rollout     ??? Healthy  56s  
?????????# revision:1                                                        
   ???????????? simple-rollout-b68b5bffb           ReplicaSet  ??? Healthy  56s  stable,active
      ???????????? simple-rollout-b68b5bffb-2zpbs  Pod         ??? Running  56s  ready:1/1
      ???????????? simple-rollout-b68b5bffb-4rx6l  Pod         ??? Running  56s  ready:1/1
```

O ??ltimo comando mostra o status da distribui????o. Como esta ?? a primeira vers??o, h?? apenas um conjunto de r??plicas com dois pods.

Voc?? tamb??m pode ver isso com

```
kubectl get rs
```

```
root@kubernetes-vm:~/workdir# kubectl get rs
NAME                       DESIRED   CURRENT   READY   AGE
simple-rollout-b68b5bffb   2         2         2       86s
```

Quando estiver pronto para prosseguir, pressione Verificar.

# Blue/Green deployments

Agora estamos prontos para ter uma implanta????o azul/verde com a pr??xima vers??o.

Altere a imagem do cont??iner do lan??amento para a pr??xima vers??o com:

```
kubectl argo rollouts set image simple-rollout webserver-simple=docker.io/kostiscodefresh/gitops-canary-app:v2.0
```

```
root@kubernetes-vm:~/workdir# kubectl argo rollouts set image simple-rollout webserver-simple=docker.io/kostiscodefresh/gitops-canary-app:v2.0
rollout "simple-rollout" image updated
```

Estamos usando o kubectl apenas para fins de ilustra????o. Normalmente voc?? deve seguir os princ??pios do GitOps e realizar um commit no reposit??rio Git do aplicativo. Mas apenas para este exerc??cio faremos todas as a????es manualmente para que voc?? tenha tempo de ver o que acontece.

Digite o seguinte para ver o que o Argo Rollouts est?? fazendo nos bastidores

```
kubectl argo rollouts get rollout simple-rollout
```

```
root@kubernetes-vm:~/workdir# kubectl argo rollouts get rollout simple-rollout
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         BlueGreenPause
Strategy:        BlueGreen
Images:          docker.io/kostiscodefresh/gitops-canary-app:v1.0 (stable, active)
                 docker.io/kostiscodefresh/gitops-canary-app:v2.0 (preview)
Replicas:
  Desired:       2
  Current:       4
  Updated:       2
  Ready:         2
  Available:     2

NAME                                        KIND        STATUS     AGE    INFO
??? simple-rollout                            Rollout     ??? Paused   3m28s  
?????????# revision:2                                                           
???  ???????????? simple-rollout-597df99d85           ReplicaSet  ??? Healthy  51s    preview
???     ???????????? simple-rollout-597df99d85-87m6n  Pod         ??? Running  51s    ready:1/1
???     ???????????? simple-rollout-597df99d85-qjfjl  Pod         ??? Running  51s    ready:1/1
?????????# revision:1                                                           
   ???????????? simple-rollout-b68b5bffb            ReplicaSet  ??? Healthy  3m28s  stable,active
      ???????????? simple-rollout-b68b5bffb-2zpbs   Pod         ??? Running  3m28s  ready:1/1
      ???????????? simple-rollout-b68b5bffb-4rx6l   Pod         ??? Running  3m28s  ready:1/1
```

Em seguida, monitore novamente o lan??amento com

```
kubectl argo rollouts get rollout simple-rollout --watch
```

```
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         BlueGreenPause
Strategy:        BlueGreen
Images:          docker.io/kostiscodefresh/gitops-canary-app:v1.0 (stable, active)
                 docker.io/kostiscodefresh/gitops-canary-app:v2.0 (preview)
Replicas:
  Desired:       2
  Current:       4
  Updated:       2
  Ready:         2
  Available:     2

NAME                                        KIND        STATUS     AGE    INFO
??? simple-rollout                            Rollout     ??? Paused   5m8s   
?????????# revision:2                                                           
???  ???????????? simple-rollout-597df99d85           ReplicaSet  ??? Healthy  2m31s  preview
???     ???????????? simple-rollout-597df99d85-87m6n  Pod         ??? Running  2m31s  ready:1/1
???     ???????????? simple-rollout-597df99d85-qjfjl  Pod         ??? Running  2m31s  ready:1/1
?????????# revision:1                                                           
   ???????????? simple-rollout-b68b5bffb            ReplicaSet  ??? Healthy  5m8s   stable,active
      ???????????? simple-rollout-b68b5bffb-2zpbs   Pod         ??? Running  5m8s   ready:1/1
      ???????????? simple-rollout-b68b5bffb-4rx6l   Pod         ??? Running  5m8s   ready:1/1
```

Depois de um tempo, voc?? ver?? os pods da vers??o antiga sendo destru??dos.

Agora todo o tr??fego ao vivo vai para a nova vers??o, como pode ser visto na guia "tr??fego ao vivo".

A implanta????o foi conclu??da com sucesso agora.

Terminar
Quando estiver pronto para terminar a faixa, pressione Check.

# 10 Canary deployments

Entrega progressiva com Argo Rollouts
O aplicativo de exemplo est?? em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/canary-app.

Objetivos
Nesta faixa, isso ?? o que voc?? aprender??:

Instala????o do controlador Argo Rollouts
Como instalar um gateway de API
Como converter uma implanta????o em um lan??amento
Como executar implanta????es can??rio
Como monitorar o estado da implanta????o
Aguarde enquanto inicializamos a VM para voc?? e iniciamos o Kubernetes.

Bem-vindo
Nosso aplicativo de exemplo pode ser encontrado em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/canary-app.

?? um aplicativo muito simples com uma implanta????o e um servi??o. Observe que a implanta????o foi convertida em um Rollout e tem uma se????o extra sobre as op????es de implanta????o canary.

Voc?? tamb??m pode ver o c??digo-fonte completo em source-code-canary se quiser entender como o aplicativo est?? renderizando uma p??gina da Web.

Quando estiver pronto para prosseguir, pressione Avan??ar.

# Install the Argo Rollouts controller

Antes de come??armos com a entrega progressiva, precisamos instalar o controlador Argo Rollouts no cluster.

Instalamos o Argo CD para voc?? e voc?? pode fazer login na guia UI.

A interface do usu??rio come??a vazia porque nada ?? implantado em nosso cluster. Clique no bot??o "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes:

application name : argo-rollouts-controller
project: default
SYNC POLICY: automatic
AUTO-CREATE Namespace: enabled
repository URL: https://github.com/codefresh-contrib/gitops-certification-examples
path: ./argo-rollouts-controller
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: argo-rollouts

Deixe todos os outros valores vazios ou com sele????es padr??o. Por fim, clique no bot??o Criar. O controlador ser?? instalado no cluster.

Observe que n??o estamos usando o namespace padr??o, mas um novo. ?? imperativo que voc?? selecione a op????o "auto-criar namespace".


Se voc?? n??o selecionar essa op????o, o ArgoCD n??o encontrar?? o namespace e a implanta????o falhar??. Exclua o aplicativo e crie-o novamente se tiver um problema de implanta????o.

Voc?? tamb??m pode fazer manualmente se esqueceu na interface do usu??rio:

```
kubectl create namespace argo-rollouts
```

```
root@kubernetes-vm:~/workdir# kubectl create namespace argo-rollouts
namespace/argo-rollouts created
```

Tente sincronizar novamente o aplicativo.

Voc?? pode ver que o GitOps n??o ?? ??til apenas para seus pr??prios aplicativos, mas tamb??m para outros aplicativos de suporte.

Aguarde algum tempo at?? que o controlador esteja totalmente sincronizado e a implanta????o seja marcada como Saud??vel.

Quando estiver pronto para prosseguir, pressione Verificar.

# Install Ambassador

As implanta????es Blue/Green podem funcionar em qualquer cluster vanilla do Kubernetes. Mas para implanta????es Canary, voc?? precisa de uma camada de servi??o inteligente que possa deslocar gradualmente o tr??fego para os pods canary, mantendo o restante do tr??fego para os pods antigos/est??veis.

Argo Rollouts suporta v??rias malhas de servi??o e gateways para esta finalidade.

Nesta li????o, usaremos o popular Ambassador API Gateway for Kubernetes para dividir o tr??fego ao vivo entre a vers??o can??rio e a vers??o antiga/est??vel.

Instalamos o Argo CD para voc?? e voc?? pode fazer login na guia UI.

Clique no bot??o "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes:

application name : ambassador
project: default
SYNC POLICY: automatic
AUTO-CREATE Namespace: enabled
repository URL: https://github.com/codefresh-contrib/gitops-certification-examples
path: ./ambassador-chart
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: ambassador

Deixe todos os outros valores vazios ou com sele????es padr??o. Por fim, clique no bot??o Criar. O Ambassador Gateway ser?? instalado no cluster.

Observe que n??o estamos usando o namespace padr??o, mas um novo. ?? imperativo que voc?? selecione a op????o "auto-criar namespace".


Se voc?? n??o selecionar essa op????o, o ArgoCD n??o encontrar?? o namespace e a implanta????o falhar??. Exclua o aplicativo e crie-o novamente se tiver um problema de implanta????o.

Voc?? tamb??m pode fazer manualmente se esqueceu na interface do usu??rio:

```
kubectl create namespace ambassador
```

```
root@kubernetes-vm:~/workdir# kubectl create namespace ambassador
namespace/ambassador created
```


Tente sincronizar novamente o aplicativo.

Novamente, voc?? pode ver que o GitOps n??o ?? ??til apenas para seus pr??prios aplicativos, mas tamb??m para outros aplicativos de suporte.

Aguarde algum tempo at?? que o gateway esteja totalmente sincronizado e a implanta????o seja marcada como Saud??vel.

Quando estiver pronto para prosseguir, pressione Verificar.

# The first deployment

O controlador Argo Rollouts est?? sendo executado agora no cluster e est?? pronto para lidar com implanta????es.

Vamos implantar nosso aplicativo. Como esta ?? a primeira implanta????o, n??o h?? vers??o anterior e, portanto, ocorrer?? uma implanta????o normal.

Instalamos o Argo CD para voc?? e voc?? pode fazer login na guia UI.

Clique no bot??o "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes

Click the "New app" button on the top left and fill the following details

application name : demo
project: default
SYNC POLICY: Manual
repository URL: https://github.com/codefresh-contrib/gitops-certification-examples
path: ./canary-app
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: default


Deixe todos os outros valores vazios ou com sele????es padr??o.

Por fim, clique no bot??o Criar. A entrada do aplicativo aparecer?? no painel principal. Clique nisso.

O aplicativo ser?? inicialmente "Fora de Sincroniza????o". Pressione o bot??o de sincroniza????o para sincroniz??-lo e aguarde at?? que esteja ??ntegro.

O aplicativo ?? implantado e voc?? pode v??-lo na guia "Tr??fego ao vivo". Voc?? deve ver o seguinte.

O Argo Rollouts tamb??m possui uma CLI opcional que pode ser usada para monitorar e promover implanta????es

J?? instalamos os lan??amentos do kubectl argo para voc?? neste exerc??cio. E podemos us??-lo para monitorar a primeira implanta????o

Execute o seguinte

```
kubectl argo rollouts list rollouts
kubectl argo rollouts status simple-rollout
kubectl argo rollouts get rollout simple-rollout
```

```
root@kubernetes-vm:~/workdir# kubectl argo rollouts list rollouts
NAME            STRATEGY   STATUS        STEP  SET-WEIGHT  READY  DESIRED  UP-TO-DATE  AVAILABLE
simple-rollout  Canary     Healthy       6/6   100         10/10  10       10          10       
root@kubernetes-vm:~/workdir# kubectl argo rollouts status simple-rollout
Healthy
root@kubernetes-vm:~/workdir# kubectl argo rollouts get rollout simple-rollout
Name:            simple-rollout
Namespace:       default
Status:          ??? Healthy
Strategy:        Canary
  Step:          6/6
  SetWeight:     100
  ActualWeight:  100
Images:          docker.io/kostiscodefresh/gitops-canary-app:v1.0 (stable)
Replicas:
  Desired:       10
  Current:       10
  Updated:       10
  Ready:         10
  Available:     10

NAME                                        KIND        STATUS     AGE    INFO
??? simple-rollout                            Rollout     ??? Healthy  2m37s  
?????????# revision:1                                                           
   ???????????? simple-rollout-67df66795d           ReplicaSet  ??? Healthy  2m10s  stable
      ???????????? simple-rollout-67df66795d-4dlmd  Pod         ??? Running  2m10s  ready:1/1
      ???????????? simple-rollout-67df66795d-jpbr6  Pod         ??? Running  2m10s  ready:1/1
      ???????????? simple-rollout-67df66795d-w7274  Pod         ??? Running  2m10s  ready:1/1
      ???????????? simple-rollout-67df66795d-27sv5  Pod         ??? Running  2m9s   ready:1/1
      ???????????? simple-rollout-67df66795d-hrznt  Pod         ??? Running  2m9s   ready:1/1
      ???????????? simple-rollout-67df66795d-8fsvs  Pod         ??? Running  2m8s   ready:1/1
      ???????????? simple-rollout-67df66795d-fxdn7  Pod         ??? Running  2m8s   ready:1/1
      ???????????? simple-rollout-67df66795d-lxqq6  Pod         ??? Running  2m8s   ready:1/1
      ???????????? simple-rollout-67df66795d-mbtcd  Pod         ??? Running  2m8s   ready:1/1
      ???????????? simple-rollout-67df66795d-n6w9k  Pod         ??? Running  2m8s   ready:1/1
```

O ??ltimo comando mostra o status da distribui????o. Como esta ?? a primeira vers??o, h?? apenas um conjunto de r??plicas com dez pods.

Voc?? tamb??m pode ver isso com

```
root@kubernetes-vm:~/workdir# kubectl get rs
NAME                        DESIRED   CURRENT   READY   AGE
simple-rollout-67df66795d   10        10        10      3m33s

root@kubernetes-vm:~/workdir# kubectl get rs
NAME                        DESIRED   CURRENT   READY   AGE
simple-rollout-67df66795d   10        10        10      5m16s
```

Quando estiver pronto para prosseguir, pressione Verificar.

# Canary deployments

Agora estamos prontos para ter uma implanta????o Canary com a pr??xima vers??o.

Altere a imagem do cont??iner do lan??amento para a pr??xima vers??o com:

root@kubernetes-vm:~/workdir# kubectl argo rollouts set image simple-rollout webserver-simple=docker.io/kostiscodefresh/gitops-canary-app:v2.0
rollout "simple-rollout" image updated

Estamos usando o kubectl apenas para fins de ilustra????o. Normalmente voc?? deve seguir os princ??pios do GitOps e realizar um commit no reposit??rio Git do aplicativo. Mas apenas para este exerc??cio faremos todas as a????es manualmente para que voc?? tenha tempo de ver o que acontece.

Digite o seguinte para ver o que o Argo Rollouts est?? fazendo nos bastidores

```
root@kubernetes-vm:~/workdir# kubectl argo rollouts get rollout simple-rollout
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
  Step:          1/6
  SetWeight:     30
  ActualWeight:  30
Images:          docker.io/kostiscodefresh/gitops-canary-app:v1.0 (stable)
                 docker.io/kostiscodefresh/gitops-canary-app:v2.0 (canary)
Replicas:
  Desired:       10
  Current:       13
  Updated:       3
  Ready:         13
  Available:     13

NAME                                        KIND        STATUS     AGE    INFO
??? simple-rollout                            Rollout     ??? Paused   8m12s  
?????????# revision:2                                                           
???  ???????????? simple-rollout-7f9d9f9f8c           ReplicaSet  ??? Healthy  34s    canary
???     ???????????? simple-rollout-7f9d9f9f8c-8qkvt  Pod         ??? Running  34s    ready:1/1
???     ???????????? simple-rollout-7f9d9f9f8c-p64ff  Pod         ??? Running  34s    ready:1/1
???     ???????????? simple-rollout-7f9d9f9f8c-rpjd7  Pod         ??? Running  34s    ready:1/1
?????????# revision:1                                                           
   ???????????? simple-rollout-67df66795d           ReplicaSet  ??? Healthy  7m45s  stable
      ???????????? simple-rollout-67df66795d-4dlmd  Pod         ??? Running  7m45s  ready:1/1
      ???????????? simple-rollout-67df66795d-jpbr6  Pod         ??? Running  7m45s  ready:1/1
      ???????????? simple-rollout-67df66795d-w7274  Pod         ??? Running  7m45s  ready:1/1
      ???????????? simple-rollout-67df66795d-27sv5  Pod         ??? Running  7m44s  ready:1/1
      ???????????? simple-rollout-67df66795d-hrznt  Pod         ??? Running  7m44s  ready:1/1
      ???????????? simple-rollout-67df66795d-8fsvs  Pod         ??? Running  7m43s  ready:1/1
      ???????????? simple-rollout-67df66795d-fxdn7  Pod         ??? Running  7m43s  ready:1/1
      ???????????? simple-rollout-67df66795d-lxqq6  Pod         ??? Running  7m43s  ready:1/1
      ???????????? simple-rollout-67df66795d-mbtcd  Pod         ??? Running  7m43s  ready:1/1
      ???????????? simple-rollout-67df66795d-n6w9k  Pod         ??? Running  7m43s  ready:1/1
```

After you change the image the following things happen

Argo Rollouts creates another replicaset with the new version
The old version is still there and gets live/active traffic
The canary version gets 30% of the live traffic.
ArgoCD will mark the application as out-of-sync
ArgoCD will also mark the health of the application as "suspended" because we have setup the new color to wait
Notice that even though the next version of our application is already deployed, live traffic goes to both new/old versions. You can verify this by looking at the "live traffic" tab.

Neste ponto, a implanta????o ?? suspensa porque usamos as propriedades de pausa na defini????o da distribui????o.

Para promover manualmente a implanta????o e mudar 60% para a nova vers??o, digite:

```
root@kubernetes-vm:~/workdir# kubectl argo rollouts promote simple-rollout
rollout 'simple-rollout' promoted
```

Em seguida, monitore novamente o lan??amento com

```
root@kubernetes-vm:~/workdir# kubectl argo rollouts get rollout simple-rollout
Name:            simple-rollout
Namespace:       default
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
  Step:          3/6
  SetWeight:     60
  ActualWeight:  60
Images:          docker.io/kostiscodefresh/gitops-canary-app:v1.0 (stable)
                 docker.io/kostiscodefresh/gitops-canary-app:v2.0 (canary)
Replicas:
  Desired:       10
  Current:       16
  Updated:       6
  Ready:         16
  Available:     16

NAME                                        KIND        STATUS     AGE   INFO
??? simple-rollout                            Rollout     ??? Paused   12m   
?????????# revision:2                                                          
???  ???????????? simple-rollout-7f9d9f9f8c           ReplicaSet  ??? Healthy  5m5s  canary
???     ???????????? simple-rollout-7f9d9f9f8c-8qkvt  Pod         ??? Running  5m5s  ready:1/1
???     ???????????? simple-rollout-7f9d9f9f8c-p64ff  Pod         ??? Running  5m5s  ready:1/1
???     ???????????? simple-rollout-7f9d9f9f8c-rpjd7  Pod         ??? Running  5m5s  ready:1/1
???     ???????????? simple-rollout-7f9d9f9f8c-4z9bn  Pod         ??? Running  63s   ready:1/1
???     ???????????? simple-rollout-7f9d9f9f8c-n77pv  Pod         ??? Running  63s   ready:1/1
???     ???????????? simple-rollout-7f9d9f9f8c-vw57w  Pod         ??? Running  63s   ready:1/1
?????????# revision:1                                                          
   ???????????? simple-rollout-67df66795d           ReplicaSet  ??? Healthy  12m   stable
      ???????????? simple-rollout-67df66795d-4dlmd  Pod         ??? Running  12m   ready:1/1

root@kubernetes-vm:~/workdir# kubectl argo rollouts get rollout simple-rollout
Name:            simple-rollout
Namespace:       default
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ??? Paused
Message:         CanaryPauseStep
Strategy:        Canary
  Step:          3/6
  SetWeight:     60
  ActualWeight:  60
Images:          docker.io/kostiscodefresh/gitops-canary-app:v1.0 (stable)
                 docker.io/kostiscodefresh/gitops-canary-app:v2.0 (canary)
Replicas:
  Desired:       10
  Current:       16
  Updated:       6
  Ready:         16
  Available:     16

NAME                                        KIND        STATUS     AGE    INFO
??? simple-rollout                            Rollout     ??? Paused   13m    
?????????# revision:2                                                           
???  ???????????? simple-rollout-7f9d9f9f8c           ReplicaSet  ??? Healthy  5m34s  canary
???     ???????????? simple-rollout-7f9d9f9f8c-8qkvt  Pod         ??? Running  5m34s  ready:1/1
???     ???????????? simple-rollout-7f9d9f9f8c-p64ff  Pod         ??? Running  5m34s  ready:1/1
???     ???????????? simple-rollout-7f9d9f9f8c-rpjd7  Pod         ??? Running  5m34s  ready:1/1

```

Repita o mesmo processo mais tr??s vezes para enviar 100% para a vers??o can??rio. Observe que em cada etapa voc?? tamb??m pode ver as guias "est??vel" e "inst??vel" e pode ver que pode manter as vers??es antigas e novas em jogo. Isso ?? ??til se, por algum motivo, voc?? tiver outros aplicativos no cluster que sempre precisam ser apontados para a vers??o antiga ou nova enquanto um can??rio est?? em andamento.

Depois de um tempo, voc?? ver?? os pods da vers??o antiga sendo destru??dos.

Agora todo o tr??fego ao vivo vai para a nova vers??o, como pode ser visto na guia "tr??fego ao vivo".

A implanta????o foi conclu??da com sucesso agora.

Terminar
Quando estiver pronto para terminar a faixa, pressione Check.