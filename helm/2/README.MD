# 1、下载安装
```
wget https://get.helm.sh/helm-v2.16.6-linux-amd64.tar.gz
tar -xf helm-v2.16.6-linux-amd64.tar.gz
cp linux-amd64/{helm,tiller} /usr/local/bin/
```

# 2、初始化安装tiller服务
```
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account tiller --override spec.selector.matchLabels.'name'='tiller',spec.selector.matchLabels.'app'='helm' --output yaml | sed 's@apiVersion: extensions/v1beta1@apiVersion: apps/v1@' |sed 's/gcr.io\/kubernetes-helm/registry.cn-hangzhou.aliyuncs.com\/google_containers/' | kubectl apply -f -
```

# 3、配置helm仓库
```
mkdir -p ~/.helm/repository/cache
```
复制repositories.yaml到~/.helm/repository/下
```
helm repo add elastic https://helm.elastic.co
helm repo add microsoft http://mirror.azure.cn/kubernetes/charts
helm repo add aliyun https://apphub.aliyuncs.com
helm repo update
```

# 4、helm常用操作命令
```
mkdir -p ~/.helm/cache/archive
cd ~/.helm/cache/archive
helm search redis
helm fetch aliyun/redis
tar -xf redis-10.5.3.tgz
cd redis
helm lint
helm install --name redis --namespace prod .
helm upgrade redis .
helm delete redis purge
```
