---
description: tekton的原生仪表盘
---

# Tekton Dashboard

## Tekton Dashboard

先给大家看下部署完之后的仪表盘页面：

![dashboard-ui](https://github.com/fishingfly/pictures_for_markdown/blob/master/tekton/dashboard-ui.png?raw=true) Tekton仪表板是Tekton Pipelines基于Web的通用UI。它允许用户管理和查看Tekton PipelineRun和TaskRun,以及在tekton中创建，执行和完成过程中涉及的资源。它还允许按标签过滤PipelineRun和TaskRun。本篇将简单介绍dashboard及其安装，当然也会将下该工具在开发中的使用。

### 安装dashboard

1、所需环境 k8s 1.15以上版本的集群，我用的1.17，公众号（南君手记）中有部署k8s集群的傻瓜教程。 装好了tekton/pipeline 0.11.3了。 使用的dashboard版本是v0.6.1 2、开始安装 官方的第一步是：

```text
kubectl apply --filename https://github.com/tektoncd/dashboard/releases/download/v0.6.1/tekton-dashboard-release.yaml
```

但是里面的dashboard镜像是谷歌的，没翻墙的话拉不下来，建议先把文件下载下来，然后改镜像为：

```text
registry.cn-hangzhou.aliyuncs.com/launcher/tektoncd-dashboard:latest
```

然后因为我这边需要前端访问，最好将里面service改成nortport。以免麻烦，我直接把yaml写在下面，你直接运行kubectl apply -f 命令就行

```text
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: extensions.dashboard.tekton.dev
spec:
  group: dashboard.tekton.dev
  names:
    categories:
    - tekton
    - tekton-dashboard
    kind: Extension
    plural: extensions
  scope: Namespaced
  subresources:
    status: {}
  version: v1alpha1
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app: tekton-dashboard
  name: tekton-dashboard
  namespace: tekton-pipelines
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: tekton-dashboard-minimal
  namespace: tekton-pipelines
rules:
- apiGroups:
  - ""
  resources:
  - serviceaccounts
  verbs:
  - update
  - patch
- apiGroups:
  - ""
  resources:
  - pods
  - services
  verbs:
  - create
  - update
  - delete
  - patch
- apiGroups:
  - ""
  resources:
  - secrets
  - configmaps
  verbs:
  - create
  - update
  - delete
- apiGroups:
  - extensions
  - apps
  resources:
  - deployments
  verbs:
  - create
  - update
  - delete
  - patch
- apiGroups:
  - tekton.dev
  resources:
  - tasks
  - clustertasks
  - taskruns
  - pipelines
  - pipelineruns
  - pipelineresources
  - conditions
  verbs:
  - create
  - update
  - delete
  - patch
- apiGroups:
  - tekton.dev
  resources:
  - taskruns/finalizers
  - pipelineruns/finalizers
  verbs:
  - create
  - update
  - delete
  - patch
- apiGroups:
  - tekton.dev
  resources:
  - tasks/status
  - clustertasks/status
  - taskruns/status
  - pipelines/status
  - pipelineruns/status
  verbs:
  - create
  - update
  - delete
  - patch
- apiGroups:
  - dashboard.tekton.dev
  resources:
  - extensions
  verbs:
  - create
  - update
  - delete
  - patch
- apiGroups:
  - triggers.tekton.dev
  resources:
  - clustertriggerbindings
  - eventlisteners
  - triggerbindings
  - triggertemplates
  verbs:
  - create
  - update
  - delete
  - patch
  - add
- apiGroups:
  - apiextensions.k8s.io
  resources:
  - customresourcedefinitions
  verbs:
  - get
  - list
- apiGroups:
  - security.openshift.io
  resources:
  - securitycontextconstraints
  verbs:
  - use
- apiGroups:
  - route.openshift.io
  resources:
  - routes
  verbs:
  - get
  - list
- apiGroups:
  - extensions
  - apps
  resources:
  - ingresses
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - serviceaccounts
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - pods
  - services
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - pods/log
  - namespaces
  - events
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - secrets
  - configmaps
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - extensions
  - apps
  resources:
  - deployments
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - tekton.dev
  resources:
  - tasks
  - clustertasks
  - taskruns
  - pipelines
  - pipelineruns
  - pipelineresources
  - conditions
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - tekton.dev
  resources:
  - taskruns/finalizers
  - pipelineruns/finalizers
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - tekton.dev
  resources:
  - tasks/status
  - clustertasks/status
  - taskruns/status
  - pipelines/status
  - pipelineruns/status
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - dashboard.tekton.dev
  resources:
  - extensions
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - triggers.tekton.dev
  resources:
  - clustertriggerbindings
  - eventlisteners
  - triggerbindings
  - triggertemplates
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tekton-dashboard-minimal
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: tekton-dashboard-minimal
subjects:
- kind: ServiceAccount
  name: tekton-dashboard
  namespace: tekton-pipelines
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: tekton-dashboard
    dashboard.tekton.dev/release: v0.6.1
    version: v0.6.1
  name: tekton-dashboard
  namespace: tekton-pipelines
spec:
  type: NodePort #不需要暴露服务的话，这行删去
  ports:
  - name: http
    port: 9097
    protocol: TCP
    targetPort: 9097
    nodePort: 30097 #不需要暴露服务的话，这行删去
  selector:
    app: tekton-dashboard
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: tekton-dashboard
    dashboard.tekton.dev/release: v0.6.1
    version: v0.6.1
  name: tekton-dashboard
  namespace: tekton-pipelines
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tekton-dashboard
  template:
    metadata:
      labels:
        app: tekton-dashboard
      name: tekton-dashboard
    spec:
      containers:
      - env:
        - name: PORT
          value: "9097"
        - name: READ_ONLY
          value: "false"
        - name: WEB_RESOURCES_DIR
          value: /var/run/ko/web
        - name: PIPELINE_RUN_SERVICE_ACCOUNT
          value: ""
        - name: TRIGGERS_NAMESPACE
          value: tekton-pipelines
        - name: PIPELINE_NAMESPACE
          value: tekton-pipelines
        - name: INSTALLED_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        image: seandragon/tekton-dashboard:v0.6.1
        livenessProbe:
          httpGet:
            path: /health
            port: 9097
        name: tekton-dashboard
        ports:
        - containerPort: 9097
        readinessProbe:
          httpGet:
            path: /readiness
            port: 9097
      serviceAccountName: tekton-dashboard

---
```

运行完后，去看下pod运行，要都是running的就算成功了。

```text
kubectl get po -n tekton-pipelines


NAME                                           READY   STATUS    RESTARTS   AGE
tekton-dashboard-6cc7c565d9-zhd8x              1/1     Running   0          75m
tekton-pipelines-controller-7cf5f4f87f-xk72r   1/1     Running   0          5h27m
tekton-pipelines-webhook-7bc6fc7d4b-j92lc      1/1     Running   0          5h27m
```

浏览器访问[http://masterIp:30097/](http://masterIp:30097/) 就有页面弹出来的。安装到此结束

### 介绍

仪表盘主要功能就是为taskRun和pipelineRuns提供视图管理界面，同时，你可以看到与taskRun和pipelineRuns相关联的资源的创建和执行过程。你可以看到所有命名空间下的资源。在我看来最主要的一个用途就是可以看到task的执行结果，哪个步骤报了什么错都能一目了然。 下面我运行一个简单的例子，看一下效果； 下面跑个taskrun看下:\(这个taskrun只是做一些简单的打印任务\)

```text
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  name: step-scripti
spec:
  taskSpec:
    params:
    - name: PARAM
      default: param-value

    steps:
    - name: noshebang
      image: ubuntu
      script: echo "no shebang"
    - name: bash
      image: ubuntu
      env:
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

运行结果如下：

![dashboard-1](https://github.com/fishingfly/pictures_for_markdown/blob/master/tekton/dashboard_1.png?raw=true)

![dashboard-2](https://github.com/fishingfly/pictures_for_markdown/blob/master/tekton/dash_2.PNG?raw=true)

### 开发使用

对于dashboard的使用场景，我觉得除了展示一些tekton中的资源外，还可以用于获取任务的状态、日志信息，有些私有云中搭建自己的流水线，也是需要执行过程的信息输出的，和dashboard是一样的，这边人家已经封装好数据了使用起来会比较方便。

