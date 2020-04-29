# tekton简单介绍及安装



## tekton简单介绍及安装

今天开始，我将逐步上新tekton的使用教程，每篇只需5分钟的阅读时间。如有写的不好的地方，欢迎评论指正

#### 1.1 背景介绍

Tekton是一个谷歌开源的kubernetes原生CI/CD系统，功能强大且灵活，开源社区也正在快速的迭代和发展壮大。其实Tekton的前身是Knative的build-pipeline项目，从名字可以看出这个项目是为了给build模块增加pipeline的功能，但是大家发现随着不同的功能加入到Knative build模块中，build模块越来越变得像一个通用的CI/CD系统，这已经脱离了Knative build设计的初衷，于是，索性将build-pipeline剥离出Knative，摇身一变成为Tekton，而Tekton也从此致力于提供全功能、标准化的原生kubernetesCI/CD解决方案。Tekton虽然还是一个挺新的项目，但是已经成为 Continuous Delivery Foundation \(CDF\) 的四个初始项目之一，另外三个则是大名鼎鼎的Jenkins、Jenkins X、Spinnaker，实际上Tekton还可以作为插件集成到JenkinsX中。所以，如果你觉得Jenkins太重，没必要用Spinnaker这种专注于多云平台的CD，为了避免和Gitlab耦合不想用gitlab-ci，那么Tekton值得一试。Tekton的特点是kubernetes原生，什么是kubernetes原生呢？简单的理解，就是all in kubernetes，所以用容器化的方式构建容器镜像是必然，另外，基于kubernetes CRD定义的pipeline流水线也是Tekton最重要的特征。那Tekton都提供了哪些CRD呢？ 1. Task 2. TaskRun 3. Pipeline 4. PipelineRun 5. PipelineResource

Tekton Pipelines 是一个开源项目，可在 Kubernetes 集群中配置和运行持续集成和持续交付（也称为 CI/CD）流水线。在本教程中，Tekton 以自定义资源的形式提供了一组 Kubernetes 扩展，用于定义流水线。 Tekton Pipelines项目提供了Kubernetes样式的资源来声明CI / CD样式的管道。Tekton pipelines是云原生的:

* 运行在k8s上；
* Have Kubernetes clusters as a first class type（不知道什么意思）
* Use containers as their building blocks（使用容器作为什么）

Tekton Pipelines是解耦的:

* 一个Pipeline可以被部署进任何k8s集群
* 组成pipeline的task可以轻松的独立运行
* git repos之类的资源可以在运行之间轻松交换

  **1.2 安装tekton/pipelines**

  先看下安装环境:kubernetes版本要在1.15之上。

  安装tekton pipeline组件到已有的k8s集群中:

```text
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

这是官方教程的安装，但是release.yaml中需要的镜像是从谷歌云拉取的，国内的环境可能拉不到镜像，建议先把release.yaml下载下来，然后编辑下镜像前缀：

```text
wget https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

然后修改以下镜像前缀：用下面的地址手动拉取镜像，我用了个简单的方法，讲镜像地址为gcr.io/tekton-releases/github.com/tektoncd/pipeline/cmd/controller:v0.10.1替换为gcr.azk8s.cn/tekton-releases/github.com/tektoncd/pipeline/cmd/controller:v0.10.1。也就是讲gcr.io改为gcr.azk8s.cn就可以了，直接走微软云的加速器拉取镜像。你需要在release.yaml文件中改掉下面这么多镜像，或者你全文搜索Image。镜像前缀域名改成微软云加速器地址就行。 需要修改的镜像如下，不要漏了：

```text
 image: gcr.azk8s.cn/tekton-releases/github.com/tektoncd/pipeline/cmd/controller:v0.10.1@sha256:f4409692f43c4781d949847e661109c69c1a9acb8453cd1219b8341d642cb756
        args: ["-kubeconfig-writer-image", "gcr.azk8s.cn/tekton-releases/github.com/tektoncd/pipeline/cmd/kubeconfigwriter:v0.10.1@sha256:206c4e5de37d13c34f9538f87096db16433aadba264f24e9995cbb6b66fb67de",
          "-creds-image", "gcr.azk8s.cn/tekton-releases/github.com/tektoncd/pipeline/cmd/creds-init:v0.10.1@sha256:959b0d9a2d43d35e15a85460cc86567d308f467ee8ec16dbd9b32f51ce75d582",
          "-git-image", "gcr.azk8s.cn/tekton-releases/github.com/tektoncd/pipeline/cmd/git-init:v0.10.1@sha256:18ffa2bfc14b1fa6d39f62271beacf6bbc38e7cd2e255184dec477c2936270bc",
          "-nop-image", "tianon/true", "-shell-image", "busybox", "-gsutil-image",
          "google/cloud-sdk", "-entrypoint-image", "gcr.azk8s.cn/tekton-releases/github.com/tektoncd/pipeline/cmd/entrypoint:v0.10.1@sha256:cf0e81477c45dca0df6253e3239f6f0603700641292bf207503a7b267dc4c916",
          "-imagedigest-exporter-image", "gcr.azk8s.cn/tekton-releases/github.com/tektoncd/pipeline/cmd/imagedigestexporter:v0.10.1@sha256:1cf3f27f3ff7c73782d8a65853e8fc7f0d4aafc6443893e0150bdbe614a9169d",
          "-pr-image", "gcr.azk8s.cn/tekton-releases/github.com/tektoncd/pipeline/cmd/pullrequest-init:v0.10.1@sha256:3783254c379b286dd0987674160d3e19a95be1ccf0985788d6dcc0f159199095",
          "-build-gcs-fetcher-image", "gcr.azk8s.cn/tekton-releases/github.com/tektoncd/pipeline/vendor/github.com/googlecloudplatform/cloud-builders/gcs-fetcher/cmd/gcs-fetcher:v0.10.1@sha256:70f8d32a572496169d451130541541cbc99434932fd28beea486189af8a2995a"]

          image: gcr.azk8s.cn/tekton-releases/github.com/tektoncd/pipeline/cmd/webhook:v0.10.1@sha256:e0a277b69a742ff0e49767d11f9a1325fcea7fb8bbbf2572af9d49116cbb2385
```

然后你在本地运行

```text
kubectl apply -f release.yaml
```

运行以下命令来查看部署pipeline是否成功，要是pod都是running，说明部署成功了

```text
kubectl get pods --namespace tekton-pipelines
```

