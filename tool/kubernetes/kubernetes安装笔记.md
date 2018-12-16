# CentOS7mini 安装 Kubernetes

## 安装CentOS

1. [CentOS官网](https://www.centos.org/download/) 选择 **Minimal ISO**
1. 安装**net-tools**
``` bash
[root@localhost ~]# yum install -y net-tools
```
3. 关闭firewalld

``` bash
[root@localhost ~]# systemctl stop firewalld && systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
[root@localhost ~]# setenforce 0
[root@localhost ~]# sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

4. 安装iptables

``` bash
yum install -y iptables-services
systemctl enable iptables
systemctl start iptables
```

## 安装Docker

1. 添加源

``` bash
cat > /etc/yum.repos.d/docker.repo << EOF
[dockerrepo] 
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7 
enabled=1 
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg 
EOF
```

2. 安装docker

``` bash
[root@localhost ~]# yum install docker-engine
```

3. 查看docker版本
> docker version 报错：*Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?*
> > [在这里找到了答案:](https://forums.docker.com/t/cannot-connect-to-the-docker-daemon-is-the-docker-daemon-running-on-this-host/8925/4) **sudo usermod -aG docker $USER**

> > 重新启动docker : **systemctl restart docker**

``` bash
[root@localhost ~]# docker version
Client:
 Version:      17.05.0-ce
 API version:  1.29
 Go version:   go1.7.5
 Git commit:   89658be
 Built:        Thu May  4 22:06:25 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.05.0-ce
 API version:  1.29 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   89658be
 Built:        Thu May  4 22:06:25 2017
 OS/Arch:      linux/amd64
 Experimental: false
```

## 安装Docker

> 如今Docker分为了Docker-CE和Docker-EE两个版本，CE为社区版即免费版，EE为企业版即商业版。我们选择使用CE版。
[官方安装地址](https://docs.docker.com/engine/installation/linux/docker-ce/centos/#install-docker-ce)

1. 安装yum源工具包

``` bash
[root@localhost ~]# yum install -y yum-utils device-mapper-persistent-data lvm2
```

2. 下载docker-ce官方的yum源配置文件

``` bash
[root@localhost ~]# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

3. 禁用docker-c-edge源配edge是不开发版，不稳定，下载stable版

``` bash
yum-config-manager --disable docker-ce-edge
```

4. 更新本地YUM源缓存

``` bash
yum makecache fast
```

5. 安装Docker-ce相应版本的

``` bash
yum -y install docker-ce
```

6. 运行hello world
``` bash
[root@localhost ~]# systemctl start docker
[root@localhost ~]# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9a0669468bf7: Pull complete
Digest: sha256:0e06ef5e1945a718b02a8c319e15bae44f47039005530bc617a5d071190ed3fc
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://cloud.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

## 安装kubectl工具

1. 如果可以上谷歌,直接执行下面的内容

``` bash
[root@localhost ~]# curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 49.8M  100 49.8M    0     0   574k      0  0:01:28  0:01:28 --:--:--  674k
[root@localhost ~]#
[root@localhost ~]# ls
anaconda-ks.cfg  kubectl  original-ks.cfg
[root@localhost ~]# chmod a+x ./kubectl
[root@localhost ~]#
[root@localhost ~]# ls
anaconda-ks.cfg  kubectl  original-ks.cfg
[root@localhost ~]# mv kubectl /usr/local/bin/kubectl
```

2. 配置阿里源(正常安装)

``` bash
cat >> /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
EOF
```

3. 安装kubectl

``` bash
[root@localhost ~]# yum list  kubectl kubelet kubeadm kubernetes-cni
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.zju.edu.cn
 * extras: mirrors.btte.net
 * updates: mirrors.btte.net
可安装的软件包
kubeadm.x86_64                                                     1.7.5-0                                              kubernetes
kubectl.x86_64                                                     1.7.5-0                                              kubernetes
kubelet.x86_64                                                     1.7.5-0                                              kubernetes
kubernetes-cni.x86_64                                              0.5.1-0                                              kubernetes
```

## 安装kubelet与kubeadm包

1. 谷歌源
``` bash
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

yum install -y kubelet kubeadm
```

## 使用kubeadm init命令初始化集群之下载Docker镜像到所有主机的实始化时会下载kubeadm必要的依赖镜像，同时安装etcd,kube-dns,kube-proxy,由于我们GFW防火墙问题我们不能直接访问，因此先通过其它方法下载下面列表中的镜像，然后导入到系统中，再使用kubeadm init来初始化集群

1. 使用[DaoCloud](https://www.daocloud.io/mirror#accelerator-doc)加速器(可以跳过这一步)

``` bash
[root@localhost ~]# curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://0d236e3f.m.daocloud.io
docker version >= 1.12
{"registry-mirrors": ["http://0d236e3f.m.daocloud.io"]}
Success.
You need to restart docker to take effect: sudo systemctl restart docker
[root@localhost ~]# systemctl restart docker
```

2. 下载镜像,自己通过[Dockerfile](https://github.com/ChamPly/kubernetes-lib)到[dockerhub](https://hub.docker.com/)生成对镜像
> 具体步骤:<https://mritd.me/2016/10/29/set-up-kubernetes-cluster-by-kubeadm/>

``` bash
images=(kube-controller-manager-amd64 etcd-amd64 k8s-dns-sidecar-amd64 kube-proxy-amd64 kube-apiserver-amd64 kube-scheduler-amd64 pause-amd64 k8s-dns-dnsmasq-nanny-amd64 k8s-dns-kube-dns-amd64)
for imageName in ${images[@]} ; do
  docker pull champly/$imageName
  docker tag champly/$imageName gcr.io/google_containers/$imageName
  docker rmi champly/$imageName
done
```

``` bash
docker tag gcr.io/google_containers/etcd-amd64 gcr.io/google_containers/etcd-amd64:3.0.17 && \
docker rmi gcr.io/google_containers/etcd-amd64 && \
docker tag gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64 gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.5 && \
docker rmi gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64 && \
docker tag gcr.io/google_containers/k8s-dns-kube-dns-amd64 gcr.io/google_containers/k8s-dns-kube-dns-amd64:1.14.5 && \
docker rmi gcr.io/google_containers/k8s-dns-kube-dns-amd64 && \
docker tag gcr.io/google_containers/k8s-dns-sidecar-amd64 gcr.io/google_containers/k8s-dns-sidecar-amd64:1.14.2 && \
docker rmi gcr.io/google_containers/k8s-dns-sidecar-amd64 && \
docker tag gcr.io/google_containers/kube-apiserver-amd64 gcr.io/google_containers/kube-apiserver-amd64:v1.7.5 && \
docker rmi gcr.io/google_containers/kube-apiserver-amd64 && \
docker tag gcr.io/google_containers/kube-controller-manager-amd64 gcr.io/google_containers/kube-controller-manager-amd64:v1.7.5 && \
docker rmi gcr.io/google_containers/kube-controller-manager-amd64 && \
docker tag gcr.io/google_containers/kube-proxy-amd64 gcr.io/google_containers/kube-proxy-amd64:v1.6.0 && \
docker rmi gcr.io/google_containers/kube-proxy-amd64 && \
docker tag gcr.io/google_containers/kube-scheduler-amd64 gcr.io/google_containers/kube-scheduler-amd64:v1.7.5 && \
docker rmi gcr.io/google_containers/kube-scheduler-amd64 && \
docker tag gcr.io/google_containers/pause-amd64 gcr.io/google_containers/pause-amd64:3.0 && \
docker rmi gcr.io/google_containers/pause-amd64
```

kubeadm join --token 49a448.4bc5610d78adc32a 192.168.0.18:6443