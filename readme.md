# Authenticating a Kubernetes app with Azure AD
This sample application is based on [https://github.com/kubernetes/ingress-nginx/tree/master/docs/examples/auth/oauth-external-auth](https://github.com/kubernetes/ingress-nginx/tree/master/docs/examples/auth/oauth-external-auth)

## Deploying to Kubernetes cluster
Note: these instructions are specific to an Azure AKS cluster. I assume you have Helm installed and initialized on your cluster.

```
helm install stable/nginx-ingress \
    --name nginx-ingress \
    --namespace kube-system \
    --set controller.service.loadBalancerIP=104.43.217.79 \
    --set rbac.create=false \
    --set rbac.createRole=false \
    --set rbac.createClusterRole=false

helm install stable/cert-manager \
    --name cert-manager \
    --namespace kube-system \
    --set ingressShim.extraArgs='{--default-issuer-name=letsencrypt-prod,--default-issuer-kind=Issuer}' \
    --set rbac.create=false

kubectl apply -f cert-issuer.yaml

kubectl apply -f ingress.yaml
kubectl apply -f oauth2-proxy-ingress.yaml

kubectl apply -f oauth2-proxy.deployment.yaml
kubectl apply -f oauth2-proxy.service.yaml

kubectl apply -f web.deployment.yaml
kubectl apply -f web.service.yaml
```