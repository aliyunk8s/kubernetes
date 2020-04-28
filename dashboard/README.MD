# 1、配置ssl证书

上传ssl证书dashboard.crt和dashboard.key,并生成secret
```
kubectl create ns kubernetes-dashboard
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

# 5、创建授权账号和Token
```
cat > dashboard-admin.yaml << EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard
  namespace: kubernetes-dashboard
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dashboard
subjects:
  - kind: ServiceAccount
    name: dashboard
    namespace: kubernetes-dashboard
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
EOF

kubectl apply -f dashboard-admin.yaml
```
查看Token
```
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep dashboard-token | awk '{print $1}' |grep -v kubernetes) |grep token: |awk '{print $2}'
```

# 6、配置kubeconfig登录
```
dash_tocken=$(kubectl get secret -n kubernetes-dashboard dashboard-token-sjs7z -o jsonpath={.data.token}|base64 -d)
kubectl config set-cluster kubernetes --server=192.168.1.210:6443 --kubeconfig=/root/dashbord-admin.conf
kubectl config set-credentials dashboard-admin --token=$dash_tocken --kubeconfig=/root/dashbord-admin.conf 
kubectl config set-context dashboard-admin@kubernetes --cluster=kubernetes --user=dashboard-admin --kubeconfig=/root/dashbord-admin.conf
kubectl config use-context dashboard-admin@kubernetes --kubeconfig=/root/dashbord-admin.conf
```

