# centos7如何重新安装yum

## 问题现象

centos的机器执行yum报错No module named yum

## 问题分析

查了下报错，这个报错是因为python版本和yum版本不一致导致的，需要修改/usr/bin/yum，将 #!/usr/bin/python 修改为 #!/usr/bin/python2.6，但是linux已经没有python2.6了，当前只能重新安装python和yum来解决。

## 解决方案

这里说说如何在centos7重新安装python和yum

### 卸载yum和python

```
rpm -qa|grep python|xargs rpm -e --allmatches --nodeps
whereis python|xargs rm -fr
rpm -qa|grep yum|xargs rpm -e --allmatches --nodeps
whereis yum|xargs rm -fr
```

### 安装python和yum

首先下载对应的rpm包

```
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-2.7.5-89.el7.x86_64.rpm 
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-iniparse-0.4-9.el7.noarch.rpm  
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-pycurl-7.19.0-19.el7.x86_64.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-devel-2.7.5-89.el7.x86_64.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-libs-2.7.5-89.el7.x86_64.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-urlgrabber-3.10-10.el7.noarch.rpm
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/rpm-python-4.11.3-45.el7.x86_64.rpm 
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-3.4.3-168.el7.centos.noarch.rpm 
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-metadata-parser-1.1.4-10.el7.x86_64.rpm 
wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.31-54.el7_8.noarch.rpm
```

安装python和yum，因为yum和python有相互依赖，需要同时安装，指定--nodeps参数即可

```
rpm -ivh python-* rpm-python-* yum-* --nodeps
```

安装完成后，就可以执行yum操作了。