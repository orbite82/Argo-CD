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

# Teste