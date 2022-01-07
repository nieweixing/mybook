# istio sidecar注入

istio网格中， Pod都必须伴随一个Istio兼容的Sidecar一同运行，sidecar的注入方式有2种，一直是自动，还有一种是手动注入，下面我们来说下这2种注入方式要怎么配置。

## 自动注入

自动注入的原理是通过webhook实现的，MutatingWebhookConfiguration 配置了Webhook 何时会被 Kubernetes调用。Isti提供的缺省配置，会在带有istio-injection=enabled标签的命名空间中选择Pod。使用kubectl edit mutatingwebhookconfiguration istio-sidecar-injector 命令可以编辑目标命名空间的范围。

如果是用的腾讯云的服务网格，则需要改下命名空间的标签，如果版本是1.10.3，则label是istio.io/rev=1-10-3，后面的版本根据你的服务网格版本确认。

打上label后，新创建的pod，都会默认注入一个sidecar，如果标签是新加的，存量的pod则需要重建才会注入sidecar。

```
[root@vm-0-3-centos ~]# kubectl get ns mesh --show-labels
NAME   STATUS   AGE    LABELS
mesh   Active   292d   field.cattle.io/projectId=p-w6scm,istio.io/rev=1-10-3,kubesphere.io/namespace=mesh
[root@vm-0-3-centos ~]# kubectl get pod -n mesh -o=custom-columns=pod_name:.metadata.name,container_name:.spec.containers[*].name
pod_name                    container_name
client-76cf54b466-bpprz     istio-proxy,client
nginx-54cdc9d45c-f29wc      istio-proxy,nginx
nginx-54cdc9d45c-sdnlb      istio-proxy,nginx
nginx-v1-5b446c9fbc-8qtxv   istio-proxy,nginx-v1
nginx-v1-5b446c9fbc-dlnnh   istio-proxy,nginx-v1
nginx-v2-58455fc8d-4mnzr    istio-proxy,nginx-v2
sleep-86b8f9575c-p56gv      istio-proxy,sleep
```

从上面的结果看，mesh命名空间加了istio.io/rev这个label后，pod都自动注入了sidecar。

## 手动注入

通常手动注入，会有下面2种场景：

* 希望某个deployment不注入sidecar
* 只希望某个deployment注入sidecar


下面我们来针对上面的场景来说明下，如何在yaml配置。

### 某个pod不注入sidecar

一般我们给命名空间打上了自动注入的label后，新建的pod默认就会注入sidecar，但是有时候我们希望某些pod不被istio管理。这里则需要给这些pod不注入sidecar，其实可以在deploy的yaml配置下，是否注入sidecar，来实现单个pod不注入sidecar，具体yaml如下

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tke-debug
  namespace: default
  labels:
    app: debug
spec:
  replicas: 1
  selector:
    matchLabels:
      app: debug
  template:
    metadata:
      labels:
        app: debug
      annotations:
        sidecar.istio.io/inject: "false"
    spec:
      containers:
      - name: debug
        image: ccr.ccs.tencentyun.com/nwx_registry/troubleshooting:latest
      tolerations:
      - operator: Exists
      dnsPolicy: ClusterFirst
```

在Pod模板中加入 sidecar.istio.io/inject 注解并赋值为false才能覆盖缺省值并阻止对这一 Pod 的sidecar注入。

### 只给某个pod注入sidecar

只给某个pod注入sidecar，这里有2种方案，一种就是参考上面的方案，给不需要注入的sidecar的pod加上注解，剩下需要注入sidecar的pod设置为true，或者不配置，采用namespace自动注入即可。

上面的方法比较麻烦，如果一个命名空间下有较多的deploy，需要一个个配置才行，其实istio-system 命名空间中的ConfigMap istio-sidecar-injector中包含了缺省的注入策略以及 Sidecar的注入模板，腾讯云的服务网格是istio-sidecar-injector-1-10-3这个configmap。

```
apiVersion: v1
data:
  config: |-
    # defaultTemplates defines the default template to use for pods that do not explicitly specify a template
    defaultTemplates: [sidecar]
    policy: disabled
    alwaysInjectSelector:
      []
    neverInjectSelector:
      []
    ........  
```

配置中有个polocy字段，里面配置了是否自动注入的策略

* disabled：Sidecar注入器缺省不会向Pod进行注入。在Pod模板中加入sidecar.istio.io/inject注解并赋值为true才能覆盖缺省值并启用注入。
* enabled：Sidecar注入器缺省会对Pod进行注入。在Pod模板中加入sidecar.istio.io/inject注解并赋值为false才能覆盖缺省值并阻止对这一Pod的注入。

这里我们将policy改成disabled，这里打上了label的命名空间里面pod也不会自动注入sidecar，我们只需要将需要注入sidecar的pod在模板中加入sidecar.istio.io/inject注解并赋值为true即可，这样就只有配置了注解的pod才注入sidcar。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tke-debug
  namespace: default
  labels:
    app: debug
spec:
  replicas: 1
  selector:
    matchLabels:
      app: debug
  template:
    metadata:
      labels:
        app: debug
      annotations:
        sidecar.istio.io/inject: "true"
    spec:
      containers:
      - name: debug
        image: ccr.ccs.tencentyun.com/nwx_registry/troubleshooting:latest
      tolerations:
      - operator: Exists
      dnsPolicy: ClusterFirst
```

### 打label给单个deployment的pod注入sidecar

如果你用的istio是腾讯云托管的服务网格，并且版本是1.10.3，可以通过给deployment的.spec.template.metadata.labels中加istio.io/rev: 1-10-3这个label，来实现sidecar的自动注入，这里不需要你的namespace是否有加上istio.io/rev这个label。具体的yaml如下

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tke-debug
  namespace: tke-test
  labels:
    app: debug
spec:
  replicas: 1
  selector:
    matchLabels:
      app: debug
  template:
    metadata:
      labels:
        app: debug
        istio.io/rev: 1-10-3
    spec:
      containers:
      - name: debug
        image: ccr.ccs.tencentyun.com/nwx_registry/troubleshooting:latest
      tolerations:
      - operator: Exists
      dnsPolicy: ClusterFirst
```