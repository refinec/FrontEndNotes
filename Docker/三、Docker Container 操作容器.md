## 一、启动容器

### 1.基于镜像新建一个容器并启动

```shell
$ docker run <镜像>
```

启动一个 bash 终端，允许用户进行交互，最后可通过 `exit` 命令或 `Ctrl+d` 来退出终端

```shell
$ docker run -t -i <镜像> /bin/bash
root@af8bae53bdd3:/#
root@ba267838cc1b:/# ps  # 在伪终端中利用 `ps` 或 `top` 来查看进程信息
  PID TTY          TIME CMD
    1 ?        00:00:00 bash
   11 ?        00:00:00 ps
```

> * `-t` 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上
> * `-i` 则让容器的标准输入保持打开
>
> 当利用 `docker run` 来创建容器时，Docker 在后台运行的标准操作包括：
>
> - 检查本地是否存在指定的镜像，不存在就从 [registry]() 下载
> - 利用镜像创建并启动一个容器
> - 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
> - 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
> - 从地址池配置一个 ip 地址给容器
> - 执行用户指定的应用程序
> - 执行完毕后容器被终止

### 2.将在终止状态（`exited`）的容器重新启动

```shell
$ docker container ls -a
$ docker container start [container ID or NAMES]
```

```sh
$ docker container restart [container ID or NAMES]
```

## 二、容器守护态(后台)运行(`-d` 参数)

使用 `-d` 参数运行容器，此时容器会在后台运行并不会把输出的结果 (STDOUT) 打印到宿主机上面。

```sh
$ docker run -d ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
77b2dc01fe0f3f1265df143181e7b9af5e05279a884f4776ee75350ea9d8017a
```

要获取容器的输出信息，可以通过 `docker container logs` 命令

```sh
$ docker container logs [container ID or NAMES]
hello world
hello world
hello world
```

## 三、终止容器

```sh
$ docker container ls -a
$ docker container stop [container ID or NAMES]
```

## 四、进入容器 `docker exec`

```sh
$ docker run -dit ubuntu
69d137adef7a8a689cbcb059e94da5489d3cddd240ff675c640c8d96e84fe1f6

$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
69d137adef7a        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           zealous_swirles

$ docker exec -i 69d1 bash
ls
bin
boot
dev
...

$ docker exec -it 69d1 bash
root@69d137adef7a:/#
```

只用 `-i` 参数时，由于没有分配伪终端，界面没有我们熟悉的 Linux 命令提示符，但命令执行结果仍然可以返回。

当 `-i` `-t` 参数一起使用时，则可以看到我们熟悉的 Linux 命令提示符。

更多参数说明请使用 `docker exec --help` 查看

## 五、导出容器和导入容器快照

### 导出到本地`docker export`

```sh
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                    PORTS               NAMES
7691a814370e        ubuntu:18.04        "/bin/bash"         36 hours ago        Exited (0) 21 hours ago                       test
$ docker export 7691a814370e > ubuntu.tar
```

### 导入容器快照`docker import`

> *注：用户既可以使用* `*docker load*` *来导入镜像存储文件到本地镜像库，也可以使用* `*docker import*` *来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。*

1. 从本地容器快照文件中导入为镜像：

   ```sh
   $ cat ubuntu.tar | docker import - test/ubuntu:v1.0
   $ docker image ls
   REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
   test/ubuntu         v1.0                9d37a6082e97        About a minute ago   171.3 MB
   ```

2. 通过指定 URL 或者某个目录来导入：

   ```sh
   $ docker import http://example.com/exampleimage.tgz example/imagerepo
   ```

## 六、删除容器

使用 `docker container rm` 来删除一个处于**终止状态**的容器

```sh
$ docker container rm trusting_newton
trusting_newton
```

如果要删除一个运行中的容器，可以添加 `-f` 参数。Docker 会发送 `SIGKILL` 信号给容器

```sh
$ docker container rm -f trusting_newton
trusting_newton
```

