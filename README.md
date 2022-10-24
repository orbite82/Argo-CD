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

# Argo CD CLI

Passo 1
Além da interface do usuário, o ArgoCD também possui uma CLI. Já instalamos o cli para você e autenticamos na instância.

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

Vamos excluir o aplicativo e implantá-lo novamente, mas desta vez a partir da CLI.

Primeiro exclua o aplicativo

```
argocd app delete demo
```

Confirme a exclusão respondendo sim no terminal. O aplicativo desaparecerá do painel do Argo CD após alguns minutos.

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

O aplicativo aparecerá no painel do ArgoCD.

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

Você precisa ter uma conta do GitHub para este exercício.

O aplicativo de exemplo está em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/simple-app. Fork este repositório em sua própria conta

Objetivos
Nesta faixa, isso é o que você aprenderá:

Como alterar o aplicativo no Git e reimplantar
Como o Argo CD detecta alterações entre o cluster e o Git
Como fazer alterações no cluster e ver o diff
Aguarde enquanto inicializamos a VM para você e iniciamos o Kubernetes.

Bem-vindo
Nosso aplicativo de exemplo pode ser encontrado em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/simple-app

Certifique-se de bifurcá-lo para sua própria conta e não para o URL. Deve ser algo como:

https://github.com/<seu usuário>/gitops-certification-examples/

Dê uma olhada nos manifestos do Kubernetes para entender o que vamos implantar. É um aplicativo muito simples com uma implantação e um serviço

Quando estiver pronto para prosseguir, pressione Avançar.

https://github.com/orbite82/gitops-certification-examples

Primeira implantação
Instalamos o Argo CD para você e você pode fazer login na guia UI.

A interface do usuário começa vazia porque nada é implantado em nosso cluster. Clique no botão "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes:

nome do aplicativo: demonstração
projeto: padrão
URL do repositório: https://github.com/<seu usuário>/gitops-certification-examples
caminho: ./simple-app
Cluster: https://kubernetes.default.svc (este é o mesmo cluster onde o ArgoCD está instalado)
Espaço de nomes: padrão
Deixe todos os outros valores vazios ou com seleções padrão. Por fim, clique no botão Criar. A entrada do aplicativo aparecerá no painel principal. Clique nisso.

Você deve ver o seguinte.

Agora sincronize o aplicativo para garantir que ele seja implantado. Você deve ver o seguinte:

You can also verify the application from the command line with:

```
root@kubernetes-vm:~/workdir# kubectl get deployments
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
simple-deployment   1/1     1            1           26s
```

# Deploy a new version

Queremos implantar outra versão do nosso aplicativo. Vamos alterar o Git e ver como o Argo CD detecta a alteração.

Execute um commit em seu arquivo simple-app/deployment.yml em sua própria conta (você pode usar o GitHub em outra guia para isso) e altere a tag do contêiner na linha 18 de v1.0 para v2.0

Normalmente, o Argo CD verifica o estado entre o Git e o cluster a cada 3 minutos por conta própria. Apenas para acelerar as coisas, você deve clicar manualmente no aplicativo no painel do Argo CD e pressionar o botão "Atualizar"

Você deve ver o seguinte.


Aqui o Argo CD nos diz que o estado do Git não é mais o mesmo que o estado do cluster. Nas setas amarelas, você pode ver que de todos os componentes do Kubernetes a implantação agora é diferente. E assim toda a aplicação é diferente.

Você pode clicar no botão "App diff" e ver a mudança exata. Ative a caixa de seleção "compact diff".

# Detecting cluster changes

Passo 1
Vamos trazer o cluster de volta ao mesmo estado do Git. Clique no botão Sincronizar na interface do usuário para sincronizar o aplicativo com a nova versão.

Você também pode executar o seguinte na CLI:

´´´
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

´´´
Detectar mudanças no Git e aplicar é um cenário bem conhecido. A grande força do GitOps é que o Argo CD também funciona na direção oposta. Se você fizer alguma alteração no cluster, o Argo CD a detectará e novamente informará que algo está diferente entre o Git e seu cluster.

Digamos que alguém altere manualmente as réplicas da implantação sem criar um Pull Request oficial (uma prática ruim em geral).

Execute o seguinte

```
root@kubernetes-vm:~/workdir# kubectl scale --replicas=3 deployment simple-deployment
deployment.apps/simple-deployment scaled
```

Normalmente, o Argo CD verifica o estado entre o Git e o cluster a cada 3 minutos por conta própria. Apenas para acelerar as coisas, você deve clicar manualmente no aplicativo no painel do Argo CD e pressionar o botão "Atualizar". Clique no botão "App diff" e ative a caixa de seleção "Compact Diff".

Você deve ver o seguinte:

O Argo CD detecta novamente a mudança entre os dois estados. Esse recurso é muito poderoso e você pode detectar facilmente o desvio de configuração entre seus ambientes.

Terminar
Clique em Verificar para terminar esta faixa.

# 04 Sync strategies

Bem-vindo
Nosso aplicativo de exemplo pode ser encontrado em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/sync-strategies

Certifique-se de bifurcá-lo para sua própria conta e anote o URL. Deve ser algo como:

https://github.com/<your user>/gitops-certification-examples/

Dê uma olhada nos manifestos do Kubernetes para entender o que vamos implantar. É um aplicativo muito simples com uma implantação e um serviço

