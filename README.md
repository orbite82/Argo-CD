# Argo-CD
https://learning.codefresh.io/


Deploy Argo CD to Kubernetes
Use the terminal to deploy Argo CD:

'''
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/codefresh-contrib/gitops-certification-examples/main/argocd-noauth/install.yaml
'''
