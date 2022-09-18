


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


KUBECONFIG=./tcss.conf oc config set-cluster tcss --server=https:/175.178.27.45:8443 --certificate-authority=./kube-apiserver/ca.crt --embed-certs=true --kubeconfig=./tcss.conf

# 创建并设置用户配置
KUBECONFIG=./tcss.conf oc config set-credentials tcss --client-certificate=tcss.crt --client-key=tcss.key --embed-certs=true --kubeconfig=./tcss.conf

# 设置context配置
KUBECONFIG=./tcss.conf oc config set-context tcss@tcss --cluster=tcss --user=tcss --kubeconfig=./tcss.conf

# 切换context配置
KUBECONFIG=./tcss.conf oc config use-context tcss@tcss --kubeconfig=./tcss.conf

oc cluster up --public-hostname=175.178.27.45 --base-dir=/opt/clus2 --skip-registry-check
--https-proxy=175.178.27.45:8443 --public-hostname=175.178.27.45


KUBECONFIG=./kube-apiserver/admin.kubeconfig oc get node




# 创建User私钥 tcss.key。
openssl genrsa -out ./tcss.key 2048
# 创建证书签署请求 tcss.csr
openssl req -new -key ./tcss.key -out ./tcss.csr -subj "/O=K8s/CN=tcss"
# 签署证书 生成 tcss.crt
openssl x509 -req -in ./tcss.csr -CA ./kube-apiserver/ca.crt -CAkey ./kube-apiserver/ca.key -CAcreateserial -out ./tcss.crt -days 365

# 创建并设置集群配置
oc config set-cluster tcss --server=$API_SERVER --certificate-authority=$CA_FILE --embed-certs=true --kubeconfig=$KUBECONFIG_TARGET
# 创建并设置用户配置
oc config set-credentials tcss --client-certificate=./tcss.crt --client-key=./tcss.key --embed-certs=true --kubeconfig=$KUBECONFIG_TARGET
# 设置context配置
oc config set-context tcss@tcss --cluster=tcss --user=tcss --kubeconfig=$KUBECONFIG_TARGET
# 切换context配置
oc config use-context tcss@tcss --kubeconfig=$KUBECONFIG_TARGET

echo "generate KUBECONFIG file success. $KUBECONFIG_TARGET"

