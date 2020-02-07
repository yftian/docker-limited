Docker: 限制容器可用的 CPU  
==  
>默认情况下容器可以使用的主机 CPU 资源是不受限制的。和内存资源的使用一样，如果不对容器可以使用的 CPU 资源进行限制，一旦发生容器内程序异常使用 CPU 的情况，很可能把整个主机的 CPU 资源耗尽，从而导致更大的灾难。本文将介绍如何限制容器可以使用的 CPU 资源。  

限制可用的 CPU 个数  
--
>在 docker 1.13 及更高的版本上，能够很容易的限制容器可以使用的主机 CPU 个数。只需要通过 --cpus 选项指定容器可以使用的 CPU 个数就可以了，并且还可以指定如 1.5 之类的小数。  
通过下面的命令创建容器，--cpus=2 表示容器最多可以使用主机上两个 CPU：  
```
$ docker run -it --rm --cpus=2 u-stress:latest /bin/bash
```
指定固定的 CPU  
--  
>通过 --cpus 选项我们无法让容器始终在一个或某几个 CPU 上运行，但是通过 --cpuset-cpus 选项却可以做到！这是非常有意义的，因为现在的多核系统中每个核心都有自己的缓存，如果频繁的调度进程在不同的核心上执行势必会带来缓存失效等开销。  
```
docker run -it --rm --cpuset-cpus="1" u-stress:latest /bin/bash  
```
设置使用 CPU 的权重  
--
>当 CPU 资源充足时，设置 CPU 的权重是没有意义的。只有在容器争用 CPU 资源的情况下， CPU 的权重才能让不同的容器分到不同的 CPU 用量。--cpu-shares 选项用来设置 CPU 权重，它的默认值为 1024。我们可以把它设置为 2 表示很低的权重，但是设置为 0 表示使用默认值 1024。  
```
$ docker run -it --rm --cpuset-cpus="0" --cpu-shares=512 u-stress:latest /bin/bash
$ docker run -it --rm --cpuset-cpus="0" --cpu-shares=1024 u-stress:latest /bin/bash
```
