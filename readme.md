# Authenticating a Kubernetes app with Azure AD

## Deploying to Kubernetes cluster
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