Quando estiver pronto para prosseguir, pressione Avançar.

# Auto-sync

Instalamos o Argo CD para você e você pode fazer login na guia UI.

A interface do usuário começa vazia porque nada é implantado em nosso cluster. Clique no botão "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes:

application name : demo
project: default
SYNC POLICY: automatic
repository URL: https://github.com/<your user>/gitops-certification-examples
path: ./sync-strategies
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: default

Deixe todos os outros valores vazios ou com seleções padrão. Por fim, clique no botão Criar. A entrada do aplicativo aparecerá no painel principal. Clique nisso.

Como selecionamos a estratégia de sincronização automática, o Argo CD implantará automaticamente o aplicativo imediatamente (porque o estado do cluster não é o mesmo que o estado de confirmação)

Você também pode verificar o aplicativo na linha de comando com:

```
kubectl get deployments
```

```
root@kubernetes-vm:~/workdir# kubectl get deployments
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
simple-deployment   1/1     1            1           65s
```

# Deploy a new version with AutoSync

Se você olhar para a guia "Aplicativo implantado", verá que nosso aplicativo está na versão 1.0

Queremos implantar outra versão do nosso aplicativo. Vamos alterar o Git e ver como o Argo CD detecta a mudança e implanta automaticamente (já que definimos a estratégia de sincronização como automática).

Execute uma confirmação em seu arquivo sync-strategies/deployment.yml em sua própria conta (você pode usar o GitHub em outra guia para isso) e altere a tag do contêiner na linha 18 de v1.0 para v2.0

Normalmente, o Argo CD verifica o estado entre o Git e o cluster a cada 3 minutos por conta própria. Apenas para acelerar as coisas, você deve clicar manualmente no aplicativo no painel do Argo CD e pressionar o botão "Atualizar"

Você deve ver o seguinte.

O aplicativo já está implantado. Se você clicar no botão "atualizar" no canto superior direito da guia "Aplicativo implantado", verá que a versão 2 já está ativa.

Quando estiver pronto para prosseguir, pressione Verificar.

# Enabling self-heal

Passo 1
A sincronização automática implantará automaticamente seu aplicativo assim que o repositório Git for alterado. Mas se alguém fizer uma alteração manual no estado do cluster, o Argo CD não fará nada por padrão (ele ainda marcará o aplicativo como fora de sincronia).

Você pode habilitar a opção de autorrecuperação para informar ao Argo CD para descartar quaisquer alterações feitas manualmente no cluster. Essa é uma grande vantagem, pois sempre torna seus ambientes à prova de balas contra alterações ad-hoc via kubectl.

Execute o seguinte

```
kubectl scale --replicas=3 deployment simple-deployment
```

```
root@kubernetes-vm:~/workdir# kubectl scale --replicas=3 deployment simple-deployment
deployment.apps/simple-deployment scaled
```

Você alterou manualmente o cluster. Agora vá para o painel do Argo CD e clique no aplicativo. Na tela do aplicativo, clique no botão superior esquerdo "Detalhes do aplicativo".

Role para baixo e encontre a "seção de política de sincronização". Clique no botão "Self-heal" para habilitá-lo. Responda ok para a caixa de diálogo de confirmação.

A alteração manual será descartada e as réplicas voltarão a ser uma.

Agora você pode tentar alterar qualquer coisa no cluster e todas as alterações sempre serão descartadas (pois não fazem parte do Git).

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
Você alterou manualmente o cluster. Agora vá para o painel do Argo CD e clique no aplicativo. Na tela do aplicativo, clique no botão superior esquerdo "Detalhes do aplicativo".

Role para baixo e encontre a "seção de política de sincronização". Clique no botão "Self-heal" para habilitá-lo. Responda ok para a caixa de diálogo de confirmação.

A alteração manual será descartada e as réplicas voltarão a ser uma.

Agora você pode tentar alterar qualquer coisa no cluster e todas as alterações sempre serão descartadas (pois não fazem parte do Git).

Tente novamente no terminal.

```
root@kubernetes-vm:~/workdir# kubectl scale --replicas=3 deployment simple-deployment
deployment.apps/simple-deployment scaled
root@kubernetes-vm:~/workdir# kubectl get deployment simple-deployment
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
simple-deployment   3/3     3            3           5m2s
```

Seu aplicativo sempre terá 1 pod implantado. Isso é o que o estado do Git diz e, portanto, o Argo CD automaticamente torna o estado do cluster o mesmo.

Terminar
Quando estiver pronto para prosseguir, pressione Verificar.

# Enable auto-prune

Mesmo depois de ativar a sincronização automática e a autorrecuperação, o Argo CD nunca excluirá recursos do cluster se você removê-los no Git.

Para habilitar esse comportamento, você precisa habilitar a opção de remoção automática.

Primeiro, faça um commit em seu repositório Git excluindo o arquivo sync-strategies/deployment.yml.

Em seguida, clique no botão "Atualizar" no painel do ArgoCD. O ArgoCD detectará a alteração e marcará o aplicativo como "Fora de sincronia". Mas a implantação ainda estará lá.

Você deve ver o seguinte.

Você ainda pode ver a implantação com

```
kubectl get deployment simple-deployment
```

```
root@kubernetes-vm:~/workdir# kubectl get deployment simple-deployment
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
simple-deployment   1/1     1            1           10m
```

