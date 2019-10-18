
## 创建coreDNS
```
cd $HOME && mkdir coredns && cd coredns
wget https://raw.githubusercontent.com/coredns/deployment/master/kubernetes/coredns.yaml.sed
wget https://raw.githubusercontent.com/coredns/deployment/master/kubernetes/deploy.sh
chmod +x deploy.sh
./deploy.sh -i 10.254.0.10 > coredns.yml
kubectl apply -f coredns.yml

# 查看
kubectl get pods -n kube-system
kubectl get svc -n kube-system
```

#### 到kubernetes/cluster/addons/dashboard目录，create dashboard相关的yaml

## 创建dashboard的token
```
curl -s https://raw.githubusercontent.com/mritd/ktool/master/k8s/addons/dashborad/create_dashboard_sa.sh | bash
```
#### 脚本内容
```
#!/bin/bash

if kubectl get sa dashboard-admin -n kube-system &> /dev/null;then
    echo -e "\033[33mWARNING: ServiceAccount dashboard-admin exist!\033[0m"
else
    kubectl create sa dashboard-admin -n kube-system
    kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
fi

kubectl describe secret -n kube-system $(kubectl get secrets -n kube-system | grep dashboard-admin | cut -f1 -d ' ') | grep -E '^token'

```
