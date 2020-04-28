# 1、下载
```
wget https://raw.githubusercontent.com/containous/traefik/v1.7/examples/k8s/traefik-ds.yaml
wget https://raw.githubusercontent.com/containous/traefik/v1.7/examples/k8s/traefik-rbac.yaml
wget https://raw.githubusercontent.com/containous/traefik/v1.7/examples/k8s/ui.yaml
```

# 2、部署
```
kubectl apply -f traefik-rbac.yaml
kubectl apply -f traefik-ds.yaml
kubectl apply -f ui.yaml
```