Vá ao painel do Argo CD e clique no aplicativo. Na tela do aplicativo, clique no botão superior esquerdo "Detalhes do aplicativo".

Role para baixo e encontre a "seção de política de sincronização". Clique no botão "Remover recursos" para habilitá-lo. Responda ok para a caixa de diálogo de confirmação.

Feche a janela de configurações e clique novamente no botão "Atualizar" no aplicativo.

Agora sua implantação será removida (já que ela não existe no Git).

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

Certifique-se de bifurcá-lo para sua própria conta e anote o URL. Deve ser algo como:

https://github.com/<seu usuário>/gitops-certification-examples/

Dê uma olhada nos manifestos do Kubernetes em secret-app/manifests/ para entender o que vamos implantar.

É um aplicativo muito simples com uma implantação e um serviço que também precisa de dois arquivos de segredos (para conexão db e certificado paypal) que são passados ​​para o aplicativo como arquivos em /secrets/.

Os dois segredos são armazenados nos arquivos db-creds-encrypted.yaml e paypal-cert-encrypted.yaml Esses arquivos estão vazios no repositório Git porque queremos criptografá-los primeiro.

Você também pode ver o código-fonte completo em source-code-secrets se quiser entender como o aplicativo está carregando segredos de arquivos.

Quando estiver pronto para prosseguir, pressione Avançar.

# Install the Sealed Secrets controller

Colocamos os segredos brutos/não criptografados em seu diretório de trabalho que são necessários para o aplicativo. Você pode ver os arquivos na guia Editor.

Esses segredos são segredos simples do Kubernetes e não são criptografados de forma alguma. Vamos instalar o controlador de segredos selados da Bitnami para criptografar nossos segredos.

Instalamos o Argo CD para você e você pode fazer login na guia UI.

A interface do usuário começa vazia porque nada é implantado em nosso cluster. Clique no botão "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes:

Deixe todos os outros valores vazios ou com seleções padrão. Por fim, clique no botão Criar. O controlador será instalado no cluster.

Observe que estão usando um namespace específico para o controlador e não o padrão. É imperativo entrar no sistema kube, caso contrário, o controlador de segredos não funcionará.

Você pode ver que o GitOps não é útil apenas para seus próprios aplicativos, mas também para outros aplicativos de suporte.

Quando estiver pronto para prosseguir, pressione Verificar.

# Use Kubeseal to encrypt secretsb

O controlador Sealed Secrets está em execução agora no cluster e está pronto para descriptografar segredos.

Agora precisamos criptografar nossos segredos e enviá-los para o git. A criptografia acontece com o executável kubeseal. Ele precisa ser instalado da mesma maneira que o kubectl. Ele reutiliza a autenticação de cluster já usada pelo kubectl.

Já instalamos o kubeseal para você neste exercício. Você pode usá-lo imediatamente para criptografar seus segredos simples do Kubernetes e convertê-los em segredos criptografados

Execute o seguinte

```
kubeseal < unsealed_secrets/db-creds.yml > sealed_secrets/db-creds-encrypted.yaml -o yaml
kubeseal < unsealed_secrets/paypal-cert.yml > sealed_secrets/paypal-cert-encrypted.yaml -o yaml
```

Agora você tem segredos criptografados. Abra os arquivos na guia "Editor" e copie o conteúdo em sua área de transferência.

Em seguida, vá para a interface do usuário do Github em outra guia do navegador e confirme/envie seu conteúdo em seu próprio fork do aplicativo, preenchendo os arquivos vazios em gitops-certification-examples/secret-app/manifests/db-creds-encrypted.yaml e gitops- certificate-examples/secret-app/manifests/paypal-cert-encrypted.yaml

Quando estiver pronto para prosseguir, pressione Verificar.

# Deploy secrets

Passo 1
Agora que todos os nossos segredos estão no Git de forma criptografada, podemos implantar nosso aplicativo normalmente.

Faça login na interface do usuário do Argo CD.

Clique no botão "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes:

application name : demo
project: default
SYNC POLICY: automatic
repository URL: https://github.com/your_github_account/gitops-certification-examples
path: ./secret-app/manifests
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: default

# Bem-vindo

Até agora, em todos os desafios anteriores, você criou aplicativos por meio da interface do usuário do ArgoCD ou CLI. Um aplicativo ArgoCD é essencialmente uma combinação de um repositório git, um projeto Argo, várias opções de sincronização e outros valores.

Esta informação não precisa ser confinada no próprio Argo CD. Ele pode ser modelado em um recurso do Kubernetes para que possa ser armazenado no Git e gerenciado com o GitOps também.

Dê uma olhada em https://github.com/codefresh-contrib/gitops-certification-examples/blob/main/declarative/single-app/my-application.yml para um exemplo de aplicativo

Quando estiver pronto para prosseguir, pressione Avançar.

# ArgoCD declarative format

Como nosso aplicativo ArgoCD agora é um recurso do Kubernetes, podemos tratá-lo como qualquer outro recurso do Kubernetes.

Verificamos para você localmente o arquivo my-application.yml Aplique-o com

```
kubectl apply -f my-application.yml
```

```
root@kubernetes-vm:~/workdir# kubectl apply -f my-application.yml
application.argoproj.io/demo created
```

Também instalamos o Argo CD para você e você o vê na guia UI.

Agora você deve ver o aplicativo já na interface do usuário do ArgoCD.

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

