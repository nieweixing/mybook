# VirtualService中hosts字段的配置使用

VirtualService配置示例

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: nginx
  namespace: mesh
spec:
  hosts:
    - 172.16.85.174
    - nginx-test.mesh.svc.cluster.local
    - nginx-test
  gateways:
    - mesh/vpc-gw
  http:
    - route:
        - destination:
            host: nginx
            subset: v1
          weight: 20
        - destination:
            host: nginx
            subset: v2
          weight: 80
```


使用hosts字段列举虚拟服务的主机——即用户指定的目标或是路由规则设定的目标。这是客户端向服务发送请求时使用的一个或多个地址。只有访问客户端的Host字段为hosts配置的地址才能路由到后端服务。

hosts:
  - 172.16.85.174
  - nginx-test.mesh.svc.cluster.local
  - nginx-test


虚拟服务主机名可以是IP 地址、DNS 名称，或者依赖于平台的一个简称（例如 Kubernetes 服务的短名称），隐式或显式地指向一个完全限定域名（FQDN）。您也可以使用通配符（“*”）前缀，让您创建一组匹配所有服务的路由规则。虚拟服务的 hosts 字段实际上不必是 Istio 服务注册的一部分，它只是虚拟的目标地址。这让您可以为没有路由到网格内部的虚拟主机建模。

上面的VirtualService配置了多个hosts，并且挂载了一个gateways，客户端直接访问后端的service是可以通的，但是我们通过域名访问后端服务时候就需要指定host了。

首先我们直接访问下gateway域名，是无法访问通的，因为VirtualService流量规则指定了hosts，我们的请求Host没在配置列表中。

```
bash-4.4# curl nginx.istio.niewx.top
```

如果你希望能通过域名直接访问，可以将域名配置到hosts下，默认发起请求的Host就是域名本身

```
spec:
  hosts:
    - 172.16.85.174
    - nginx-test.mesh.svc.cluster.local
    - nginx-test
    - nginx.istio.niewx.top
```

直接访问域名

```
bash-4.4# curl -Iv nginx.istio.niewx.top
* Rebuilt URL to: nginx.istio.niewx.top/
*   Trying 10.0.0.183...
* TCP_NODELAY set
* Connected to nginx.istio.niewx.top (10.0.0.183) port 80 (#0)
> HEAD / HTTP/1.1
> Host: nginx.istio.niewx.top
> User-Agent: curl/7.61.1
> Accept: */*
> 
< HTTP/1.1 200 OK
HTTP/1.1 200 OK
< server: envoy
server: envoy
< date: Mon, 13 Sep 2021 10:34:12 GMT
date: Mon, 13 Sep 2021 10:34:12 GMT
< content-type: text/html
content-type: text/html
< content-length: 15
content-length: 15
< last-modified: Mon, 13 Sep 2021 05:50:32 GMT
last-modified: Mon, 13 Sep 2021 05:50:32 GMT
< etag: "613ee6a8-f"
etag: "613ee6a8-f"
< accept-ranges: bytes
accept-ranges: bytes
< x-envoy-upstream-service-time: 12
x-envoy-upstream-service-time: 12

< 
* Connection #0 to host nginx.istio.niewx.top left intact
```

下面我们在请求中指定Host为172.16.85.174看下

```
bash-4.4# curl --silent -H "Host: 172.16.85.174"  "nginx.istio.niewx.top"
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

指定Host为nginx-test.mesh.svc.cluster.local域名

```
bash-4.4# curl --silent -H "Host: nginx-test.mesh.svc.cluster.local"  "nginx.istio.niewx.top"
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

当我们在请求中指定配置的hosts列表中Host时候是可以访问成功的。


