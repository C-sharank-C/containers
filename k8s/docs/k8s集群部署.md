## Kubernetes（K8S）简介

### K8S是什么

它是一个由Google开发为容器化应用提供集群部署和管理的开源工具。

### 主要特征

- 高可用，不宕机，自动灾难恢复
- 灰度更新，不影响业务正常运转
- 一键回滚到历史版本
- 方便的伸缩拓展（应用伸缩，机器加减），提供负载均衡
- 完善的生态

### 不同的应用部署方案

![image-20220717181226891](E:\NoteFiles\containers\k8s\images\app_deploy_methods.png)

##### 传统部署方式：

应用直接在物理机上部署，机器资源分配不好控制，出现Bug时，可能机器的大部分资源被某个应用占用，导致其他应用无法正常运行，无法做到应用隔离。

##### 虚拟机部署

在单个物理机上运行多个虚拟机，每个虚拟机都是完整独立的系统，性能损耗大。

##### 容器部署

所有容器共享主机的系统，轻量级的虚拟机，性能损耗小，资源隔离，CPU和内存可按需分配

### 什么时候需要 Kubernetes

当你的应用只是跑在一台机器，直接一个 docker + docker-compose 就够了，方便轻松；
当你的应用需要跑在 3，4 台机器上，你依旧可以每台机器单独配置运行环境 + 负载均衡器；
当你应用访问数不断增加，机器逐渐增加到十几台、上百台、上千台时，每次加机器、软件更新、版本回滚，都会变得非常麻烦、痛不欲生，再也不能好好的摸鱼了，人生浪费在那些没技术含量的重复性工作上。

这时候，Kubernetes 就可以一展身手了，让你轻松管理百万千万台机器的集群。“谈笑间，樯橹灰飞烟灭”，享受着一手掌控所有，年薪百万指日可待。

Kubernetes 可以为你提供集中式的管理集群机器和应用，加机器、版本升级、版本回滚，那都是一个命令就搞定的事，不停机的灰度更新，确保高可用、高性能、高扩展。

### Kubernetes 集群架构

![architecture](E:\NoteFiles\containers\k8s\images\cluster_arch.png)

##### master

主节点，控制平台，不需要很高性能，不跑任务，通常一个就行了，也可以开多个主节点来提高集群可用度。

##### worker

工作节点，可以是虚拟机或物理计算机，任务都在这里跑，机器性能需要好点；通常都有很多个，可以不断加机器扩大集群；每个工作节点由主节点管理

##### 重要概念 Pod

豆荚，K8S 调度、管理的最小单位，一个 Pod 可以包含一个或多个容器，每个 Pod 有自己的虚拟IP。一个工作节点可以有多个 pod，主节点会考量负载自动调度 pod 到哪个节点运行。

![cluster1](E:\NoteFiles\containers\k8s\images\cluster_1.png)

##### Kubernetes 组件

`kube-apiserver` API 服务器，公开了 Kubernetes API
`etcd` 键值数据库，可以作为保存 Kubernetes 所有集群数据的后台数据库
`kube-scheduler` 调度 Pod 到哪个节点运行
`kube-controller` 集群控制器
`cloud-controller` 与云服务商交互

![cluster2](E:\NoteFiles\containers\k8s\images\cluster_2.png)

#  安装 Kubernetes 集群

### 安装方式介绍

- **minikube**
  只是一个 K8S 集群模拟器，只有一个节点的集群，只为测试用，master 和 worker 都在一起
- **直接用云平台 Kubernetes**
  可视化搭建，只需简单几步就可以创建好一个集群。
  优点：安装简单，生态齐全，负载均衡器、存储等都给你配套好，简单操作就搞定
- **裸机安装（Bare Metal）**
  至少需要两台机器（主节点、工作节点各一台），需要自己安装 Kubernetes 组件，配置会稍微麻烦点。
  可以到各云厂商按时租用服务器，费用低，用完就销毁。
  缺点：配置麻烦，缺少生态支持，例如负载均衡器、云存储。

