# Dockerfile之CMD和ENTRYPOINT命令详解


CMD和ENTRYPOINT都是用来指定容器启动命令的？但是什么时候用CMD，什么时候用ENTRYPOINT呢？或者说当dockerfile里面同时存在CMD和ENTRYPOINT又是怎么样的一种情况呢？下面我们来详细说说这2个命令的用法。

讲解这2个命令之前，我们先说说CMD和ENTRYPOINT执行模式，分为shell和exec模式


## exec模式

```
CMD [ "top" ]
```

exec模式，相当于将启动的命令解析为一个数组，然后每个命令和参数都是用逗号都隔开，并用加上双引号，需要注意的是exec模式不会通过shell执行相关指令

```
CMD [ "echo", "$HOME" ]
```

上面这种情况是无法读取环境变量的，如果要读取环境变量要怎么做呢？

```
CMD [ "sh", "-c", "echo $HOME" ]
```

要在exec模式下读取环境变量，则需要指定shell执行。

## shell模式

使用shell模式时，docker 会以/bin/sh -c "task command" 的方式执行任务命令

```
CMD top
```

容器内的进程实际执行的命令是/bin/sh -c top

## CMD命令写法

```
CMD ["executable","param1","param2"] // 这是 exec 模式的写法，注意需要使用双引号。
CMD command param1 param2            // 这是 shell 模式的写法。
```

需要注意的是，如果你在容器启动的时候加了命令，会覆盖CMD命令，并且dockerfile里面如果有多个CMD命令，只有最后一个才会生效。

CMD也可以为ENTRYPOINT提供参数

```
ENTRYPOINT ["executable"]
CMD ["param1","param2"]
```

最终启动命令就是executable param1 param2

## ENTRYPOINT命令的写法

```
ENTRYPOINT ["executable", "param1", "param2"] // 这是 exec 模式的写法，注意需要使用双引号。
ENTRYPOINT command param1 param2              // 这是 shell 模式的写法。
```

如果指定的ENTRYPOINT启动命令，当前运行docker也有配置启动参数，参数会加到ENTRYPOINT的参数列表里面

```
FROM ubuntu
ENTRYPOINT [ "top", "-b" ]
```

容器启动命令为

```
docker run -it test:v1 -c
```

最终启动命令是top -b -c

如果ENTRYPOINT是shell模式，启动时候的命令行参数则会完全忽略，如果要覆盖ENTRYPOINT，可以通过--entrypoint来实现

## 同时使用CMD和ENTRYPOINT

对于 CMD 和 ENTRYPOINT 的设计而言，多数情况下它们应该是单独使用的。当然，有一个例外是 CMD 为 ENTRYPOINT 提供默认的可选参数。
我们大概可以总结出下面几条规律：

* 如果 ENTRYPOINT 使用了 shell 模式，CMD 指令会被忽略。
* 如果 ENTRYPOINT 使用了 exec 模式，CMD 指定的内容被追加为 ENTRYPOINT 指定命令的参数。
* 如果 ENTRYPOINT 使用了 exec 模式，CMD 也应该使用 exec 模式。

docker也给了一个表格来完美的解释了各类情况

![upload-image](images/2018031211363722.png) 
