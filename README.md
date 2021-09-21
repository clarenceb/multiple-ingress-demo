Multiple Ingress Controllers
============================

```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

Application 1
-------------

Deploy the app:

```sh
kubectl create ns app1
kubectl apply -f app1.yaml -n app1
```

Install ingress controller for app1:

```sh
kubectl create namespace app1-ingress

helm install app1-ingress ingress-nginx/ingress-nginx \
    --namespace app1-ingress \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."kubernetes\.io/os"=linux \
    --set controller.admissionWebhooks.patch.nodeSelector."kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."kubernetes\.io/os"=linux \
    --set controller.ingressClass=app1 \
    --set controller.ingressClassResource.name=app1 \
    --set controller.ingressClassResource.controllerValue="k8s.io/ingress-nginx"

kubectl wait --namespace app1-ingress \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s

# Name to associate with public IP address
DNSNAME="app1-<uniquename>"

# Get the ingress public IP address
PUBLIC_IP="xx.xx.xx.xx"

# Get the resource-id of the public ip
PUBLICIPID=$(az network public-ip list --query "[?ipAddress!=null]|[?contains(ipAddress, '$PUBLIC_IP')].[id]" --output tsv)

# Update public ip address with dns name
az network public-ip update --ids $PUBLICIPID --dns-name $DNSNAME

kubectl apply -f app1-ingress.yaml -n app1
```

Application 2
-------------

Deploy the app:

```sh
kubectl create ns app2
kubectl apply -f app2.yaml -n app2
```

Install ingress controller for app2:

```sh
kubectl create namespace app2-ingress

helm install app2-ingress ingress-nginx/ingress-nginx \
    --namespace app2-ingress \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."kubernetes\.io/os"=linux \
    --set controller.admissionWebhooks.patch.nodeSelector."kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."kubernetes\.io/os"=linux \
    --set controller.ingressClass=app2 \
    --set controller.ingressClassResource.name=app2 \
    --set controller.ingressClassResource.controllerValue="k8s.io/ingress-nginx"

kubectl wait --namespace app2-ingress \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s

# Name to associate with public IP address
DNSNAME="app2-<uniquename>"

# Get the ingress public IP address
PUBLIC_IP="xx.xx.xx.xx"

# Get the resource-id of the public ip
PUBLICIPID=$(az network public-ip list --query "[?ipAddress!=null]|[?contains(ipAddress, '$PUBLIC_IP')].[id]" --output tsv)

# Update public ip address with dns name
az network public-ip update --ids $PUBLICIPID --dns-name $DNSNAME

kubectl apply -f app2-ingress.yaml -n app2
```
