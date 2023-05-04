### 									K8s kubeadmin安装(1.23.6)



#### 1.关闭防火墙

```
systemctl stop firewalld
systemctl disable firewalld
```

#### 2.关闭selinux

```
sed -i 's/enforcing/disabled/*' /etc/selinux/config
```

#### 3.关闭swap

```
swapoff -a  #临时
sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab  #永久
关闭swap后，一定要重启虚拟机
```

#### 4.添加主机hosts

```
cat >> /etc/hosts << EOF
192.168.1.110 k8s-master
192.168.1.111 k8s-node1
192.168.1.110 k8s-node2
EOF
```

#### 5.IPv4 和 IPv6的桥接配置

```
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system  #生效
```

#### 6.时间同步

```
# 修改时区（创建软连接）
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# 将硬件写入主板
hwclock -w 
yum install ntpdate -y
ntpdate time.windows.com
```

#### 7.添加docker仓库阿里云yum源

```
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=kubernetes
enabled=1
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
EOF


wget https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

rpm --import rpm-package-key.gpg

yum repolist （更新本地源）
```

#### 8.安装kubeadm、kubelet、kubectl

```
yum -y install kubelet-1.23.6 kubeadm-1.23.6 kubectl-1.23.6
systemctl enable kubelet
```

#### 9.k8s初始化master节点

```
kubeadm init \
--apiserver-advertise-address=192.168.1.110 \
--image-repository registry.aliyumcs.com/google_containers \
--kubernetes-version v1.23.6 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16
```

#### 10.安装排查

```
yum安装失败:
		yum makecache   清yum缓存
		yum clean all   清空所有yum缓存
系统日志查看
		journalctl -xefu kubelet
查看docker驱动信息更改为systemd 禁用cggroup
		docker info | grep Driver
vim /etc/docker/daemon.json
	添加 "exec-opts":["native.cgroupdriver=systemd"]	
systemctl daemon-reload	
systemctl restart docker
systemctl restart kubelet
k8s 重置安装
	kubeadm reset -y

```

#### 11.安装成功后，复制如下配置并执行

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubenetes/admin.conf $HOME/.kube/config
sudo chwon $(id -u):$(id -g) $HOME/.kube/config
kubectl get nodes
```

#### 12.节点加入集群

```
kubeadm join 192.168.1.100:6443 --token <master控制台的> \
--discovery-token-ca-cert-hash <master控制台的hash>
#如果初始化的token不小心清空了
kubeadm token list
kubeadm token create
#获取hash值
openssl x509 -pubkey -in /etc/kubenetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
openssl dgst -sha256 -hex | sed 's/^.* //'
hash值: sha256:hash值
```

####   13.部署CNI网络插件

```
#在master节点上执行
#下载calico配置文件,可能会网络超时
curl https://docs.projectcalico.org/manifests/calico.yaml -O
#修改calico.yaml文件中的CALICO_IPV4POOL_CIDR配置,配置为master初始化POD cidr
#修改 IP_AUTODETECTION_METHOD下的网卡名称
grep image calico.yaml
#删除镜像docker.io/前缀，避免下载过慢导致失败
sed -i 's#docker.io/##g' calico.yaml
kubectl apply -f calico.yaml
kubectl get pod -n kube-system
```

#### 14.验证集群

```
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get pod,svc
```

#### 15.节点配置kubectl使用

```
1.将master节点中/etc/kubenetes/admin.conf 拷贝到运行的服务器的 /etc/kubenetes目录中
	scp /etc/kubenetes/admin.conf root@k8s-node1:/etc/kubenetes/
2.在对应的服务器上配置环境变量
    echo "export KUBECONFIG=/etc/kubenetes/admin.conf" >> ~/.bash_profile
    source .bash_profile
```

