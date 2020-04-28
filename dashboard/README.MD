# 1、配置ssl证书

上传ssl证书dashboard.crt和dashboard.key
创建secret
```
kubectl create secret generic kubernetes-dashboard-certs --from-file="./dashboard.crt,./dashboard.key" -n kubernetes-dashboard
```

# 2、下载
```
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```

# 3、修改Service类型为NodePort
```
vim recommended.yaml

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 31258
  selector:
    k8s-app: kubernetes-dashboard
```

# 4、部署
```
kubectl apply -f recommended.yaml
```
