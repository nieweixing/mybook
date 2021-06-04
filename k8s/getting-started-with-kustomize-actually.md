kustomize是kubernetes原生的配置管理，以无模板方式来定制应用的配置。kustomize使用k8s原生概念帮助创建并复用资源配置(YAML)，允许用户以一个应用描述文件（YAML 文件）为基础（Base YAML），然后通过Overlay的方式生成最终部署应用所需的描述文件。

这里简单了解下几个概念

* overlay

overlay 是一个 kustomization, 它修改(并因此依赖于)另外一个kustomization. overlay中的kustomization指的是一些其它的kustomization, 称为其 base. 没有 base, overlay 无法使用，并且一个 overlay 可以用作 另一个 overlay 的 base(基础)。简而言之，overlay 声明了与 base 之间的差异。通过 overlay 来维护基于 base 的不同 variants(变体)，例如开发、QA 和生产环境的不同variants，其实overlay就是不同版本的工作空间，依赖于base工作空间。

* variant

variant 是在集群中将 overlay 应用于 base 的结果。例如开发和生产环境都修改了一些共同 base 以创建不同的 variant。这些 variant 使用相同的总体资源，并与简单的方式变化，例如 deployment 的副本数、ConfigMap使用的数据源等。简而言之，variant 是含有同一组 base 的不同 kustomization，其实variant就是某一个版本环境的所有资源文件。

* resource

在kustomize的上下文中，resource 是描述 k8s API 对象的 YAML 或 JSON 文件的相对路径。即是指向一个声明了 kubernetes API对象的YAML文件

* patch

修改文件的一般说明。文件路径，指向一个声明了 kubernetes API patch 的 YAML 文件


# kustomize安装

```
curl -s "https://raw.githubusercontent.com/\
kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
```

安装成功后，执行kustomize可以查看帮助指导

```
[root@VM-0-13-centos mesh]# kustomize

Manages declarative configuration of Kubernetes.
See https://sigs.k8s.io/kustomize

Usage:
  kustomize [command]

Available Commands:
  build                     Print configuration per contents of kustomization.yaml
  cfg                       Commands for reading and writing configuration.
  completion                Generate shell completion script
  create                    Create a new kustomization in the current directory
  edit                      Edits a kustomization file
  fn                        Commands for running functions against configuration.
  help                      Help about any command
  version                   Prints the kustomize version

Flags:
  -h, --help          help for kustomize
      --stack-trace   print a stack-trace on error

Additional help topics:
  kustomize docs-fn                   [Alpha] Documentation for developing and invoking Configuration Functions.
  kustomize docs-fn-spec              [Alpha] Documentation for Configuration Functions Specification.
  kustomize docs-io-annotations       [Alpha] Documentation for annotations used by io.
  kustomize docs-merge                [Alpha] Documentation for merging Resources (2-way merge).
  kustomize docs-merge3               [Alpha] Documentation for merging Resources (3-way merge).
  kustomize tutorials-command-basics  [Alpha] Tutorials for using basic config commands.
  kustomize tutorials-function-basics [Alpha] Tutorials for using functions.

Use "kustomize [command] --help" for more information about a command.
```

# kustomize部署helloword

kustomize的demo示例可以参考链接<https://github.com/kubernetes-sigs/kustomize/tree/master/examples>，下面我们以helloworld为例进行示范下

## 创建base


首先我们创建一个一个helloworld的工作空间，在/tmp下创建一个临时目录

```
DEMO_HOME=$(mktemp -d)
```

如果我们需要用到overlay，则需要创建base工作空间，让集群的资源放在base下

```
BASE=$DEMO_HOME/base
mkdir -p $BASE

curl -s -o "$BASE/#1.yaml" "https://raw.githubusercontent.com\
/kubernetes-sigs/kustomize\
/master/examples/helloWorld\
/{configMap,deployment,kustomization,service}.yaml"
```

这样我们就将基础的yaml文件放到了base下

```
[root@VM-0-13-centos base]# tree $DEMO_HOME
/tmp/tmp.w5Ic40K11n
└── base
    ├── configMap.yaml
    ├── deployment.yaml
    ├── kustomization.yaml
    └── service.yaml
```

如果你想部署这些资源，可以用kubectl命令部署

```
kubectl apply -f $DEMO_HOME/base
```

我们可以预览下base的资源，会将base下的yaml内容打印在标准输出

