### v3 备份

```
CurrentDate=`date -d today +"%Y-%m-%d"`
export ETCDCTL_API=3
etcdctl --endpoints="https://127.0.0.1:2379" --cacert /etc/etcd/ssl/etcd-ca.pem --cert /etc/etcd/ssl/etcd.pem --key /etc/etcd/ssl/etcd-key.pem snapshot save "/tmp/snapshot-$CurrentDate.db"
```
### v3 恢复
```
etcdctl snapshot restore "/tmp/snapshot-$CurrentDate.db" --name pg-docker-11161 --data-dir=/export/etcd/pg-docker-55161/
```

### v2 备份
```
etcdctl backup --data-dir /export/etcd/pg-docker-55161/ --backup-dir /home/etcd_backup
```

### v2 恢复 带参数启动服务
```
etcd -data-dir=/home/etcd_backup/  -force-new-cluster

### 将/home/etcd_backup/下的目录copy到原etcd的数据目录
```
