## 集群搭建前

#### 禁用selinux
```
sed -i 's/SELINUX=permissive/SELINUX=disabled/' /etc/sysconfig/selinux
setenforce 0
```
#### 关闭swap
 
```
swapoff -a
或者修改/etc/fstab
```

#### 开启forward
- Docker从1.13版本开始调整了默认的防火墙规则
- 禁用了iptables filter表中FOWARD链,这样会引起Kubernetes集群中跨Node的Pod无法通信
```
iptables -P FORWARD ACCEPT
```

#### 配置转发相关参数，否则可能会出错
```
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
vm.swappiness=0
EOF
```
```
sysctl --system
```

#### 加载ipvs相关内核模块,如果重新开机，需要重新加载
```
modprobe ip_vs
modprobe ip_vs_rr
modprobe ip_vs_wrr
modprobe ip_vs_sh
modprobe nf_conntrack_ipv4
lsmod | grep ip_vs
```
