# k8s CRD（一）quick start



## k8s CRD（一）quick start

### 1.安装CRD所需环境

1、去找一个k8s集群 2、找一台master节点（可以用kuberctl命令的即可），安装golang,配置好Gopath

### 2.开始安装

访问[kubebuilder指导手册](https://book.kubebuilder.io/quick-start.html)

```text
os=$(go env GOOS)
arch=$(go env GOARCH)
curl -L https://go.kubebuilder.io/dl/2.3.0/${os}/${arch} | tar -xz -C /tmp/ #下载包并放到/tmp下
sudo mv /tmp/kubebuilder_2.3.0_${os}_${arch} /usr/local/kubebuilder #拷贝到/usr/local/kubebuilder下
export PATH=$PATH:/usr/local/kubebuilder/bin ##最好放在/etc/profile。然后source /etc/profile
```

这边可能下载不下来包，你可以访问这个网址[https://go.kubebuilder.io/dl/2.3.0。自己下载就行](https://go.kubebuilder.io/dl/2.3.0。自己下载就行)

上面完成后，试一下kubebuilder --help会出现指导就算安装完成 接下来创建一个简单的CRD

```text
mkdir $GOPATH/src/example
cd $GOPATH/src/example
export GO111MODULE=on 
export GOPROXY=https://goproxy.io
kubebuilder init --domain my.domain ##这边会拉取很多包
kubebuilder create api --group webapp --version v1 --kind Guestbook
要是遇到：/bin/sh: kustomize: command not found
那就：go get github.com/kubernetes-sigs/kustomize
make install
make run
```

要是上述步骤没抱什么其他错的话，那就说明你的CRD controller已经部署在集群了，运行以下命令查看：

```text
kubectl get crd
会看到如下：
guestbooks.webapp.my.domain            2020-03-10T04:23:03Z
```

可以在源码目录下看到config文件下有个sample文件夹，里面有例子：

```text
kubectl apply -f config/samples/
然后
kubectl get guestbook
返回
NAME               AGE
guestbook-sample   29s
```

那就完成了一个简单的CRD demo了

### 3.构建CRD镜像并部署进k8s

构建镜像并推送至你的镜像仓库

```text
make docker-build docker-push IMG=<some-registry>/<project-name>:tag
```

指定镜像将controller部署进你的集群

```text
make deploy IMG=<some-registry>/<project-name>:tag
```

### 4.卸载CRD

```text
make uninstall
```