Observe que o valor do namespace é o namespace no qual o aplicativo pai é implantado e não o namespace no qual os aplicativos individuais são implantados. Os aplicativos ArgoCD devem ser implantados no mesmo namespace que o próprio ArgoCD.

Deixe todos os outros valores vazios ou com seleções padrão. Por fim, clique no botão Criar.

O ArgoCD implantará o aplicativo pai e seus 3 filhos. Clique no aplicativo pai. Você deve ver o seguinte.

Passe algum tempo na interface do usuário para entender onde cada aplicativo é implantado. Você também pode usar a linha de comando

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

Dê uma olhada nos gráficos do Helm para entender o que vamos implantar. É um aplicativo muito simples com uma implantação e um serviço. O modelo do Ingress no gráfico está desabilitado por padrão.

Quando estiver pronto para prosseguir, pressione Avançar.

# Using the ArgoCD GUI

Instalamos o Argo CD para você e você pode fazer login na guia UI.

A interface do usuário começa vazia porque nada é implantado em nosso cluster. Clique no botão "NOVO APP" no canto superior esquerdo e preencha os seguintes detalhes:

application name : helm-gitops-example
project: default
sync policy: automatic
repository URL: https://github.com/codefresh-contrib/gitops-certification-examples
path: ./helm-app/
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: default

Deixe todos os outros valores vazios ou com seleções padrão. Observe que, no caso de gráficos do Helm, você também pode substituir valores específicos na parte inferior da caixa de diálogo. Isso é totalmente opcional, pois nosso gráfico Helm já possui o arquivo values.yaml correto.

Por fim, clique no botão Criar. A entrada do aplicativo aparecerá no painel principal. Clique nisso.

Parabéns! Você configurou seu primeiro aplicativo Helm com o GitOps. Você deve ver o seguinte:

Você pode verificar a implantação com

```
kubectl get deployment
```

```
root@kubernetes-vm:~/workdir# kubectl get deployment
NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
helm-gitops-example-helm-example   1/1     1            1           39s
```

Observe que, se você usar qualquer um dos comandos do leme, como a lista do leme, não verá nada. Um gráfico do Helm implantado pelo ArgoCD não é mais registrado como uma implantação do Helm. Isso ocorre porque o ArgoCD não inclui as informações de carga útil do Helm. Ao implantar um aplicativo Helm, o Argo CD executa o "modelo de leme" e implanta os manifestos resultantes.

# Using ArgoCD CLI

Passo 1
Além da interface do usuário, o ArgoCD também possui uma CLI. Já instalamos o cli para você e autenticamos na instância.

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
Vamos excluir o aplicativo e implantá-lo novamente, mas desta vez a partir da CLI.

Primeiro exclua o aplicativo

```
argocd app delete helm-gitops-example
```

```
root@kubernetes-vm:~/workdir# argocd app delete helm-gitops-example
Are you sure you want to delete 'helm-gitops-example' and all its resources? [y/n]
y
```

Confirme a exclusão respondendo sim no terminal. O aplicativo desaparecerá do painel do Argo CD após alguns minutos.

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

O aplicativo aparecerá novamente no painel do ArgoCD.

Confirme a implantação

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
Se você confirmou a implantação, clique em Verificar para concluir esta faixa.


# 08 Using Kustomize with ArgoCD

Bem-vindo
Nosso aplicativo de exemplo pode ser encontrado em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/kustomize-app

Dê uma olhada nas pastas base e overlays para entender o que vamos implantar. É um aplicativo muito simples com 2 ambientes para implantação: preparação e produção.

Os ambientes diferem em vários aspectos como número de réplicas, banco de dados utilizado, rede etc.

Quando estiver pronto para prosseguir, pressione Avançar.

# Using the ArgoCD GUI

Instalamos o Argo CD para você e você pode fazer login na guia UI.

A interface do usuário começa vazia porque nada é implantado em nosso cluster. Clique no botão "NOVO APP" no canto superior esquerdo e preencha os seguintes detalhes:
Abrir no Google T

application name : kustomize-gitops-example
project: default
sync policy: automatic
repository URL: https://github.com/codefresh-contrib/gitops-certification-examples
path: ./kustomize-app/overlays/staging
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: default

Deixe todos os outros valores vazios ou com seleções padrão. Por fim, clique no botão Criar. A entrada do aplicativo aparecerá no painel principal. Clique nisso.

Parabéns! Você configurou seu primeiro aplicativo Kustomize com GitOps.

Você deve ver o seguinte.

Aguarde algum tempo para o aplicativo iniciar e, em seguida, visite o "Aplicativo implantado" para vê-lo em execução. Você verá que ele carregou o valor de teste para o banco de dados.

# Using ArgoCD CLI

Passo 1
Além da interface do usuário, o ArgoCD também possui uma CLI. Já instalamos o cli para você e autenticamos na instância.

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

Vamos excluir o aplicativo e implantá-lo novamente, mas desta vez a partir da CLI.

Primeiro exclua o aplicativo

```
argocd app delete kustomize-gitops-example
```

```
root@kubernetes-vm:~/workdir# argocd app delete kustomize-gitops-example
Are you sure you want to delete 'kustomize-gitops-example' and all its resources? [y/n]
y
```

Confirme a exclusão respondendo sim no terminal. O aplicativo desaparecerá do painel do Argo CD após alguns minutos.

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

O aplicativo aparecerá no painel do ArgoCD.

