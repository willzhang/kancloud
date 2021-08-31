## 脚本快速安装docker
官方提供了安装脚本来快速安装最新版本docker-ce:
```shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
systemctl enable --now docker
docker version
```
参考：
[https://github.com/docker/docker-install](https://github.com/docker/docker-install)
[https://docs.docker.com/engine/install/centos/#install-using-the-convenience-script](https://docs.docker.com/engine/install/centos/#install-using-the-convenience-script)

## 官方安装Docker-CE

官方安装参考：[https://docs.docker.com/engine/installation/](https://docs.docker.com/engine/installation/)

**安装环境：**

- 操作系统：CentOS7 CentOS8
- Docker版本：Docker-CE最新版本

如果安装较旧版本docker，执行以下命令直接安装即可：
```shell
yum install -y docker
systemctl enable --now docker
```

以下是按照官方文档安装最新版本docker的方法：

**卸载旧版本docker：**

如果存在旧版docker可以执行以下命令卸载，没有则跳过。
```python
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

**安装必要的依赖包：**

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```

**配置Docker yum源：**

这里使用国内阿里云Yum源：
```shell
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
也可以配置官方yum源：

```shell
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
**安装Docker-CE：**

```shell
yum install -y docker-ce docker-ce-cli containerd.io
```
**查看docker版本，确认安装成功**

```java
$ docker --version
Docker version 18.09.0, build 4d60db4
```

**启动Docker服务并设为开机启动**

```shell
$ systemctl enable --now docker

#确认docker服务运行正常，显示active (running)说明服务正常运行
$ systemctl status docker
```
**运行容器测试**

```shell
#运行容器
docker run -d --name nginx-app -p 80:80 --restart always nginx

#查看容器运行状态
# docker ps -a

#启动和重启
docker start/stop/restart nginx

#删除和强制删除
docker rm nginx
docker rm -f nginx

#拉取和查看镜像
docker pull nginx
docker images

#删除镜像
docker rmi nginx
```
浏览器能够成功访问 http://<本机ip>:80说明运行正常。

**安装Docker-CE指定版本（可选）**

列出可用版本，然后选择并安装：

```python
[root@host2 ~]# yum list docker-ce --showduplicates | sort -r
......
docker-ce.x86_64            3:18.09.0-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:18.09.0-3.el7                    @docker-ce-stable
docker-ce.x86_64            18.06.1.ce-3.el7                   docker-ce-stable 
docker-ce.x86_64            18.06.0.ce-3.el7                   docker-ce-stable 
docker-ce.x86_64            18.03.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            18.03.0.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.12.1.ce-1.el7.centos            docker-ce-stable 
......
[root@host2 ~]# 

#以安装18.06版本为例
[root@host2 ~]# yum install docker-ce-18.06.1.ce-3.el7  
```

##  配置国内镜像加速（可选）

鉴于国内网络问题，后续拉取 Docker 镜像比较慢，建议安装 Docker 之后配置国内镜像加速。

阿里云镜像加速
```shell
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://uyah70su.mirror.aliyuncs.com"]
}
EOF
```

Azure 中国镜像加速
```shell
{
  "registry-mirrors": ["https://dockerhub.azk8s.cn"],
}
```
清华源镜像加速
```shell
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```
daocloud镜像加速
```shell
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
```
配置完成后重启docker服务
```shell
systemctl daemon-reload && systemctl restart docker
```

检查加速配置是否生效
执行 $ docker info，如果从结果中看到了如下内容，说明配置成功。
```shell
$ docker info | grep Mirrors -A1
Registry Mirrors:
 https://uyah70su.mirror.aliyuncs.com/
```
测试镜像拉取速度
```shell
$ time docker pull centos
```

## centos8安装docker-ce
```shell
#配置阿里云yum源
dnf config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#安装containerd.io
dnf install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.13-3.1.el7.x86_64.rpm

#安装docker-ce
dnf install -y docker-ce

#配置docker镜像加速
mkdir -p /etc/docker
cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["https://uyah70su.mirror.aliyuncs.com"]
}
EOF

#启动docker服务
systemctl enable --now docker
```

## 离线安装docker
[https://github.com/willzhang/shell/tree/master/docker](https://github.com/willzhang/shell/tree/master/docker)

```shell
sh docker.sh
```