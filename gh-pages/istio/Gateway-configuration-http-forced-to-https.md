# gateway配置http强转https

使用istio的过程中，有时候不想让用户可以http访问，这时候就需要在gateway配置http强转为https访问，下面我们来说明下如何在gateway配置http强转https。

首先我们测试下正常配置http和https，看下是否分别通过http和https访问到后端的服务。

```
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: vpc-gw
  namespace: mesh
spec:
  selector:
    app: istio-ingressgateway-vpc
    istio: ingressgateway
  servers:
  - hosts:
    - nginx.istio.niewx.top
    port:
      name: HTTP-80-iy2r
      number: 80
      protocol: HTTP
  - hosts:
    - nginx.istio.niewx.top
    port:
      name: HTTPS-443-1krv
      number: 443
      protocol: HTTPS
    tls:
      credentialName: vpc-gw-https-443-1krv
      mode: SIMPLE
```

分别通过http和https都可以成功的访问到后端。

```
[root@VM-0-13-centos ~]# curl -I  http://nginx.istio.niewx.top
HTTP/1.1 200 OK
server: istio-envoy
date: Tue, 21 Sep 2021 15:41:56 GMT
content-type: text/html
content-length: 15
last-modified: Sat, 18 Sep 2021 18:32:54 GMT
etag: "614630d6-f"
accept-ranges: bytes
x-envoy-upstream-service-time: 26

[root@VM-0-13-centos ~]# curl -I  https://nginx.istio.niewx.top
HTTP/1.1 200 OK
server: istio-envoy
date: Tue, 21 Sep 2021 15:42:16 GMT
content-type: text/html
content-length: 15
last-modified: Sat, 18 Sep 2021 18:32:54 GMT
etag: "614630d6-f"
accept-ranges: bytes
x-envoy-upstream-service-time: 1

```

gateway配置http强转https，只需要在gateway的http的配置中加上如下配置即可。

```
tls:
  httpsRedirect: true
```

下面我们在gateway中加上强制跳转的配置，再来通过http访问下。

```
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: vpc-gw
  namespace: mesh
spec:
  selector:
    app: istio-ingressgateway-vpc
    istio: ingressgateway
  servers:
  - hosts:
    - nginx.istio.niewx.top
    port:
      name: HTTP-80-iy2r
      number: 80
      protocol: HTTP
    tls:
      httpsRedirect: true
  - hosts:
    - nginx.istio.niewx.top
    port:
      name: HTTPS-443-1krv
      number: 443
      protocol: HTTPS
    tls:
      credentialName: vpc-gw-https-443-1krv
      mode: SIMPLE
```

从下面的测试结果可以发现，访问http的时候会出现301，说明我们配置的永久重定向成功了。

```
[root@VM-0-13-centos ~]# curl -I  http://nginx.istio.niewx.top
HTTP/1.1 301 Moved Permanently
location: https://nginx.istio.niewx.top/
date: Tue, 21 Sep 2021 15:45:12 GMT
server: istio-envoy
transfer-encoding: chunked

[root@VM-0-13-centos ~]# curl -I  https://nginx.istio.niewx.top
HTTP/1.1 200 OK
server: istio-envoy
date: Tue, 21 Sep 2021 15:45:14 GMT
content-type: text/html
content-length: 15
last-modified: Sat, 18 Sep 2021 18:32:54 GMT
etag: "614630d6-f"
accept-ranges: bytes
x-envoy-upstream-service-time: 1
```