> 本文档课件需配套 [视频](https://www.bilibili.com/video/BV1Tg411P7EB?p=2) 一起学习

### minikube

安装非常简单，支持各种平台，[安装方法](https://minikube.sigs.k8s.io/docs/start/)

> 从官网下载exe文件，（command命令下载无法正常运行）
>
> 需要提前安装好 Docker

```
# 启动集群
minikube start
# 查看节点。kubectl 是一个用来跟 K8S 集群进行交互的命令行工具
kubectl get node
# 停止集群
minikube stop
# 清空集群
minikube delete --all
# 安装集群可视化 Web UI 控制台
minikube dashboard
```

> 以下两种方案成本极高，未实施（doge）

### 云平台搭建

- [腾讯云 TKE](https://cloud.tencent.com/product/tke)（控制台搜索容器）
- 登录阿里云控制台 - 产品搜索 Kubernetes

### 裸机搭建（Bare Metal）

##### 主节点需要组件

- docker（也可以是其他容器运行时）
- kubectl 集群命令行交互工具
- kubeadm 集群初始化工具

##### 工作节点需要组件 [文档](https://kubernetes.io/zh/docs/concepts/overview/components/#node-components)

- docker（也可以是其他容器运行时）
- kubelet 管理 Pod 和容器，确保他们健康稳定运行。
- kube-proxy 网络代理，负责网络相关的工作

#### 开始安装

> 你也可以试下 [这个项目](https://github.com/lework/kainstall)，用脚本快速搭建 K8S 裸机集群
> 当然，为了更好的理解，你应该先手动搭建一次

```
# 每个节点分别设置对应主机名
hostnamectl set-hostname master
hostnamectl set-hostname node1
hostnamectl set-hostname node2
# 所有节点都修改 hosts
vim /etc/hosts
172.16.32.2 node1
172.16.32.6 node2
172.16.0.4 master
# 所有节点关闭 SELinux
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```

> 所有节点确保防火墙关闭
> systemctl stop firewalld
> systemctl disable firewalld

添加安装源（所有节点）

```
# 添加 k8s 安装源
cat <<EOF > kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
mv kubernetes.repo /etc/yum.repos.d/

# 添加 Docker 安装源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

安装所需组件（所有节点）
`yum install -y kubelet-1.22.4 kubectl-1.22.4 kubeadm-1.22.4 docker-ce`

> 注意，据学员反馈，1.24 以上的版本会报错，跟教程有差异，所以建议大家指定版本号安装，版本号确保跟老师的一样

启动 kubelet、docker，并设置开机启动（所有节点）

```
systemctl enable kubelet
systemctl start kubelet
systemctl enable docker
systemctl start docker
```

修改 docker 配置（所有节点）

```
# kubernetes 官方推荐 docker 等使用 systemd 作为 cgroupdriver，否则 kubelet 启动不了
cat <<EOF > daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://ud6340vz.mirror.aliyuncs.com"]
}
EOF
mv daemon.json /etc/docker/

# 重启生效
systemctl daemon-reload
systemctl restart docker
```

用 [kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/) 初始化集群（仅在主节点跑），

```
# 初始化集群控制台 Control plane
# 失败了可以用 kubeadm reset 重置
kubeadm init --image-repository=registry.aliyuncs.com/google_containers

# 记得把 kubeadm join xxx 保存起来
# 忘记了重新获取：kubeadm token create --print-join-command

# 复制授权文件，以便 kubectl 可以有权限访问集群
# 如果你其他节点需要访问集群，需要从主节点复制这个文件过去其他节点
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

# 在其他机器上创建 ~/.kube/config 文件也能通过 kubectl 访问到集群
```

> 有兴趣了解 kubeadm init 具体做了什么的，可以 [查看文档](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/)

把工作节点加入集群（只在工作节点跑）

```
kubeadm join 172.16.32.10:6443 --token xxx --discovery-token-ca-cert-hash xxx
```

安装网络插件，否则 node 是 NotReady 状态（主节点跑）

```
# 很有可能国内网络访问不到这个资源，你可以网上找找国内的源安装 flannel
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

