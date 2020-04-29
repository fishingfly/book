# standard\_init\_linux.go:190: exec user process caused "no such file or directory

今天打golang工程的镜像遇到一个问题，就是我在centos的主机上编译golang项目，然后使用Dockerfile构建镜像，用docker运行镜像时，日志一直报如下错误，且一直重启：

```text
standard_init_linux.go:190: exec user process caused "no such file or directory"
```

Dockerfile如下：

```text
FROM alpine:latest
COPY test-latest-linux-amd64 /usr/local/bin/test
RUN echo -e "https://mirrors.ustc.edu.cn/alpine/latest-stable/main\nhttps://mirrors.ustc.edu.cn/alpine/latest-stable/community" > /etc/apk/repositories && \
    apk update &&\
    apk --no-cache add tzdata && \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" >  /etc/timezone
ENTRYPOINT ["test"]
```

Dockerfile没啥问题。编译命令是go build -o test-latest-linux-amd64 main.go

网上一搜，发现centos下编译生成的二进制文件，拿进alpine下运行时有问题的。

编译命令加入如下参数：-tags netgo

```text
go build -tags netgo -o test-latest-linux-amd64 main.go
```

就可以了 具体原理不明。我建议以后编译golang项目统一在docker 容器中编译。

参考文章

