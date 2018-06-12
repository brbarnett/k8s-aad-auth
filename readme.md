# Authenticating a Kubernetes app with Azure AD
This sample application is based on [https://github.com/kubernetes/ingress-nginx/tree/master/docs/examples/auth/oauth-external-auth](https://github.com/kubernetes/ingress-nginx/tree/master/docs/examples/auth/oauth-external-auth)

See [https://github.com/brbarnett/much-todo-about-containers](https://github.com/brbarnett/much-todo-about-containers) for more details on how to set up the cluster.

## Architecture
This project is set up with three primary ingress resources:

- Unsecured -- this is for any front-end, static content that you wouldn't typically secure. For example, this is where I would serve a React or Angular single-page application.
- Secured -- these rules are for more sensitive endpoints, such as your API. I am assuming that the SPA delivered by unsecured rules will handle its own authentication against an OAuth 2.0 application, in this case using Azure AD.
- oauth2_proxy -- this ingress route `/oauth2` traffic to the `oauth2_proxy` service (based on [bitly/oauth2_proxy](https://github.com/bitly/oauth2_proxy)) for Azure AD authentication.

Here is a high-level architectural diagram of how this application works:

![High-level architecture](/images/k8s-aad-architecture.jpg "High-level architecture")

## Deploying to Kubernetes cluster
Note: these instructions are specific to an Azure AKS cluster. I assume you have Helm installed and initialized on your cluster. The `104.43.217.79` IP address is a pre-configured static IP address resource in Azure.

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

kubectl apply -f secured-ingress.yaml
kubectl apply -f unsecured-ingress.yaml
kubectl apply -f oauth2-proxy-ingress.yaml

kubectl apply -f oauth2-proxy.deployment.yaml
kubectl apply -f oauth2-proxy.service.yaml

kubectl apply -f web.deployment.yaml
kubectl apply -f web.service.yaml

kubectl apply -f api.deployment.yaml
kubectl apply -f api.service.yaml
```

## Demo
- To demo the unsecured web content, navigate here: [https://todo-aks-cluster.centralus.cloudapp.azure.com](https://todo-aks-cluster.centralus.cloudapp.azure.com)
- To demo the API, send a GET request with bearer token to https://todo-aks-cluster.centralus.cloudapp.azure.com/api