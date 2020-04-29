# tekton之Task&TaskRun概念篇

## tekton之Task&TaskRun概念篇

### 1 Tasks

#### 1.1 task定义和简单使用

Tasks就是任务，任务是你希望在连续集成流程中运行的顺序步骤的集合。1个task在k8s集群中是以pod的形式运行，task中的每个step都是由各自的容器去完成step。举个例子：

```text
apiVersion: tekton.dev/v1alpha1
kind: Task
metadata:
  name: echo-hello-world
spec:
  steps:
    - name: echo
      image: ubuntu
      command:
        - echo
      args:
        - "hello world"
```

上面这个task中的steps后面就是各个step。这边只有一个step，该step的内容就是在ubuntu:latest容器中运行 echo "hello world"这个命令，你可以在在一个task中定义多个step，每个step还可以执行脚本，后面我会加一个执行脚本的例子。你光创建task资源，任务不会执行，需要使用taskRun才可以运行task。下面简单定义一个taskRun：

```text
apiVersion: tekton.dev/v1alpha1
kind: TaskRun
metadata:
  name: echo-hello-world-task-run
spec:
  taskRef:
    name: echo-hello-world
```

这两个yaml创建后，就可以跑任务了，查看运行的任务情况，使用tkn命令前需要先安装tekton CLI[地址](https://github.com/tektoncd/cli)， 这边简单介绍下linux下安装tekton cli：

```text
# Get the tar.xz
curl -LO https://github.com/tektoncd/cli/releases/download/v0.7.1/tkn_0.7.1_Linux_x86_64.tar.gz
# Extract tkn to your PATH (e.g. /usr/local/bin)
sudo tar xvzf tkn_0.7.1_Linux_x86_64.tar.gz -C /usr/local/bin/
```

然后运行tkn命令查看taskrun任务： `tkn taskrun describe echo-hello-world-task-run` 可以看下如下输出：

```text
Name:        echo-hello-world-task-run
Namespace:   default
Task Ref:    echo-hello-world

Status
STARTED         DURATION    STATUS
4 minutes ago   9 seconds   Succeeded

Input Resources
No resources

Output Resources
No resources

Params
No params

Steps
NAME
echo
```

当然对于想进一步获取task中每个step日志信息的用户可以去这个地址上看下 [log](https://github.com/tektoncd/pipeline/blob/master/docs/logs.md)。我这边参见下面的章节有专门讲述日志的。

task的step可用于执行脚本，如下所示：

```text
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  generateName: step-script-
spec:
  taskSpec:#直接定义task。而不引用task。我不建议这样使用，这样的话task无法复用。task和taskrun藕合在一块了
    params:
    - name: PARAM
      default: param-value

    steps:
    - name: noshebang
      image: ubuntu
      script: echo "no shebang"
    - name: bash
      image: ubuntu
      env:#环境变量
      - name: FOO
        value: foooooooo
      script: |
        #!/usr/bin/env bash
        set -euxo pipefail
        echo "Hello from Bash!"
        echo FOO is ${FOO}
        echo substring is ${FOO:2:4}
        for i in {1..10}; do
          echo line $i
        done
    - name: place-file
      image: ubuntu
      script: |
        #!/usr/bin/env bash
        echo "echo Hello from script file" > /workspace/hello
        chmod +x /workspace/hello
    - name: run-file
      image: ubuntu
      script: |
        #!/usr/bin/env bash
        /workspace/hello
    - name: contains-eof
      image: ubuntu
      script: |
        #!/usr/bin/env bash
        cat > file << EOF
        this file has some contents
        EOF
        cat file
    - name: node
      image: node
      script: |
        #!/usr/bin/env node
        console.log("Hello from Node!")
    - name: python
      image: python
      script: |
        #!/usr/bin/env python3
        print("Hello from Python!")
    - name: perl
      image: perl
      script: |
        #!/usr/bin/perl
        print "Hello from Perl!"
    # Test that param values are replaced.
    - name: params-applied
      image: python
      script: |
        #!/usr/bin/env python3
        v = '$(params.PARAM)'
        if v != 'param-value':
          print('Param values not applied')
          print('Got: ', v)
          exit(1)
    # Test that args are allowed and passed to the script as expected.
    - name: args-allowed
      image: ubuntu
      args: ['hello', 'world']
      script: |
        #!/usr/bin/env bash
        [[ $# == 2 ]]
        [[ $1 == "hello" ]]
        [[ $2 == "world" ]]
```

step中可执行脚本，可操作性大，可以在容器中执行你要的任务。

#### 1.2 task的输入和输出

在常用的场景中，task需要多个步骤（step）来处理输入和输出资源，例如task需要从Github上拉代码仓库或者构建docker镜像。PipelineResources就是被用来定义任务输入和输出的工件，有几种系统定义的资源类型可供使用，以下是两个常用资源的示例。 这是一个定义git的pipelineResource

```text
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: skaffold-git
spec:
  type: git
  params:
    - name: revision
      value: master #分支名
    - name: url
      value: https://github.com/fishingfly/golang-test.git #github仓库地址
```

这是定义docker image的pipelineResource：

```text
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: skaffold-image-leeroy-web
spec:
  type: image
  params:
    - name: url
      value: gcr.io/<use your project>/leeroy-web #这边是谷歌云的地址，最好改成你阿里云仓库地址或者也可以是你是私有仓库比地址（私有仓库的话需要额外配置，后面会有例子）
```

#### 1.3 task、pipelineResource和taskRun使用场景

这三者连用就可以实现的场景就是制作镜像并推送到仓库。代码源来自Github，可以是公有仓库代码也可以是私有仓库代码，拉下来之后根据代码是java还是golang进行构建编译，最后根据仓库中的Dockerfile进行镜像制作，制作完成可以推送到公有云容器镜像仓库或者你的私有仓库。这类场景是我用tekton的初衷。之后会分享出来，尽情期待。

### 2 TaskRun

#### 2.1 定义

使用TaskRun资源对象创建并运行群集上的进程以完成操作。task只是定义了一个任务模版，taskRun才真正代表了一次实际的运行，启动taskrun才可以运行task，当然你也可以自己手动创建一个taskRun，taskRun创建出来之后，就会自动触发task描述的构建任务。taskRun只有当task的所有Step都执行完成才会运行完。

#### 2.2 使用

在taskrun中引用task的方式:

```text
spec:
  taskRef:
    name: read-task
```

在taskrun中直接定义task的内嵌:

```text
spec:
  taskSpec:
    resources:
      inputs:
        - name: workspace
          type: git
    steps:
      - name: build-and-push
        image: gcr.io/kaniko-project/executor:v0.17.1
        # specifying DOCKER_CONFIG is required to allow kaniko to detect docker credential
        env:
          - name: "DOCKER_CONFIG"
            value: "/tekton/home/.docker/"
        command:
          - /kaniko/executor
        args:
          - --destination=gcr.io/my-project/gohelloworld
```

taskrun中设置参数：

```text
spec:
  params:
    - name: flags
      value: -someflag
```

提供resource: task需要inputresource和outputresouce，分别代表task的输入资源和输出资源，可以由已有的pipelineresource提供：

```text
spec:
  resources:
    inputs:
      - name: workspace
        resourceRef:
          name: java-git-resource
    outputs:
      - name: image
        resourceRef:
          name: my-app-image
```

当然你直接可以在taskrun中定义这些resource:

```text
spec:
  resources:
    inputs:
      - name: workspace
        resourceSpec:
          type: git
          params:
            - name: url
              value: https://github.com/pivotal-nader-ziada/gohelloworld
```

配置超时时间: default-timeout-minutes未设置的i情况下，默认60分钟，也就是说如果taskrun在60分钟内没有运行完成，就会自动kill taskrun，以免挂着占用资源，这个参数可以在config/config.yaml中修改，这个之后会有案例。 下面是task和taskrun联合使用的例子：

```text
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: mytask
  namespace: default
spec:
  steps:
    - name: writesomething
      image: ubuntu
      command: ["bash", "-c"]
      args: ["echo 'foo' > /my-cache/bar"]
      volumeMounts:
        - name: my-cache
          mountPath: /my-cache
---
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  name: mytaskrun
  namespace: default
spec:
  taskRef:
    name: mytask
  podTemplate:
    schedulerName: volcano
    securityContext: #安全策略
      runAsNonRoot: true #使用非root的用户去运行
    volumes:
    - name: my-cache
      persistentVolumeClaim:
        claimName: my-volume-claim
```

## 总结

本篇只介绍了task和taskrun的基本概念，属于教程类，后面我会根据实际需要写一个完整的例子，便于大家理解和实际使用。