Confirme a implantação

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
Se você confirmou a implantação, clique em Verificar para concluir esta faixa.

# 09 Blue/Green deployments

Bem-vindo
Nosso aplicativo de exemplo pode ser encontrado em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/blue-green-app.

É um aplicativo muito simples com uma implantação e um serviço. Observe que a implantação foi convertida em um Rollout e tem uma seção extra sobre as opções de implantação azul/verde.

Você também pode ver o código-fonte completo em source-code-canary se quiser entender como o aplicativo está renderizando uma página da Web.

Quando estiver pronto para prosseguir, pressione Avançar.

# Install the Argo Rollouts controller

Antes de começarmos com a entrega progressiva, precisamos instalar o controlador Argo Rollouts no cluster.

Instalamos o Argo CD para você e você pode fazer login na guia UI.

A interface do usuário começa vazia porque nada é implantado em nosso cluster. Clique no botão "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes:

application name : argo-rollouts-controller
project: default
SYNC POLICY: automatic
AUTO-CREATE Namespace: enabled
repository URL: https://github.com/codefresh-contrib/gitops-certification-examples
path: ./argo-rollouts-controller
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: argo-rollouts

Deixe todos os outros valores vazios ou com seleções padrão. Por fim, clique no botão Criar. O controlador será instalado no cluster.

Observe que não estamos usando o namespace padrão, mas um novo. É imperativo que você selecione a opção "auto-criar namespace".

Se você não selecionar essa opção, o ArgoCD não encontrará o namespace e a implantação falhará. Exclua o aplicativo e crie-o novamente se tiver um problema de implantação.

Você também pode fazer manualmente se esqueceu na interface do usuário:

```
kubectl create namespace argo-rollouts
```

```
root@kubernetes-vm:~/workdir# kubectl create namespace argo-rollouts
namespace/argo-rollouts created
```
Tente sincronizar novamente o aplicativo.

Você pode ver que o GitOps não é útil apenas para seus próprios aplicativos, mas também para outros aplicativos de suporte.

Aguarde algum tempo até que o controlador esteja totalmente sincronizado e a implantação seja marcada como Saudável.

Quando estiver pronto para prosseguir, pressione Verificar.

# The first deployment

O controlador Argo Rollouts está sendo executado agora no cluster e está pronto para lidar com implantações.

Vamos implantar nosso aplicativo. Como esta é a primeira implantação, não há versão anterior e, portanto, ocorrerá uma implantação normal.

Instalamos o Argo CD para você e você pode fazer login na guia UI.

Clique no botão "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes


application name : demo
project: default
SYNC POLICY: Manual
repository URL: https://github.com/codefresh-contrib/gitops-certification-examples
path: ./blue-green-app
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: default


Deixe todos os outros valores vazios ou com seleções padrão.

Por fim, clique no botão Criar. A entrada do aplicativo aparecerá no painel principal. Clique nisso.

O aplicativo será inicialmente "Fora de Sincronização". Pressione o botão de sincronização para sincronizá-lo e aguarde até que esteja íntegro.

O aplicativo é implantado e você pode vê-lo na guia "Tráfego ao vivo". Você deve ver o seguinte.

O Argo Rollouts também possui uma CLI opcional que pode ser usada para monitorar e promover implantações

Já instalamos os lançamentos do kubectl argo para você neste exercício. E podemos usá-lo para monitorar a primeira implantação

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
Status:          ✔ Healthy
Strategy:        BlueGreen
Images:          docker.io/kostiscodefresh/gitops-canary-app:v1.0 (stable, active)
Replicas:
  Desired:       2
  Current:       2
  Updated:       2
  Ready:         2
  Available:     2

NAME                                       KIND        STATUS     AGE  INFO
⟳ simple-rollout                           Rollout     ✔ Healthy  56s  
└──# revision:1                                                        
   └──⧉ simple-rollout-b68b5bffb           ReplicaSet  ✔ Healthy  56s  stable,active
      ├──□ simple-rollout-b68b5bffb-2zpbs  Pod         ✔ Running  56s  ready:1/1
      └──□ simple-rollout-b68b5bffb-4rx6l  Pod         ✔ Running  56s  ready:1/1
```

O último comando mostra o status da distribuição. Como esta é a primeira versão, há apenas um conjunto de réplicas com dois pods.

Você também pode ver isso com

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

Agora estamos prontos para ter uma implantação azul/verde com a próxima versão.

Altere a imagem do contêiner do lançamento para a próxima versão com:

```
kubectl argo rollouts set image simple-rollout webserver-simple=docker.io/kostiscodefresh/gitops-canary-app:v2.0
```

```
root@kubernetes-vm:~/workdir# kubectl argo rollouts set image simple-rollout webserver-simple=docker.io/kostiscodefresh/gitops-canary-app:v2.0
rollout "simple-rollout" image updated
```

Estamos usando o kubectl apenas para fins de ilustração. Normalmente você deve seguir os princípios do GitOps e realizar um commit no repositório Git do aplicativo. Mas apenas para este exercício faremos todas as ações manualmente para que você tenha tempo de ver o que acontece.

Digite o seguinte para ver o que o Argo Rollouts está fazendo nos bastidores

```
kubectl argo rollouts get rollout simple-rollout
```

```
root@kubernetes-vm:~/workdir# kubectl argo rollouts get rollout simple-rollout
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
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
⟳ simple-rollout                            Rollout     ॥ Paused   3m28s  
├──# revision:2                                                           
│  └──⧉ simple-rollout-597df99d85           ReplicaSet  ✔ Healthy  51s    preview
│     ├──□ simple-rollout-597df99d85-87m6n  Pod         ✔ Running  51s    ready:1/1
│     └──□ simple-rollout-597df99d85-qjfjl  Pod         ✔ Running  51s    ready:1/1
└──# revision:1                                                           
   └──⧉ simple-rollout-b68b5bffb            ReplicaSet  ✔ Healthy  3m28s  stable,active
      ├──□ simple-rollout-b68b5bffb-2zpbs   Pod         ✔ Running  3m28s  ready:1/1
      └──□ simple-rollout-b68b5bffb-4rx6l   Pod         ✔ Running  3m28s  ready:1/1
