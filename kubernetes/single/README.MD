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

















