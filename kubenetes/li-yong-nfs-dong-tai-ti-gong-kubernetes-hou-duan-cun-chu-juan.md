---
description: 本文将介绍使用nfs-client-provisioner这个应用，利用NFS Server给Kubernetes作为持久存储的后端，并且动态提供PV。
---

# 利用NFS动态提供Kubernetes后端存储卷

## 利用NFS动态提供Kubernetes后端存储卷

本文将介绍使用nfs-client-provisioner这个应用，利用NFS Server给Kubernetes作为持久存储的后端，并且动态提供PV。

### 安装nfs

我这边安装的是一台nfs服务器，比较简单。其他节点安装nfs-utils就够了。

```text
yum -y install nfs-utils rpcbind
find /etc/ -name '*rpcbind.socket*'   
vim /etc/systemd/system/sockets.target.wants/rpcbind.socket #上一条文件结果
#查看是否和下面一样
[Unit]
Description=RPCbind Server Activation Socket
[Socket]
ListenStream=/var/run/rpcbind.sock
# RPC netconfig can't handle ipv6/ipv4 dual sockets
BindIPv6Only=ipv6-only
ListenStream=0.0.0.0:111
ListenDatagram=0.0.0.0:111
#ListenStream=[::]:111
#ListenDatagram=[::]:111
[Install]
WantedBy=sockets.target
```

配置服务开机运行：

```text
systemctl enable rpcbind.service &&systemctl start rpcbind.service
systemctl enable nfs.service &&systemctl start nfs.service
```

配置共享目录:

```text
#创建共享目录，目录自己定
mkdir -p /usr/share/k8s
#按需设定目录权限
chmod -R 666 /usr/share/k8s
#更改共享设置
vi /etc/exports
/usr/share/k8s *(insecure,rw,no_root_squash) 
systemctl restart nfs
```

测试Nfs服务是否正常： 选择另外一台主机进行测试，另一台主机也安装了nfs-utils，没安装就执行：

```text
#安装nfs-utils用于测试
yum -y install nfs-utils rpcbind
#查看Nfs主机上的共享
showmount -e 192.168.161.180
Export list for 192.168.161.180:
/usr/share/k8s *

#尝试挂载
mount -t nfs 192.168.161.180:/usr/share/k8s（共享目录） /usr/share/k8s(本地目录)

#查看是否挂载成功
df -Th
```

### k8s中使用nfs做存储盘

1、配置rbac：

```text
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-provisioner
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: nfs-provisioner-runner
rules:
   -  apiGroups: [""]
      resources: ["persistentvolumes"]
      verbs: ["get", "list", "watch", "create", "delete"]
   -  apiGroups: [""]
      resources: ["persistentvolumeclaims"]
      verbs: ["get", "list", "watch", "update"]
   -  apiGroups: ["storage.k8s.io"]
      resources: ["storageclasses"]
      verbs: ["get", "list", "watch"]
   -  apiGroups: [""]
      resources: ["events"]
      verbs: ["watch", "create", "update", "patch"]
   -  apiGroups: [""]
      resources: ["services", "endpoints"]
      verbs: ["get","create","list", "watch","update"]
   -  apiGroups: ["extensions"]
      resources: ["podsecuritypolicies"]
      resourceNames: ["nfs-provisioner"]
      verbs: ["use"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-provisioner
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
```

2、storageClass

```text
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: stateful-nfs
provisioner: zy-test                  #这个要和nfs-client-provisioner的env环境变量中的PROVISIONER_NAME的value值对应。
reclaimPolicy: Retain               #指定回收策略为Retain（手动释放）
```

3、pvc

```text
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
spec:
  storageClassName: stateful-nfs              #定义存储类的名称，需与SC的名称对应
  accessModes:
    - ReadWriteMany                        #访问模式为RWM
  resources:
    requests:
      storage: 100Mi
```

4、测试deployment

```text
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
spec:
  replicas: 1                              #指定副本数量为1
  strategy:
    type: Recreate                      #指定策略类型为重置
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccount: nfs-provisioner            #指定rbac yanl文件中创建的认证用户账号
      containers:
        - name: nfs-client-provisioner
          image: registry.cn-hangzhou.aliyuncs.com/open-ali/nfs-client-provisioner     #使用的镜像
          volumeMounts:
            - name: nfs-client-root
              mountPath:  /persistentvolumes             #指定容器内挂载的目录
          env:
            - name: PROVISIONER_NAME           #容器内的变量用于指定提供存储的名称
              value: zy-test
            - name: NFS_SERVER                      #容器内的变量用于指定nfs服务的IP地址
              value: 192.168.161.180                #ip是nfs服务器地址
            - name: NFS_PATH                       #容器内的变量指定nfs服务器对应的目录
              value: /usr/share/k8s
      volumes:                                                #指定挂载到容器内的nfs的路径及IP
        - name: nfs-client-root
          nfs:
            server: 192.168.161.180
            path: /usr/share/k8s
```

看下如果有pv创建出来和pvc被绑定，当然po需要是running状态，那就是成功：

```text
[root@master1 nfs]# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM              STORAGECLASS   REASON   AGE
pvc-6d85fe39-b6b4-4c29-ade8-4aff4ce7fb4e   100Mi      RWX            Delete           Bound    default/test-pvc   stateful-nfs            26m
[root@master1 nfs]# kubectl get pvc
NAME       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
test-pvc   Bound    pvc-6d85fe39-b6b4-4c29-ade8-4aff4ce7fb4e   100Mi      RWX            stateful-nfs   61m
```

参考文章：

\[3\]:[https://blog.51cto.com/14157628/2470107](https://blog.51cto.com/14157628/2470107)