```

Em seguida, monitore novamente o lançamento com

```
kubectl argo rollouts get rollout simple-rollout --watch
```

```
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
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
⟳ simple-rollout                            Rollout     ॥ Paused   5m8s   
├──# revision:2                                                           
│  └──⧉ simple-rollout-597df99d85           ReplicaSet  ✔ Healthy  2m31s  preview
│     ├──□ simple-rollout-597df99d85-87m6n  Pod         ✔ Running  2m31s  ready:1/1
│     └──□ simple-rollout-597df99d85-qjfjl  Pod         ✔ Running  2m31s  ready:1/1
└──# revision:1                                                           
   └──⧉ simple-rollout-b68b5bffb            ReplicaSet  ✔ Healthy  5m8s   stable,active
      ├──□ simple-rollout-b68b5bffb-2zpbs   Pod         ✔ Running  5m8s   ready:1/1
      └──□ simple-rollout-b68b5bffb-4rx6l   Pod         ✔ Running  5m8s   ready:1/1
```

Depois de um tempo, você verá os pods da versão antiga sendo destruídos.

Agora todo o tráfego ao vivo vai para a nova versão, como pode ser visto na guia "tráfego ao vivo".

A implantação foi concluída com sucesso agora.

Terminar
Quando estiver pronto para terminar a faixa, pressione Check.

# 10 Canary deployments

Entrega progressiva com Argo Rollouts
O aplicativo de exemplo está em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/canary-app.

Objetivos
Nesta faixa, isso é o que você aprenderá:

Instalação do controlador Argo Rollouts
Como instalar um gateway de API
Como converter uma implantação em um lançamento
Como executar implantações canário
Como monitorar o estado da implantação
Aguarde enquanto inicializamos a VM para você e iniciamos o Kubernetes.

Bem-vindo
Nosso aplicativo de exemplo pode ser encontrado em https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/canary-app.

É um aplicativo muito simples com uma implantação e um serviço. Observe que a implantação foi convertida em um Rollout e tem uma seção extra sobre as opções de implantação canary.

Você também pode ver o código-fonte completo em source-code-canary se quiser entender como o aplicativo está renderizando uma página da Web.

Quando estiver pronto para prosseguir, pressione Avançar.

# Install the Argo Rollouts controller

Antes de começarmos com a entrega progressiva, precisamos instalar o controlador Argo Rollouts no cluster.

Instalamos o Argo CD para você e você pode fazer login na guia UI.

A interface do usuário começa vazia porque nada é implantado em nosso cluster. Clique no botão "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes:

application name : argo-rollouts-controller
project: default
SYNC POLICY: automatic
AUTO-CREATE Namespace: enabled
repository URL: https://github.com/codefresh-contrib/gitops-certification-examples
path: ./argo-rollouts-controller
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: argo-rollouts

Deixe todos os outros valores vazios ou com seleções padrão. Por fim, clique no botão Criar. O controlador será instalado no cluster.

Observe que não estamos usando o namespace padrão, mas um novo. É imperativo que você selecione a opção "auto-criar namespace".


Se você não selecionar essa opção, o ArgoCD não encontrará o namespace e a implantação falhará. Exclua o aplicativo e crie-o novamente se tiver um problema de implantação.

Você também pode fazer manualmente se esqueceu na interface do usuário:

```
kubectl create namespace argo-rollouts
```

```
root@kubernetes-vm:~/workdir# kubectl create namespace argo-rollouts
namespace/argo-rollouts created
```

Tente sincronizar novamente o aplicativo.

Você pode ver que o GitOps não é útil apenas para seus próprios aplicativos, mas também para outros aplicativos de suporte.

Aguarde algum tempo até que o controlador esteja totalmente sincronizado e a implantação seja marcada como Saudável.

Quando estiver pronto para prosseguir, pressione Verificar.

# Install Ambassador

As implantações Blue/Green podem funcionar em qualquer cluster vanilla do Kubernetes. Mas para implantações Canary, você precisa de uma camada de serviço inteligente que possa deslocar gradualmente o tráfego para os pods canary, mantendo o restante do tráfego para os pods antigos/estáveis.

Argo Rollouts suporta várias malhas de serviço e gateways para esta finalidade.

Nesta lição, usaremos o popular Ambassador API Gateway for Kubernetes para dividir o tráfego ao vivo entre a versão canário e a versão antiga/estável.

Instalamos o Argo CD para você e você pode fazer login na guia UI.

Clique no botão "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes:

application name : ambassador
project: default
SYNC POLICY: automatic
AUTO-CREATE Namespace: enabled
repository URL: https://github.com/codefresh-contrib/gitops-certification-examples
path: ./ambassador-chart
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: ambassador

Deixe todos os outros valores vazios ou com seleções padrão. Por fim, clique no botão Criar. O Ambassador Gateway será instalado no cluster.

Observe que não estamos usando o namespace padrão, mas um novo. É imperativo que você selecione a opção "auto-criar namespace".


Se você não selecionar essa opção, o ArgoCD não encontrará o namespace e a implantação falhará. Exclua o aplicativo e crie-o novamente se tiver um problema de implantação.

Você também pode fazer manualmente se esqueceu na interface do usuário:

```
kubectl create namespace ambassador
```

```
root@kubernetes-vm:~/workdir# kubectl create namespace ambassador
namespace/ambassador created
```


Tente sincronizar novamente o aplicativo.

Novamente, você pode ver que o GitOps não é útil apenas para seus próprios aplicativos, mas também para outros aplicativos de suporte.

Aguarde algum tempo até que o gateway esteja totalmente sincronizado e a implantação seja marcada como Saudável.

Quando estiver pronto para prosseguir, pressione Verificar.

# The first deployment

O controlador Argo Rollouts está sendo executado agora no cluster e está pronto para lidar com implantações.

Vamos implantar nosso aplicativo. Como esta é a primeira implantação, não há versão anterior e, portanto, ocorrerá uma implantação normal.

Instalamos o Argo CD para você e você pode fazer login na guia UI.

Clique no botão "Novo aplicativo" no canto superior esquerdo e preencha os seguintes detalhes

Click the "New app" button on the top left and fill the following details

application name : demo
project: default
SYNC POLICY: Manual
repository URL: https://github.com/codefresh-contrib/gitops-certification-examples
path: ./canary-app
Cluster: https://kubernetes.default.svc (this is the same cluster where ArgoCD is installed)
Namespace: default


Deixe todos os outros valores vazios ou com seleções padrão.

Por fim, clique no botão Criar. A entrada do aplicativo aparecerá no painel principal. Clique nisso.

O aplicativo será inicialmente "Fora de Sincronização". Pressione o botão de sincronização para sincronizá-lo e aguarde até que esteja íntegro.

O aplicativo é implantado e você pode vê-lo na guia "Tráfego ao vivo". Você deve ver o seguinte.

O Argo Rollouts também possui uma CLI opcional que pode ser usada para monitorar e promover implantações

Já instalamos os lançamentos do kubectl argo para você neste exercício. E podemos usá-lo para monitorar a primeira implantação

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
Status:          ✔ Healthy
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
⟳ simple-rollout                            Rollout     ✔ Healthy  2m37s  
└──# revision:1                                                           
   └──⧉ simple-rollout-67df66795d           ReplicaSet  ✔ Healthy  2m10s  stable
      ├──□ simple-rollout-67df66795d-4dlmd  Pod         ✔ Running  2m10s  ready:1/1
      ├──□ simple-rollout-67df66795d-jpbr6  Pod         ✔ Running  2m10s  ready:1/1
      ├──□ simple-rollout-67df66795d-w7274  Pod         ✔ Running  2m10s  ready:1/1
      ├──□ simple-rollout-67df66795d-27sv5  Pod         ✔ Running  2m9s   ready:1/1
      ├──□ simple-rollout-67df66795d-hrznt  Pod         ✔ Running  2m9s   ready:1/1
      ├──□ simple-rollout-67df66795d-8fsvs  Pod         ✔ Running  2m8s   ready:1/1
      ├──□ simple-rollout-67df66795d-fxdn7  Pod         ✔ Running  2m8s   ready:1/1
      ├──□ simple-rollout-67df66795d-lxqq6  Pod         ✔ Running  2m8s   ready:1/1
      ├──□ simple-rollout-67df66795d-mbtcd  Pod         ✔ Running  2m8s   ready:1/1
      └──□ simple-rollout-67df66795d-n6w9k  Pod         ✔ Running  2m8s   ready:1/1
```

