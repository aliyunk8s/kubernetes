# 一、服务器环境配置
**1、安装epel源**
```
yum -y install epel-release
```
**2、常用工具安装**
```
yum -y install vim wget nmap lsof iotop lrzsz ntpdate tree
```
**3、修改服务器时区，时间同步**
```
rm -rf /etc/localtime
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
ntpdate time1.aliyun.com
crontab -e
*/3 * * * * /usr/sbin/ntpdate time1.aliyun.com >/dev/null 2>&1
```
**4、关闭防火墙、selinux**
```
systemctl disable firewalld.service
systemctl stop firewalld.service
sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config && setenforce 0
```
**5、关闭swap**
```
swapoff -a
sed -i 's/^.*swap/#&/g'  /etc/fstab
```
**6、内核参数优化**
```
vim /etc/sysctl.conf

net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 1024 65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_fin_timeout = 30
vm.max_map_count = 262144
fs.file-max = 655350
net.core.somaxconn=20480
net.ipv4.ip_forward = 1

sysctl -p
```
**7、调整文件描述符和进程数量的限制**
```
vim /etc/security/limits.conf

*    soft    nofile   655350
*    hard    nofile   655350
*    soft    nproc    655350
*    hard    nproc    655350
```
**8、关闭内存transparent_hugepage特性**
```
cat << 'EOF' >> /etc/rc.local
echo never > /sys/kernel/mm/transparent_hugepage/enabled
EOF
echo never > /sys/kernel/mm/transparent_hugepage/enabled
chmod +x /etc/rc.local
```

# 二、安装docker-ce服务
**卸载服务器可能已存在的docker环境**
```
yum -y remove docker-ce docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-selinux docker-engine-selinux docker-engine docker-ce-cli
```
**2、安装docker依赖包**
```
yum -y install yum-utils device-mapper-persistent-data lvm2
```
**3、添加docker的yum源**
```
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
**4、安装docker-ce服务**
```
yum search docker-ce --showduplicates
yum -y install --setopt=obsoletes=0 docker-ce-18.09.8-3.el7.x86_64
```
**5、配置启动docker服务**
```
vim /etc/docker/daemon.json

{
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "100m",
        "max-file": "10"
    },
    "bip": "169.254.123.1/24",
    "oom-score-adjust": -1000,
    "registry-mirrors": ["https://pqbap4ya.mirror.aliyuncs.com"],
    "storage-driver": "overlay2",
    "storage-opts":["overlay2.override_kernel_check=true"],
    "live-restore": true
}

systemctl daemon-reload
systemctl enable docker
systemctl start docker
```

# 三、安装kubeadm、kubectl、kubelet(v1.16.2)
**1、添加yum源**
```
echo "[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg" >/etc/yum.repos.d/kubernetes.repo
```
**2、卸载服务器上可能已存在的安装**
```
yum -y remove kubelet kubeadm kubectl
```
**3、安装kubeadm、kubectl、kubelet**
```
yum -y install kubelet-1.16.2-0.x86_64 kubeadm-1.16.2-0.x86_64 kubectl-1.16.2-0.x86_64 --disableexcludes=kubernetes
systemctl daemon-reload
systemctl enable kubelet
```
**4、开启bridge-nf模式**
```
cat >> /etc/sysctl.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl -p
echo 1 >/proc/sys/net/bridge/bridge-nf-call-iptables
echo 1 >/proc/sys/net/bridge/bridge-nf-call-ip6tables
```
**5、开启IP数据包转发功能**
```
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p
```

# 五、准备kubernetes服务镜像
```
docker pull registry.aliyuncs.com/google_containers/kube-apiserver:v1.16.2
docker pull registry.aliyuncs.com/google_containers/kube-controller-manager:v1.16.2
docker pull registry.aliyuncs.com/google_containers/kube-scheduler:v1.16.2
docker pull registry.aliyuncs.com/google_containers/kube-proxy:v1.16.2
docker pull registry.aliyuncs.com/google_containers/etcd:3.3.15-0
docker pull registry.aliyuncs.com/google_containers/pause:3.1
docker pull registry.aliyuncs.com/google_containers/coredns:1.6.2
docker pull quay.io/coreos/flannel:v0.12.0-amd64

docker tag registry.aliyuncs.com/google_containers/kube-apiserver:v1.16.2 k8s.gcr.io/kube-apiserver:v1.16.2
docker tag registry.aliyuncs.com/google_containers/kube-scheduler:v1.16.2 k8s.gcr.io/kube-scheduler:v1.16.2
docker tag registry.aliyuncs.com/google_containers/kube-controller-manager:v1.16.2 k8s.gcr.io/kube-controller-manager:v1.16.2
docker tag registry.aliyuncs.com/google_containers/kube-proxy:v1.16.2 k8s.gcr.io/kube-proxy:v1.16.2
docker tag registry.aliyuncs.com/google_containers/etcd:3.3.15-0 k8s.gcr.io/etcd:3.3.15-0
docker tag registry.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1
docker tag registry.aliyuncs.com/google_containers/coredns:1.6.2 k8s.gcr.io/coredns:1.6.2

docker rmi registry.aliyuncs.com/google_containers/kube-apiserver:v1.16.2
docker rmi registry.aliyuncs.com/google_containers/kube-controller-manager:v1.16.2
docker rmi registry.aliyuncs.com/google_containers/kube-scheduler:v1.16.2
docker rmi registry.aliyuncs.com/google_containers/kube-proxy:v1.16.2
docker rmi registry.aliyuncs.com/google_containers/etcd:3.3.15-0
docker rmi registry.aliyuncs.com/google_containers/pause:3.1
docker rmi registry.aliyuncs.com/google_containers/coredns:1.6.2
```



























