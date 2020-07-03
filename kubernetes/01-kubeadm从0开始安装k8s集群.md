# kubeadm安装k8s集群
## 1. 参考文档
* __docker官方__ 在Ubuntu中[安装docker](https://docs.docker.com/engine/install/ubuntu/ "Install Docker Engine on Ubuntu")
* __kubernetes官方__ [安装kubeadm](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/ "install-kubeadm")设置工具
## 2. 服务器列表
* 内存：建议2G及以上
* CPU：建议2核及以上

|  主机名   | IP  | 角色  | 系统  |
|  ----  | ----  | ----  | ----  |
| ubuntu01  | 192.168.2.11 | master | ubuntu18.04 |
| ubuntu02  | 192.168.2.12 | node | ubuntu18.04 |
| ubuntu03  | 192.168.2.13 | node | ubuntu18.04 |
## 3. 提前准备
* 关闭交换分区
```bash
sudo swapoff -a
sudo sed -ri 's/.*swap.*/#&/' /etc/fstab
```
* 关闭防火墙
```bash
sudo ufw disable
```
* 修改内核配置
```bash
$ sudo bash -c "cat > /etc/modules-load.d/containerd.conf" <<EOF
overlay
br_netfilter
EOF
$ sudo modprobe overlay
$ sudo modprobe br_netfilter
$ sudo bash -c "cat > /etc/sysctl.d/k8s.conf" <<EOF
bridge-nf-call-ip6tables = 1
bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
$ sudo sysctl --system
```
## 4. 安装docker
* 基础安装
```bash
$ sudo apt-get update
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```
* 配置docker
```bash
sudo bash -c "cat > /etc/docker/daemon.json" <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```
* 重启docker
```bash
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```
## 5. 安装kubeadm
```bash
$ curl -s http://mirrors.aliyun.com/kubernetes/apt/doc//apt-key.gpg | sudo apt-key add -
$ sudo cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
$ sudo apt-get install -y kubelet kubeadm kubectl
$ sudo apt-mark hold kubelet kubeadm kubectl
```
## 6. 创建主节点
* 初始化主节点
```bash
$ sudo kubeadm init --apiserver-advertise-address=192.168.2.11 --image-repository=registry.cn-hangzhou.aliyuncs.com/kube_open_images --kubernetes-version=v1.18.5 --pod-network-cidr=172.20.0.0/16
```
> apiserver-advertise-address 主节点地址（多个网络接口需要指定）
> image-repository 由于众所周知的原因，指定能够访问的仓库
> kubernetes-version kubernetes版本号
> pod-network-cidr pod网络范围
* 配置普通用户运行kubectl
```bash
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
* 安装pod网络插件
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
## 添加节点
* 在主节点初始化成功后，会有添加节点的提示，可以拷贝使用
* 使用新token添加节点
```bash
$ sudo kubeadm token create
# 生产token,如：whwmg4.6e0oc8i9f2eqsnye
$ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null |
   openssl dgst -sha256 -hex | sed 's/^.* //'
# 生成token hash,如：0da69dcaf283c9ea86663dc798414614687f8d9991090b4af3ef859ba9f5f4a
$ kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
# 如：sudo kubeadm join 192.168.2.11:6443 --token whwmg4.6e0oc8i9f2eqsnye     --discovery-token-ca-cert-hash sha256:0da69dcaf283c9ea86663dc798414614687f8d9991090b4af3ef859ba9f5f4a
```