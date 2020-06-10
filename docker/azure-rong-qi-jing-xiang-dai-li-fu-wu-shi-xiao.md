# Azure容器镜像代理服务失效

## Azure容器镜像代理服务失效

### 解决方案

前段时间 Azure 中国提供的容器镜像代理服务已经不开放了，只给 Azure 云使用。最近使用Azure的加速地址都显示404。像以下拉取镜像的方式，现在不能用了：

```text
docker pull dockerhub.azk8s.cn/library/<imagename>:<version> 
#例子
docker pull dockerhub.azk8s.cn/library/centos
```

建议使用阿里云的比较稳定，这边另外推荐一个容器镜像代理服务，西北农林科技大学的dockerhub 反向代理。还比较稳定，当然首选还是阿里云。 下面介绍下西北农林科技大学的反向代理： 修改 /etc/docker/daemon.json ，加入：

```text
{
  "registry-mirrors": ["https://dockerhub.mirrors.nwafu.edu.cn/"]
}
```

重启docker服务：

```text
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 笔者使用方式

对于daemon.json中的registry-mirros字段后面的值，是个数组，可以加入多个反向代理地址，docker会按顺序去拉取镜像，第一个代理地址拉不到就使用第二个地址，要是这边配置的地址都不能拉取镜像，dockder会去dockerhub官网拉取。 所以我的daemon.json是这样配置的：

```text
{
  "registry-mirrors": [""https://cduvuqsh.mirror.aliyuncs.com","https://dockerhub.mirrors.nwafu.edu.cn/"]
}
```

以阿里云为主，西北农林科技大学的为辅，做个备用。

更多云计算方面技术文章，请关注"南君手记"公众号。欢迎批评指正。