```
[root@VM-0-13-centos base]# kustomize build $BASE
apiVersion: v1
data:
  altGreeting: Good Morning!
  enableRisky: "false"
kind: ConfigMap
metadata:
  labels:
    app: hello
  name: the-map
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello
  name: the-service
spec:
  ports:
  - port: 8666
    protocol: TCP
    targetPort: 8080
  selector:
    app: hello
    deployment: hello
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: hello
  name: the-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
      deployment: hello
  template:
    metadata:
      labels:
        app: hello
        deployment: hello
    spec:
      containers:
      - command:
        - /hello
        - --port=8080
        - --enableRiskyFeature=$(ENABLE_RISKY)
        env:
        - name: ALT_GREETING
          valueFrom:
            configMapKeyRef:
              key: altGreeting
              name: the-map
        - name: ENABLE_RISKY
          valueFrom:
            configMapKeyRef:
              key: enableRisky
              name: the-map
        image: monopole/hello:1
        name: the-container
        ports:
        - containerPort: 8080
```

当然我们也可以订制base下的，下面我们订制下app的label

```
[root@VM-0-13-centos base]# sed -i.bak 's/app: hello/app: my-hello/' \
>     $BASE/kustomization.yaml
[root@VM-0-13-centos base]# ll
total 20
-rw-r--r-- 1 root root 117 Jun  4 12:10 configMap.yaml
-rw-r--r-- 1 root root 750 Jun  4 12:10 deployment.yaml
-rw-r--r-- 1 root root 266 Jun  4 12:37 kustomization.yaml
-rw-r--r-- 1 root root 263 Jun  4 12:10 kustomization.yaml.bak
-rw-r--r-- 1 root root 183 Jun  4 12:10 service.yaml
[root@VM-0-13-centos base]# kustomize build $BASE | grep -C 3 app:
kind: ConfigMap
metadata:
  labels:
    app: my-hello
  name: the-map
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: my-hello
  name: the-service
spec:
  ports:
--
    protocol: TCP
    targetPort: 8080
  selector:
    app: my-hello
    deployment: hello
  type: LoadBalancer
---
--
kind: Deployment
metadata:
  labels:
    app: my-hello
  name: the-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-hello
      deployment: hello
  template:
    metadata:
      labels:
        app: my-hello
        deployment: hello
    spec:
      containers:
```

下面我们来部署多个Overlays来对应多个helloword订制版本

```
OVERLAYS=$DEMO_HOME/overlays
mkdir -p $OVERLAYS/staging
mkdir -p $OVERLAYS/production
```

## 创建staging Overlays

在staging目录中创建一个kustomization 文件，用来定义一个新的名称前缀和一些不同的 labels 。

```
cat <<'EOF' >$OVERLAYS/staging/kustomization.yaml
namePrefix: staging-
commonLabels:
  variant: staging
  org: acmeCorporation
commonAnnotations:
  note: Hello, I am staging!
resources:
- ../../base
patchesStrategicMerge:
- map.yaml
EOF
```

新增一个自定义的 configMap 将问候消息从 Good Morning! 改为 Have a pineapple! 。

同时，将 risky 标记设置为 true 。

```
cat <<EOF >$OVERLAYS/staging/map.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: the-map
data:
  altGreeting: "Have a pineapple!"
  enableRisky: "true"
EOF
```

## 创建production Overlays

在 production 目录中创建一个 kustomization 文件，用来定义一个新的名称前缀和 labels 。

```
cat <<EOF >$OVERLAYS/production/kustomization.yaml
namePrefix: production-
commonLabels:
  variant: production
  org: acmeCorporation
commonAnnotations:
  note: Hello, I am production!
resources:
- ../../base
patchesStrategicMerge:
- deployment.yaml
EOF
```


Production Patch，因为生产环境需要处理更多的流量，新建一个production patch来增加副本数。

```
cat <<EOF >$OVERLAYS/production/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: the-deployment
spec:
  replicas: 5
EOF
```

## 比较overlays

DEMO_HOME 现在包含：

* base目录：对拉取到的源配置进行了简单定制

* overlays目录：包含在集群中创建不同 staging 和 production variants 的 kustomizations 和 patches 。

查看目录结构和差异：

```
[root@VM-0-13-centos base]# tree $DEMO_HOME
/tmp/tmp.w5Ic40K11n
├── base
│   ├── configMap.yaml
│   ├── deployment.yaml
│   ├── kustomization.yaml
│   ├── kustomization.yaml.bak
│   └── service.yaml
└── overlays
    ├── production
    │   ├── deployment.yaml
    │   └── kustomization.yaml
    └── staging
        ├── kustomization.yaml
        └── map.yaml
```

直接比较 staging 和 production 输出的不同：

