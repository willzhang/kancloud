## kubesphere简介

KubeSphere是一个以应用为中心的容器平台，完全开源，KubeSphere 帮助企业在云、虚拟化及物理机等任何环境中快速构建、部署和运维基于 Kubernetes 的容器架构，轻松实现微服务治理、多租户管理、DevOps 与 CI/CD、监控日志告警、应用商店、大数据、以及人工智能等业务场景。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200421180455562.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25ldHdvcmtlbg==,size_16,color_FFFFFF,t_70)
官网：[https://kubesphere.io/](https://kubesphere.io/)

项目地址：[https://github.com/kubesphere/kubesphere](https://github.com/kubesphere/kubesphere)

## kubesphere部署方式
kubesphere大致有2种部署方式：

 - 部署k8s集群及kubesphere
 - 已有kubernetes集群部署kubesphere

已有k8s集群部署kubesphere具有更高的灵活性，下面演示单独部署k8s集群，并在集群上部署kubesphere。使用rancher开源的云原生分布式存储longhorn作为底层存储。

## 部署k8s集群
使用sealos工具部署k8s集群，准备4个节点，3个master，1个node，所有节点必须配置主机名，并确认节点时间同步
```shell
hostnamectl set-hostname xx
yum install -y chrony
systemctl enable --now chronyd
timedatectl set-timezone Asia/Shanghai
```
在第一个master节点操作，下载部署工具及离线包
```shell
#基于go的二进制安装程序
wget -c https://sealyun.oss-cn-beijing.aliyuncs.com/latest/sealos && \
    chmod +x sealos && mv sealos /usr/bin

# 以k8s v1.18.8为例，不建议使用v1.19.x,kubesphere暂不支持
wget -c https://sealyun.oss-cn-beijing.aliyuncs.com/cd3d5791b292325d38bbfaffd9855312-1.18.8/kube1.18.8.tar.gz
```
执行以下命令部署集群，passwd为所有节点root密码
```shell
sealos init --passwd 123456 \
  --master 10.39.140.248 \
  --master 10.39.140.249 \
  --master 10.39.140.250 \
  --node 10.39.140.251 \
  --pkg-url kube1.18.8.tar.gz \
  --version v1.18.8
```
确认k8s集群运行正常
```shell
# kubectl get nodes
NAME          STATUS   ROLES    AGE   VERSION
k8s-master1   Ready    master   13h   v1.18.8
k8s-master2   Ready    master   13h   v1.18.8
k8s-master3   Ready    master   13h   v1.18.8
k8s-node1     Ready    <none>   13h   v1.18.8
```
## 部署longhorn存储

longhorn推荐单独挂盘作为存储使用，这里作为测试直接使用本地存储目录/data/longhorn，默认为/var/lib/longhorn。

注意，kubesphere有几个组件申请的pv大小为20G，确保节点空间充足，否则可能出现pv能够绑定成功但没有满足条件的节点可调度的情况。如果仅仅测试环境，可以提前修改cluster-configuration.yaml缩减pv大小。

安装具有3数据副本的longhorn至少需要3个节点，这里去除master节点污点使其可调度pod：

```shell
kubectl taint nodes --all node-role.kubernetes.io/master-  
```

k8s-master1安装helm
```shell
version=v3.3.1
curl -LO https://repo.huaweicloud.com/helm/${version}/helm-${version}-linux-amd64.tar.gz
tar -zxvf helm-${version}-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/helm && rm -rf linux-amd64
```
所有节点安装longhorn依赖
```shell
yum install -y iscsi-initiator-utils
systemctl enable --now iscsid
```
添加longhorn chart，如果网络较差可以longhorn github release下载chart源码
```shell
helm repo add longhorn https://charts.longhorn.io
helm repo update
```

部署longhorn，支持离线部署，需要提前推送镜像到私有仓库longhornio下
```shell
kubectl create namespace longhorn-system

helm install longhorn \
  --namespace longhorn-system \
  --set defaultSettings.defaultDataPath="/data/longhorn/" \
  --set defaultSettings.defaultReplicaCount=3 \
  --set service.ui.type=NodePort \
  --set service.ui.nodePort=30890 \
  #--set privateRegistry.registryUrl=10.39.140.196:8081 \
  longhorn/longhorn
```
确认longhorn运行正常
```shell
[root@jenkins longhorn]# kubectl -n longhorn-system get pods
NAME                                        READY   STATUS    RESTARTS   AGE
csi-attacher-58b856dcff-9kqdt               1/1     Running   0          13h
csi-attacher-58b856dcff-c4zzp               1/1     Running   0          13h
csi-attacher-58b856dcff-tvfw2               1/1     Running   0          13h
csi-provisioner-56dd9dc55b-6ps8m            1/1     Running   0          13h
csi-provisioner-56dd9dc55b-m7gz4            1/1     Running   0          13h
csi-provisioner-56dd9dc55b-s9bh4            1/1     Running   0          13h
csi-resizer-6b87c4d9f8-2skth                1/1     Running   0          13h
csi-resizer-6b87c4d9f8-sqn2g                1/1     Running   0          13h
csi-resizer-6b87c4d9f8-z6xql                1/1     Running   0          13h
engine-image-ei-b99baaed-5fd7m              1/1     Running   0          13h
engine-image-ei-b99baaed-jcjxj              1/1     Running   0          12h
engine-image-ei-b99baaed-n6wxc              1/1     Running   0          12h
engine-image-ei-b99baaed-qxfhg              1/1     Running   0          12h
instance-manager-e-44ba7ac9                 1/1     Running   0          12h
instance-manager-e-48676e4a                 1/1     Running   0          12h
instance-manager-e-57bd994b                 1/1     Running   0          12h
instance-manager-e-753c704f                 1/1     Running   0          13h
instance-manager-r-4f4be1c1                 1/1     Running   0          12h
instance-manager-r-68bfb49b                 1/1     Running   0          12h
instance-manager-r-ccb87377                 1/1     Running   0          12h
instance-manager-r-e56429be                 1/1     Running   0          13h
longhorn-csi-plugin-fqgf7                   2/2     Running   0          12h
longhorn-csi-plugin-gbrnf                   2/2     Running   0          13h
longhorn-csi-plugin-kjj6b                   2/2     Running   0          12h
longhorn-csi-plugin-tvbvj                   2/2     Running   0          12h
longhorn-driver-deployer-74bb5c9fcb-khmbk   1/1     Running   0          14h
longhorn-manager-82ztz                      1/1     Running   0          12h
longhorn-manager-8kmsn                      1/1     Running   0          12h
longhorn-manager-flmfl                      1/1     Running   0          12h
longhorn-manager-mz6zj                      1/1     Running   0          14h
longhorn-ui-77c6d6f5b7-nzsg2                1/1     Running   0          14h
```
确认默认的storageclass已就绪
```shell
# kubectl get sc
NAME                 PROVISIONER          RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
longhorn (default)   driver.longhorn.io   Delete          Immediate           true                   14h
```
登录longhorn UI确认节点处于可调度状态

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200923132200912.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25ldHdvcmtlbg==,size_16,color_FFFFFF,t_70#pic_center)



## 部署kubesphere
参考：

[https://kubesphere.com.cn/en/docs/installing-on-kubernetes/](https://kubesphere.com.cn/en/docs/installing-on-kubernetes/)
[https://github.com/kubesphere/ks-installer](https://github.com/kubesphere/ks-installer)

部署kubesphere 3.0版本，下载yaml文件

```shell
wget https://raw.githubusercontent.com/kubesphere/ks-installer/v3.0.0/deploy/kubesphere-installer.yaml
wget https://raw.githubusercontent.com/kubesphere/ks-installer/v3.0.0/deploy/cluster-configuration.yaml
```
修改cluster-configuration.yaml，找到相应字段开启需要安装的组件，以下仅为参考：
```yaml
  devops:
    enabled: true
    ......
  logging:
    enabled: true
    ......
  metrics_server:
    enabled: true
    ......
  openpitrix:
    enabled: true
    ......
```
执行kubesphere部署
```shell
kubectl apply -f kubesphere-installer.yaml
kubectl apply -f cluster-configuration.yaml
```

查看部署日志，确认无报错

```shell
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```

部署完成后确认所有pod运行正常
```shell
[root@k8s-master1 ~]# kubectl get pods -A | grep kubesphere
kubesphere-controls-system     default-http-backend-857d7b6856-q24v2                             1/1     Running     0          12h
kubesphere-controls-system     kubectl-admin-58f985d8f6-jl9bj                                    1/1     Running     0          11h
kubesphere-controls-system     kubesphere-router-demo-ns-6c97d4968b-njgrc                        1/1     Running     1          154m
kubesphere-devops-system       ks-jenkins-54455f5db8-hm6kc                                       1/1     Running     0          11h
kubesphere-devops-system       s2ioperator-0                                                     1/1     Running     1          11h
kubesphere-devops-system       uc-jenkins-update-center-cd9464fff-qnvfz                          1/1     Running     0          12h
kubesphere-logging-system      elasticsearch-logging-curator-elasticsearch-curator-160079hmdmb   0/1     Completed   0          11h
kubesphere-logging-system      elasticsearch-logging-data-0                                      1/1     Running     0          12h
kubesphere-logging-system      elasticsearch-logging-data-1                                      1/1     Running     0          12h
kubesphere-logging-system      elasticsearch-logging-discovery-0                                 1/1     Running     0          12h
kubesphere-logging-system      fluent-bit-c45h2                                                  1/1     Running     0          12h
kubesphere-logging-system      fluent-bit-kptfc                                                  1/1     Running     0          12h
kubesphere-logging-system      fluent-bit-rzjfp                                                  1/1     Running     0          12h
kubesphere-logging-system      fluent-bit-wztkp                                                  1/1     Running     0          12h
kubesphere-logging-system      fluentbit-operator-855d4b977d-fk6hs                               1/1     Running     0          12h
kubesphere-logging-system      ks-events-exporter-5bc4d9f496-x297f                               2/2     Running     0          12h
kubesphere-logging-system      ks-events-operator-8dbf7fccc-9qmml                                1/1     Running     0          12h
kubesphere-logging-system      ks-events-ruler-698b7899c7-fkn4l                                  2/2     Running     0          12h
kubesphere-logging-system      ks-events-ruler-698b7899c7-hw6rq                                  2/2     Running     0          12h
kubesphere-logging-system      logsidecar-injector-deploy-74c66bfd85-cxkxm                       2/2     Running     0          12h
kubesphere-logging-system      logsidecar-injector-deploy-74c66bfd85-lzxbm                       2/2     Running     0          12h
kubesphere-monitoring-system   alertmanager-main-0                                               2/2     Running     0          11h
kubesphere-monitoring-system   alertmanager-main-1                                               2/2     Running     0          11h
kubesphere-monitoring-system   alertmanager-main-2                                               2/2     Running     0          11h
kubesphere-monitoring-system   kube-state-metrics-95c974544-r8kmq                                3/3     Running     0          12h
kubesphere-monitoring-system   node-exporter-9ddxn                                               2/2     Running     0          12h
kubesphere-monitoring-system   node-exporter-dw929                                               2/2     Running     0          12h
kubesphere-monitoring-system   node-exporter-ht868                                               2/2     Running     0          12h
kubesphere-monitoring-system   node-exporter-nxdsm                                               2/2     Running     0          12h
kubesphere-monitoring-system   notification-manager-deployment-7c8df68d94-hv56l                  1/1     Running     0          12h
kubesphere-monitoring-system   notification-manager-deployment-7c8df68d94-ttdsg                  1/1     Running     0          12h
kubesphere-monitoring-system   notification-manager-operator-6958786cd6-pllgc                    2/2     Running     0          12h
kubesphere-monitoring-system   prometheus-k8s-0                                                  3/3     Running     1          11h
kubesphere-monitoring-system   prometheus-k8s-1                                                  3/3     Running     1          11h
kubesphere-monitoring-system   prometheus-operator-84d58bf775-5rqdj                              2/2     Running     0          12h
kubesphere-system              etcd-65796969c7-whbzx                                             1/1     Running     0          12h
kubesphere-system              ks-apiserver-b4dbcc67-2kknm                                       1/1     Running     0          11h
kubesphere-system              ks-apiserver-b4dbcc67-k6jr2                                       1/1     Running     0          11h
kubesphere-system              ks-apiserver-b4dbcc67-q8845                                       1/1     Running     0          11h
kubesphere-system              ks-console-786b9846d4-86hxw                                       1/1     Running     0          12h
kubesphere-system              ks-console-786b9846d4-l6mhj                                       1/1     Running     0          12h
kubesphere-system              ks-console-786b9846d4-wct8z                                       1/1     Running     0          12h
kubesphere-system              ks-controller-manager-7fd8799789-478ks                            1/1     Running     0          11h
kubesphere-system              ks-controller-manager-7fd8799789-hwgmp                            1/1     Running     0          11h
kubesphere-system              ks-controller-manager-7fd8799789-pdbch                            1/1     Running     0          11h
kubesphere-system              ks-installer-64ddc4b77b-c7qz8                                     1/1     Running     0          12h
kubesphere-system              minio-7bfdb5968b-b5v59                                            1/1     Running     0          12h
kubesphere-system              mysql-7f64d9f584-kvxcb                                            1/1     Running     0          12h
kubesphere-system              openldap-0                                                        1/1     Running     0          12h
kubesphere-system              openldap-1                                                        1/1     Running     0          12h
kubesphere-system              redis-ha-haproxy-5c6559d588-2rt6v                                 1/1     Running     9          12h
kubesphere-system              redis-ha-haproxy-5c6559d588-mhj9p                                 1/1     Running     8          12h
kubesphere-system              redis-ha-haproxy-5c6559d588-tgpjv                                 1/1     Running     11         12h
kubesphere-system              redis-ha-server-0                                                 2/2     Running     0          12h
kubesphere-system              redis-ha-server-1                                                 2/2     Running     0          12h
kubesphere-system              redis-ha-server-2                                                 2/2     Running     0          12h
```

注意，kubesphere部分组件使用helm部署

```shell
[root@k8s-master1 ~]# helm ls -A | grep kubesphere
elasticsearch-logging           kubesphere-logging-system       1               2020-09-23 00:49:08.526873742 +0800 CST deployed        elasticsearch-1.22.1            6.7.0-0217                  
elasticsearch-logging-curator   kubesphere-logging-system       1               2020-09-23 00:49:16.117842593 +0800 CST deployed        elasticsearch-curator-1.3.3     5.5.4-0217                  
ks-events                       kubesphere-logging-system       1               2020-09-23 00:51:45.529430505 +0800 CST deployed        kube-events-0.1.0               0.1.0                       
ks-jenkins                      kubesphere-devops-system        1               2020-09-23 01:03:15.106022826 +0800 CST deployed        jenkins-0.19.0                  2.121.3-0217                
ks-minio                        kubesphere-system               2               2020-09-23 00:48:16.990599158 +0800 CST deployed        minio-2.5.16                    RELEASE.2019-08-07T01-59-21Z
ks-openldap                     kubesphere-system               1               2020-09-23 00:03:28.767712181 +0800 CST deployed        openldap-ha-0.1.0               1.0                         
ks-redis                        kubesphere-system               1               2020-09-23 00:03:19.439784188 +0800 CST deployed        redis-ha-3.9.0                  5.0.5                       
logsidecar-injector             kubesphere-logging-system       1               2020-09-23 00:51:57.519733074 +0800 CST deployed        logsidecar-injector-0.1.0       0.1.0                       
notification-manager            kubesphere-monitoring-system    1               2020-09-23 00:54:14.662762759 +0800 CST deployed        notification-manager-0.1.0      0.1.0                       
uc                              kubesphere-devops-system        1               2020-09-23 00:51:37.885154574 +0800 CST deployed        jenkins-update-center-0.8.0     3.0.0    
```

获取web console 监听端口，默认为30880

```shell
kubectl get svc/ks-console -n kubesphere-system
```

默认登录账号为

```shell
admin/P@88w0rd
```
登录kubesphere UI

![在这里插入图片描述](https://img-blog.csdnimg.cn/202009231301350.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25ldHdvcmtlbg==,size_16,color_FFFFFF,t_70#pic_center)
集群节点信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200923130342758.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25ldHdvcmtlbg==,size_16,color_FFFFFF,t_70#pic_center)
服务组件信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200923130412386.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25ldHdvcmtlbg==,size_16,color_FFFFFF,t_70#pic_center)
longhorn UI查看绑定的pv卷

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020092313235684.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25ldHdvcmtlbg==,size_16,color_FFFFFF,t_70#pic_center)
查看卷详情

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200923132450281.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25ldHdvcmtlbg==,size_16,color_FFFFFF,t_70#pic_center)
## 离线部署kubesphere
1、准备harbor镜像仓库，镜像仓库地址：http://192.168.93.9

2、下载kubesphere离线镜像包并解压
```shell
curl -Ok https://kubesphere-installer.pek3b.qingstor.com/offline/v3.0.0/kubesphere-all-v3.0.0-offline-linux-amd64.tar.gz
tar -zxvf kubesphere-all-v3.0.0-offline-linux-amd64.tar.gz
```
3、推送镜像到harbor仓库

下载脚本
```shell
wget https://raw.githubusercontent.com/kubesphere/ks-installer/master/scripts/create_project_harbor.sh
```
修改create_project_harbor.sh脚本，指定镜像仓库地址和登录信息：
```shell
url="http://192.168.93.9"
user="admin"
passwd="Harbor12345"
```
如果使用2.x版本harbor修改最后行为以下内容:
```shell
${url}/api/v2.0/projects
```
创建项目
```shell
sh create_project_harbor.sh
```

推送镜像到私有镜像仓库
```shell
cd kubesphere-all-v3.0.0-offline-linux-amd64/kubesphere-images-v3.0.0
sh push-images.sh 192.168.93.9
```

3、部署kubesphere容器平台

```shell
helm repo add test https://charts.kubesphere.io/test
helm pull test/ks-installer
tar -zxvf ks-installer-0.2.1.tgz

helm install kubesphere \
  --namespace=kubesphere-system \
  --create-namespace \
  --set image.repository=192.168.93.9/kubesphere/ks-installer \
  --set image.tag=v3.0.0 \
  --set persistence.storageClass=longhorn \
  --set .registry=192.168.93.9 \
  ./ks-installer
```


## 清理kubesphere集群
参考：
[https://kubesphere.com.cn/en/docs/installing-on-kubernetes/uninstalling/uninstalling-kubesphere-from-k8s/](https://kubesphere.com.cn/en/docs/installing-on-kubernetes/uninstalling/uninstalling-kubesphere-from-k8s/)

```shell
wget https://raw.githubusercontent.com/kubesphere/ks-installer/master/scripts/kubesphere-delete.sh
sh kubesphere-delete.sh
```