O último comando mostra o status da distribuição. Como esta é a primeira versão, há apenas um conjunto de réplicas com dez pods.

Você também pode ver isso com

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

Agora estamos prontos para ter uma implantação Canary com a próxima versão.

Altere a imagem do contêiner do lançamento para a próxima versão com:

root@kubernetes-vm:~/workdir# kubectl argo rollouts set image simple-rollout webserver-simple=docker.io/kostiscodefresh/gitops-canary-app:v2.0
rollout "simple-rollout" image updated

Estamos usando o kubectl apenas para fins de ilustração. Normalmente você deve seguir os princípios do GitOps e realizar um commit no repositório Git do aplicativo. Mas apenas para este exercício faremos todas as ações manualmente para que você tenha tempo de ver o que acontece.

Digite o seguinte para ver o que o Argo Rollouts está fazendo nos bastidores

```
root@kubernetes-vm:~/workdir# kubectl argo rollouts get rollout simple-rollout
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
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
⟳ simple-rollout                            Rollout     ॥ Paused   8m12s  
├──# revision:2                                                           
│  └──⧉ simple-rollout-7f9d9f9f8c           ReplicaSet  ✔ Healthy  34s    canary
│     ├──□ simple-rollout-7f9d9f9f8c-8qkvt  Pod         ✔ Running  34s    ready:1/1
│     ├──□ simple-rollout-7f9d9f9f8c-p64ff  Pod         ✔ Running  34s    ready:1/1
│     └──□ simple-rollout-7f9d9f9f8c-rpjd7  Pod         ✔ Running  34s    ready:1/1
└──# revision:1                                                           
   └──⧉ simple-rollout-67df66795d           ReplicaSet  ✔ Healthy  7m45s  stable
      ├──□ simple-rollout-67df66795d-4dlmd  Pod         ✔ Running  7m45s  ready:1/1
      ├──□ simple-rollout-67df66795d-jpbr6  Pod         ✔ Running  7m45s  ready:1/1
      ├──□ simple-rollout-67df66795d-w7274  Pod         ✔ Running  7m45s  ready:1/1
      ├──□ simple-rollout-67df66795d-27sv5  Pod         ✔ Running  7m44s  ready:1/1
      ├──□ simple-rollout-67df66795d-hrznt  Pod         ✔ Running  7m44s  ready:1/1
      ├──□ simple-rollout-67df66795d-8fsvs  Pod         ✔ Running  7m43s  ready:1/1
      ├──□ simple-rollout-67df66795d-fxdn7  Pod         ✔ Running  7m43s  ready:1/1
      ├──□ simple-rollout-67df66795d-lxqq6  Pod         ✔ Running  7m43s  ready:1/1
      ├──□ simple-rollout-67df66795d-mbtcd  Pod         ✔ Running  7m43s  ready:1/1
      └──□ simple-rollout-67df66795d-n6w9k  Pod         ✔ Running  7m43s  ready:1/1
```

