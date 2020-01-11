# [基于Docker本地运行Kubernetes](https://www.kubernetes.org.cn/doc-5)

## 概览

下面的指引将高速你如何通过Docker创建一个单机、单节点的Kubernetes集群。

下图是最终的结果：

![](C:\Users\Asus\Desktop\20161107161910.jpg)

## 先决条件

\1. 你必须拥有一台安装有Docker的机器。

\2. 你的内核必须支持 memory and swap accounting 。确认你的linux内核开启了如下配置：

```
CONFIG_RESOURCE_COUNTERS=y
CONFIG_MEMCG=y
CONFIG_MEMCG_SWAP=y
CONFIG_MEMCG_SWAP_ENABLED=y
CONFIG_MEMCG_KMEM=y
```

\3. 以命令行参数方式，在内核启动时开启 memory and swap accounting 选项：

```
GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"
```

注意：以上只适用于GRUB2。通过查看/proc/cmdline可以确认命令行参数是否已经成功

传给内核：

```
$cat /proc/cmdline
BOOT_IMAGE=/boot/vmlinuz-3.18.4-aufs root=/dev/sda5 ro cgroup_enable=memory
swapaccount=1
```

**第一步：运行Etcd**

```
docker run --net=host -d gcr.io/google_containers/etcd:2.0.12 /usr/local/bin/etcd --addr=127.0.0.1:4001 --bind-addr=0.0.0.0:4001 --data-dir=/var/etcd/data
```

**第二步：启动master**

```
docker run \
    --volume=/:/rootfs:ro \
    --volume=/sys:/sys:ro \
    --volume=/dev:/dev \
    --volume=/var/lib/docker/:/var/lib/docker:ro \
    --volume=/var/lib/kubelet/:/var/lib/kubelet:rw \
    --volume=/var/run:/var/run:rw \
    --net=host \
    --pid=host \
    --privileged=true \
    -d \
    gcr.io/google_containers/hyperkube:v1.0.1 \
    /hyperkube kubelet --containerized --hostname-override=&quot;127.0.0.1&quot; --address=&quot;0.0.0.0&quot; --api-servers=http://localhost:8080 --config=/etc/kubernetes/manifests
```

这一步实际上运行的是 kubelet ，并启动了一个包含其他master组件的[pod](../userguide/pods.md）。

**第三步：运行service proxy**

```
docker run -d --net=host --privileged gcr.io/google_containers/hyperkube:v1.0.1 /hyperkube proxy --master=http://127.0.0.1:8080 --v=2
```

## 测试

此时你应该已经运行起了一个Kubernetes集群。你可以下载kubectl二进制程序进行测试：

```
(OS X) (linux)
```

注意： 再OS/X上你需要通过ssh设置端口转发：

```
boot2docker ssh -L8080:localhost:8080
```

列出集群中的节点：

```
kubectl get nodes
```

应该输出以下内容：

```
NAME LABELS STATUS
127.0.0.1 Ready
```

如果你运行了不同的Kubernetes集群，你可能需要指定 -s http://localhost:8080 选项来访问本地集群。

## 运行一个应用

```
kubectl -s http://localhost:8080 run nginx --image=nginx --port=80
```

运行 docker ps 你应该就能看到nginx在运行。下载镜像可能需要等待几分钟。

## 暴露为service

```
kubectl expose rc nginx --port=80
```

运行以下命令来获取刚才创建的service的IP地址。有两个IP，第一个是内部的

（CLUSTER_IP），第二个是外部的负载均衡IP。

```
kubectl get svc nginx
```

同样你也可以通过运行以下命令只获取第一个IP（CLUSTER_IP）：

```
kubectl get svc nginx --template={{.spec.clusterIP}}
```

通过第一个IP（CLUSTER_IP）访问服务：

```
curl <insert-cluster-ip-here>
```

注意如果再OSX上需要再boot2docker虚拟机上运行curl。

## 关于关闭集群的说明

上面的各种容器都是运行在 kubelet 程序的管理下，它会保证容器一直运行，甚至容器意外退出时也不例外。所以，如果想关闭集群，你需要首先关闭 kubelet 容器，再关闭其他。

可以使用 docker kill $(docker ps -aq) 。注意这样会关闭Docker下运行的所有容器，请谨慎使用。