```
[root@VM-0-13-centos base]# diff \
>   <(kustomize build $OVERLAYS/staging) \
>   <(kustomize build $OVERLAYS/production) |\
>   more
3,4c3,4
<   altGreeting: Have a pineapple!
<   enableRisky: "true"
---
>   altGreeting: Good Morning!
>   enableRisky: "false"
8c8
<     note: Hello, I am staging!
---
>     note: Hello, I am production!
12,13c12,13
<     variant: staging
<   name: staging-the-map
---
>     variant: production
>   name: production-the-map
19c19
<     note: Hello, I am staging!
---
>     note: Hello, I am production!
23,24c23,24
<     variant: staging
<   name: staging-the-service
---
>     variant: production
>   name: production-the-service
34c34
<     variant: staging
---
>     variant: production
41c41
<     note: Hello, I am staging!
---
>     note: Hello, I am production!
45,46c45,46
<     variant: staging
<   name: staging-the-deployment
---
>     variant: production
>   name: production-the-deployment
48c48
<   replicas: 3
---
>   replicas: 5
54c54
<       variant: staging
---
>       variant: production
58c58
<         note: Hello, I am staging!
---
>         note: Hello, I am production!
63c63
<         variant: staging
---
>         variant: production
75c75
<         
```

## 部署不同的overlys

输出不同 overlys 的配置：

```
[root@VM-0-13-centos base]# kustomize build $OVERLAYS/staging
apiVersion: v1
data:
  altGreeting: Have a pineapple!
  enableRisky: "true"
kind: ConfigMap
metadata:
  annotations:
    note: Hello, I am staging!
  labels:
    app: my-hello
    org: acmeCorporation
    variant: staging
  name: staging-the-map
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    note: Hello, I am staging!
  labels:
    app: my-hello
    org: acmeCorporation
    variant: staging
  name: staging-the-service
spec:
  ports:
  - port: 8666
    protocol: TCP
    targetPort: 8080
  selector:
    app: my-hello
    deployment: hello
    org: acmeCorporation
    variant: staging
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    note: Hello, I am staging!
  labels:
    app: my-hello
    org: acmeCorporation
    variant: staging
  name: staging-the-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-hello
      deployment: hello
      org: acmeCorporation
      variant: staging
  template:
    metadata:
      annotations:
        note: Hello, I am staging!
      labels:
        app: my-hello
        deployment: hello
        org: acmeCorporation
        variant: staging
    spec:
      containers:
      - command:
        - /hello
        - --port=8080
        - --enableRiskyFeature=$(ENABLE_RISKY)
        env:
        - name: ALT_GREETING
          valueFrom:
            configMapKeyRef:
              key: altGreeting
              name: staging-the-map
        - name: ENABLE_RISKY
          valueFrom:
            configMapKeyRef:
              key: enableRisky
              name: staging-the-map
        image: monopole/hello:1
        name: the-container
        ports:
        - containerPort: 8080
```

```
[root@VM-0-13-centos base]# kustomize build $OVERLAYS/production
apiVersion: v1
data:
  altGreeting: Good Morning!
  enableRisky: "false"
kind: ConfigMap
metadata:
  annotations:
    note: Hello, I am production!
  labels:
    app: my-hello
    org: acmeCorporation
    variant: production
  name: production-the-map
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    note: Hello, I am production!
  labels:
    app: my-hello
    org: acmeCorporation
    variant: production
  name: production-the-service
spec:
  ports:
  - port: 8666
    protocol: TCP
    targetPort: 8080
  selector:
    app: my-hello
    deployment: hello
    org: acmeCorporation
    variant: production
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    note: Hello, I am production!
  labels:
    app: my-hello
    org: acmeCorporation
    variant: production
  name: production-the-deployment
spec:
  replicas: 5
  selector:
    matchLabels:
      app: my-hello
      deployment: hello
      org: acmeCorporation
      variant: production
  template:
    metadata:
      annotations:
        note: Hello, I am production!
      labels:
        app: my-hello
        deployment: hello
        org: acmeCorporation
        variant: production
    spec:
      containers:
      - command:
        - /hello
        - --port=8080
        - --enableRiskyFeature=$(ENABLE_RISKY)
        env:
        - name: ALT_GREETING
          valueFrom:
            configMapKeyRef:
              key: altGreeting
              name: production-the-map
        - name: ENABLE_RISKY
          valueFrom:
            configMapKeyRef:
              key: enableRisky
              name: production-the-map
        image: monopole/hello:1
        name: the-container
        ports:
        - containerPort: 8080
```


将上述命令传递给kubectl进行部署

```
kustomize build $OVERLAYS/staging |\
    kubectl apply -f -
kustomize build $OVERLAYS/production |\
   kubectl apply -f -
```

也可直接使用kubectl部署，但是需要注意的是**kubectl版本需要在v1.14.0以上**

```
kubectl apply -k $OVERLAYS/staging
kubectl apply -k $OVERLAYS/production
```