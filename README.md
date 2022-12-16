# kong-headers

Playground to test removal of the Kong-Debug headers

## Installation

```sh
kubectl create namespace kong

kubectl kustomize kubernetes/overlay --enable-helm | kubectl apply --force --filename -

# Wait until all pods are RUNNING, check with:
kubectl get pods -A 

kubectl port-forward --namespace kong service/ingress-kong-proxy 8000:80 &
```

## Query endpoints

```sh
# Admin endpoint
curl -i "http://localhost:8000/admin"
curl -H "Kong-Debug: 1" -i "http://localhost:8000/admin"

# Other endpoint
curl -i "http://localhost:8000/echo-server"
curl -H "Kong-Debug: 1" -i "http://localhost:8000/echo-server"
```

## Enable/disable Kong plugins

Simply comment/uncomment the following line in [kubernetes/base/kustomization.yml](kubernetes/base/kustomization.yml):

```yaml
resources:
    - cluster_plugins_kong.yml
```

## Cleanup

```sh
kubectl kustomize kubernetes/overlay --enable-helm | kubectl delete --filename -

kubectl get daemonsets.apps -n kube-system --no-headers=true | awk '/svclb-ingress-kong-proxy/{print $1}'| xargs  kubectl delete -n kube-system daemonsets.apps

kubectl delete namespace kong

sudo kill $(lsof -ti:8000)
```
