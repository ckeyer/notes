


```
Host shift1
    HostName 43.138.186.106 
    172.16.32.12
    Port 22
    User root
Host shift2
    HostName 43.138.222.49
    172.16.32.11
    Port 22
    User root
Host shift3
    HostName 175.178.27.45
    172.16.32.4
    Port 22
    User root
```

yum install -y wget git net-tools bind-utils yum-utils iptables-services bridge-utils bash-completion kexec-tools sos psacct java-1.8.0-openjdk-headless python-passlib


yum install -y nfs-utils lrzsz gcc gcc-c++ make cmake libxml2-devel openssl-devel curl curl-devel unzip sudo ntp libaio-devel vim ncurses-devel autoconf automake zlib-devel python-devel epel-release lrzsz openssh-server socat ipvsadm conntrack


# master

sed -i -e "s/^enabled=1/enabled=0/" /etc/yum.repos.d/epel.repo


/etc/sysconfig/docker

{"registry-mirrors": ["https://rsbud4vc.mirror.aliyuncs.com","https://registry.docker-cn.com","https://docker.mirrors.ustc.edu.cn","https://dockerhub.azk8s.cn","http://hub-mirror.c.163.com","http://qtid6917.mirror.aliyuncs.com"]}

systemctl daemon-reload
systemctl restart docker.service

175.178.27.45:8443


oc config set-cluster tcss --server=https:/175.178.27.45:8443 --certificate-authority=./kube-apiserver/ca.crt --embed-certs=true --kubeconfig=./tcss.conf

# 创建并设置用户配置
kubectl config set-credentials tcss --client-certificate=tcss.crt --client-key=tcss.key --embed-certs=true --kubeconfig=/root/tcss.conf

# 设置context配置
kubectl config set-context tcss@tcss --cluster=tcss --user=tcss --kubeconfig=/root/tcss.conf

# 切换context配置
kubectl config use-context tcss@tcss --kubeconfig=/root/tcss.conf


