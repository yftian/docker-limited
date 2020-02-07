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
Docker: 限制容器可用的内存  
==  
限制内存使用上限
--
>在进入繁琐的设置细节之前我们先完成一个简单的用例：限制容器可以使用的最大内存为 300M。  
>-m(--memory=) 选项可以完成这样的配置：  
```
docker run -it -m 300M --memory-swap -1 --name con1 u-stress /bin/bash
```
限制可用的 swap 大小
--
强调一下 --memory-swap 是必须要与 --memory 一起使用的。  
>正常情况下， --memory-swap 的值包含容器可用内存和可用 swap。所以 --memory="300m" --memory-swap="1g" 的含义为：  
>容器可以使用 300M 的物理内存，并且可以使用 700M(1G -300M) 的 swap。--memory-swap 居然是容器可以使用的物理内存和可以使用的 swap 之和！  
>把 --memory-swap 设置为 0 和不设置是一样的，此时如果设置了 --memory，容器可以使用的 swap 大小为 --memory 值的两倍。  
```
$ docker run -it --rm -m 300M --memory-swap=300M u-stress /bin/bash
```
如果 --memory-swap 的值和 --memory 相同，则容器不能使用 swap。下面的 demo 演示了在没有 swap 可用的情况下向系统申请大量内存的场景： 

