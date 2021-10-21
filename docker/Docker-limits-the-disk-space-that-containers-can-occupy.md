# docker限制容器可占用的磁盘空间

Docker容器默认启动的虚拟机，会占用宿主机的资源（CPU、内存、硬盘），例如默认Docker基于Overlay2驱动方式，容器硬盘的rootfs根分区空间是整个宿主机的空间大小。

可以指定默认容器的大小（在启动容器的时候指定），可以在docker配置文件指定Docker容器rootfs容量大小。

如果这里需要设置容器可用磁盘空间大小，需要保证节点的文件系统是xfs

具体配置如下，修改/etc/docker/daemon.json文件，设置容器的磁盘空间大小为20G。


```
{
    "data-root": "/data/docker",
    "storage-driver": "overlay2",
    "storage-opts": [
      "overlay2.override_kernel_check=true",
      "overlay2.size=20G"
    ]
}
```

