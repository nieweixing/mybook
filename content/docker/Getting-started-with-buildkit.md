# buildkit入门使用

最近因为K8s抛弃Docker了,所以就只装了个containerd,这样就需要一个单独的镜像构建工具了,就用了buildkit,这也是Docker公司扶持的,他们公司的人出来搞的开源工具,官网在 https://github.com/moby/buildkit

## 架构

* 服务端为buildkitd,负责和runc或containerd后端连接干活,目前只支持这两个后端
* 客户端为buildctl,负责解析镜像构建文件Dockerfile,并向服务端发出构建指令,所以客户端可以和服务端不在一台机器上,也不需要root权限之类
* 服务端默认使用runc后端,但是建议使用containerd后端,这样构建出的镜像就会存在containerd的buildkit名字空间下

## buildkit安装部署

登录你的机器，我这里机器是k8s的节点，节点安装的runtime是containerd，然后从github上下载安装包

```
# wget https://github.com/moby/buildkit/releases/download/v0.9.3/buildkit-v0.9.3.linux-amd64.tar.gz
# tar -xvf buildkit-v0.9.3.linux-amd64.tar.gz
```

将安装包解压后，我们来启动buildkitd服务，使用--oci-worker=false --containerd-worker=true参数,可以让buildkitd服务使用containerd后端

```
buildkitd --oci-worker=false --containerd-worker=true & 
```

## 采用buildctl构建本地镜像

这里我们写一个简单的dockerfile来构建镜像

```
[root@nwx-gr-node1 ~]# cat Dockerfile
FROM centos:7

RUN yum install openssh-server openssh-clients tree nmap dos2unix lrzsz nc lsof wget tcpdump htop iftop iotop sysstat nethogs bind-u
```

然后执行下面命令来构建镜像

```
buildctl build \
    --frontend=dockerfile.v0 \
    --local context=. \
    --local dockerfile=. \
    --output type=image,name=ccr.ccs.tencentyun.com/v_cjweichen/nwx-reg:buildctl
```

* frontend可以使用网关做前端,未做其他尝试,这里直接使用dockerfile.0
* --local context 指向当前目录,这是Dockerfile执行构建时的路径上下文,比如在从目录中拷贝文件到镜像里
* --local dockerfile指向当前目录,表示Dockerfile在此目录
* --output 的 name 表示构建的镜像名称
* 构建完成后镜像会存在本地containerd的buildkit名字空间下

## 推送镜像到远程仓库

这里我们先查看下刚构建好的镜像，可以用containerd的命令行工具ctr，类似已docker命令

```
[root@nwx-gr-node1 ~]# ctr -n buildkit i ls
REF                                                 TYPE                                                 DIGEST                                                                  SIZE      PLATFORMS   LABELS
ccr.ccs.tencentyun.com/v_cjweichen/nwx-reg:buildctl application/vnd.docker.distribution.manifest.v2+json sha256:82c93337b755c5b1cb7ce7370aba1293ce0ba89d87782083ea2c54a7a05f73c9 146.2 MiB linux/amd64 -
```

然后执行下面命令进行镜像推送

```
[root@nwx-gr-node1 ~]# ctr -n buildkit i push -u xxxx:xxxx ccr.ccs.tencentyun.com/v_cjweichen/nwx-reg:buildctl
manifest-sha256:82c93337b755c5b1cb7ce7370aba1293ce0ba89d87782083ea2c54a7a05f73c9: done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:bfadf5a699433130807948d95bf0f130370b0fc1187e13dde4f5eb889fa2906b:    done           |++++++++++++++++++++++++++++++++++++++|
config-sha256:e863cf7ae13f8eae21f9aea93c3d5d4ffbac9ca6222edf7c561081b889a9d38d:   done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:2d473b07cdd5f0912cd6f1a703352c82b512407db6b05b43f2553732b55df3bc:    done           |++++++++++++++++++++++++++++++++++++++|
elapsed: 4.8 s                                                                    total:  146.2  (30.3 MiB/s)
```

## 构建自动推送镜像到镜像仓库

上面我们是先构建好镜像，然后再推送镜像，其实buildkit支持构建完自动推送镜像。

首先配置下镜像仓库的登录配置文件，registry的帐号密码配置在 ~/.docker/config.json 文件中 , 这里沿用了Docker的配置,虽然我们并没有装Docker

```
{
        "auths": {
                "docker.io": {
                        "auth": "base64(username:password)"
                }
        }
}
```

登录镜像的仓库的配置文件设置好之后，在执行命令构建并推送镜像，其实就是在后面加上push=true的参数即可

```
buildctl build \
    --frontend=dockerfile.v0 \
    --local context=. \
    --local dockerfile=. \
    --output type=image,name=ccr.ccs.tencentyun.com/v_cjweichen/nwx-reg:buildctl,push=true
```