After you change the image the following things happen

Argo Rollouts creates another replicaset with the new version
The old version is still there and gets live/active traffic
The canary version gets 30% of the live traffic.
ArgoCD will mark the application as out-of-sync
ArgoCD will also mark the health of the application as "suspended" because we have setup the new color to wait
Notice that even though the next version of our application is already deployed, live traffic goes to both new/old versions. You can verify this by looking at the "live traffic" tab.

Neste ponto, a implantação é suspensa porque usamos as propriedades de pausa na definição da distribuição.

Para promover manualmente a implantação e mudar 60% para a nova versão, digite:

```
root@kubernetes-vm:~/workdir# kubectl argo rollouts promote simple-rollout
rollout 'simple-rollout' promoted
```

Em seguida, monitore novamente o lançamento com

```
root@kubernetes-vm:~/workdir# kubectl argo rollouts get rollout simple-rollout
Name:            simple-rollout
Namespace:       default
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
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
⟳ simple-rollout                            Rollout     ॥ Paused   12m   
├──# revision:2                                                          
│  └──⧉ simple-rollout-7f9d9f9f8c           ReplicaSet  ✔ Healthy  5m5s  canary
│     ├──□ simple-rollout-7f9d9f9f8c-8qkvt  Pod         ✔ Running  5m5s  ready:1/1
│     ├──□ simple-rollout-7f9d9f9f8c-p64ff  Pod         ✔ Running  5m5s  ready:1/1
│     ├──□ simple-rollout-7f9d9f9f8c-rpjd7  Pod         ✔ Running  5m5s  ready:1/1
│     ├──□ simple-rollout-7f9d9f9f8c-4z9bn  Pod         ✔ Running  63s   ready:1/1
│     ├──□ simple-rollout-7f9d9f9f8c-n77pv  Pod         ✔ Running  63s   ready:1/1
│     └──□ simple-rollout-7f9d9f9f8c-vw57w  Pod         ✔ Running  63s   ready:1/1
└──# revision:1                                                          
   └──⧉ simple-rollout-67df66795d           ReplicaSet  ✔ Healthy  12m   stable
      ├──□ simple-rollout-67df66795d-4dlmd  Pod         ✔ Running  12m   ready:1/1

root@kubernetes-vm:~/workdir# kubectl argo rollouts get rollout simple-rollout
Name:            simple-rollout
Namespace:       default
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
Name:            simple-rollout
Namespace:       default
Status:          ॥ Paused
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
⟳ simple-rollout                            Rollout     ॥ Paused   13m    
├──# revision:2                                                           
│  └──⧉ simple-rollout-7f9d9f9f8c           ReplicaSet  ✔ Healthy  5m34s  canary
│     ├──□ simple-rollout-7f9d9f9f8c-8qkvt  Pod         ✔ Running  5m34s  ready:1/1
│     ├──□ simple-rollout-7f9d9f9f8c-p64ff  Pod         ✔ Running  5m34s  ready:1/1
│     ├──□ simple-rollout-7f9d9f9f8c-rpjd7  Pod         ✔ Running  5m34s  ready:1/1

```

Repita o mesmo processo mais três vezes para enviar 100% para a versão canário. Observe que em cada etapa você também pode ver as guias "estável" e "instável" e pode ver que pode manter as versões antigas e novas em jogo. Isso é útil se, por algum motivo, você tiver outros aplicativos no cluster que sempre precisam ser apontados para a versão antiga ou nova enquanto um canário está em andamento.

Depois de um tempo, você verá os pods da versão antiga sendo destruídos.

Agora todo o tráfego ao vivo vai para a nova versão, como pode ser visto na guia "tráfego ao vivo".

A implantação foi concluída com sucesso agora.

Terminar
Quando estiver pronto para terminar a faixa, pressione Check.