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
yum -y install conntrack-tools ipvsadm ipset

cat >/etc/sysconfig/modules/ipvs.modules<<EOF
#!/bin/bash
ipvs_modules="ip_vs ip_vs_lc ip_vs_wlc ip_vs_rr ip_vs_wrr ip_vs_lblc ip_vs_lblcr ip_vs_dh ip_vs_sh ip_vs_fo ip_vs_nq ip_vs_sed ip_vs_ftp nf_conntrack_ipv4"
for kernel_module in \${ipvs_modules}; do
    /sbin/modinfo -F filename \${kernel_module} > /dev/null 2>&1
    if [ $? -eq 0 ]; then
        /sbin/modprobe \${kernel_module}
    fi
done
EOF

chmod +x /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep ip_vs
```
##### 内核参数参考：
```
net.ipv4.ip_forward = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
kernel.sysrq = 0
kernel.core_uses_pid = 1
net.ipv4.tcp_syncookies = 1
kernel.msgmnb = 409600
kernel.msgmax = 409600
kernel.shmmax = 2122177284
kernel.shmall = 134217728
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 300
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 0
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_max_syn_backlog = 65536
net.ipv4.tcp_max_tw_buckets  = 65535
net.core.netdev_max_backlog =  32768
net.core.somaxconn = 32768
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_mem = 94500000 915000000 927000000
net.ipv4.tcp_max_orphans = 3276800
net.ipv4.ip_local_port_range = 1024  65535
net.ipv4.tcp_window_scaling = 0
net.ipv4.tcp_sack = 0
net.ipv4.tcp_no_metrics_save = 1
net.ipv4.tcp_slow_start_after_idle = 0
fs.file-max = 655350
kernel.core_pattern = /dev/null
net.ipv4.tcp_low_latency = 1

vm.dirty_background_ratio = 5
vm.dirty_ratio = 10
vm.min_free_kbytes=409600
vm.vfs_cache_pressure=200
net.nf_conntrack_max = 25000000
net.netfilter.nf_conntrack_max = 25000000
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_nonlocal_bind = 1
vm.swappiness=0
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
net.ipv4.tcp_sack = 0

net.ipv4.neigh.default.gc_thresh3 = 4096
net.ipv4.neigh.default.gc_thresh2 = 2048
net.ipv4.neigh.default.gc_thresh1 = 1024

```
