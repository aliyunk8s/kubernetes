# 1、在n节点上创建一块新硬盘，格式化挂载
```
mkfs.xfs /dev/sdb
DISK_UUID=$(blkid -s UUID -o value /dev/sdb)
mkdir -p /data/$DISK_UUID
mount -t xfs /dev/sdb /data/$DISK_UUID
echo UUID=`blkid -s UUID -o value /dev/sdb` /data/$DISK_UUID xfs defaults 0 0 | tee -a /etc/fstab
for i in $(seq 1 20); do
mkdir -p /data/${DISK_UUID}/vol${i} /data/disks/${DISK_UUID}_vol${i}
mount --bind /data/${DISK_UUID}/vol${i} /data/disks/${DISK_UUID}_vol${i}
done
for i in $(seq 1 20); do
echo /data/${DISK_UUID}/vol${i} /data/disks/${DISK_UUID}_vol${i} none bind 0 0 | tee -a /etc/fstab
done
```
# 2、配置部署local-storage
```
wget https://raw.githubusercontent.com/pingcap/tidb-operator/master/manifests/local-dind/local-volume-provisioner.yaml
sed -i 's/mnt/data/g' local-volume-provisioner.yaml
sed -i '/WaitForFirstConsumer/a\reclaimPolicy: Retain' local-volume-provisioner.yaml
kubectl apply -f local-volume-provisioner.yaml
```
