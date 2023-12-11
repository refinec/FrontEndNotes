## 一、相关命令

### 搜索镜像

| 命令                                          | 说明                              |
| --------------------------------------------- | --------------------------------- |
| `docker search <名称关键字>`                  | 搜索镜像                          |
| `docker search <名称关键字> --filter=stars=N` | 搜索显示收藏数量为 `N` 以上的镜像 |

### 下载/上传镜像

| 命令                           | 说明     |
| ------------------------------ | -------- |
| `docker pull <镜像名:tag版本>` | 下载镜像 |
| `docker push <镜像名:tag版本>` | 上传镜像 |

* **上传镜像**

  > 以下命令中的 `username` 请替换为你的 Docker 账号用户名

  ```sh
  $ docker tag ubuntu:18.04 username/ubuntu:18.04
  
  $ docker image ls
  
  REPOSITORY                                               TAG                    IMAGE ID            CREATED             SIZE
  ubuntu                                                   18.04                  275d79972a86        6 days ago          94.6MB
  username/ubuntu                                          18.04                  275d79972a86        6 days ago          94.6MB
  
  $ docker push username/ubuntu:18.04
  
  $ docker search username
  
  NAME                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
  username/ubuntu
  ```

  

### 列出/查看镜像

| 命令                               | 说明                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| `docker images`                    | 列出本地镜像                                                 |
| `docker image ls [镜像名:tag版本]` | 列出(所有/具体)本地镜像                                      |
| `docker image ls -a`               | 列出所有本地镜像，加 `-a` 参数会显示包括中间层镜像在内的所有镜像 |
| `docker image ls -f [条件]`        | 列出过滤后的镜像，过滤器参数 `--filter`，或者简写 `-f`。<br />如，我们希望看到在 `mongo:3.2` 之后建立的镜像：`docker image ls -f since=mongo:3.2`。想查看某个位置之前的镜像也可以，只需要把 `since` 换成 `before` 即可 |
| `docker system df`                 | 查看镜像、容器、数据卷所占用的实际存储空间                   |

### 打包/导入镜像

| 命令                                             | 说明             |
| ------------------------------------------------ | ---------------- |
| `docker save -o <输出文件路径> <镜像名:tag版本>` | 打包本地镜像文件 |
| `docker load -i <加载文件路径>`                  | 导入本地镜像文件 |

### 删除镜像

| 命令                                           | 说明                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ |
| `docker rmi <镜像名:tag版本>`                  | 删除镜像                                                     |
| `docker image rm [选项] <镜像1> [<镜像2> ...]` | 删除镜像。<镜像>可以是 `镜像短 ID`、`镜像长 ID`、`镜像名`、`镜像摘要` |

## 二、获取镜像 `docker pull`

> 获取镜像命令格式：`docker pull [OPTIONS] [Docker Registry 地址[:端口号]/]仓库名[:标签]`

* `OPTIONS`

  > 具体的选项可以通过 `docker pull --help` 命令查看

* `Docker 镜像仓库地址`

  > 地址的格式一般是 `<域名/IP>[:端口号]`。默认地址是 Docker Hub(`docker.io`)。

* `仓库名`

  > 仓库名是两段式名称，即 `<用户名>/<软件名>`。
  >
  > 对于 Docker Hub，如果不给出用户名，则默认为 `library`，也就是官方镜像。

```bash
$ docker pull ubuntu:18.04
18.04: Pulling from library/ubuntu
92dc2a97ff99: Pull complete # 分层的 ID 的前 12 位
be13a9d27eb8: Pull complete # 分层的 ID 的前 12 位
c8299583700a: Pull complete # 分层的 ID 的前 12 位
Digest: sha256:4bc3ae6596938cb0d9e5ac51a1152ec9dcac2a1c50829c74abd9c4361e321b26 # 镜像完整的 sha256 的摘要，以确保下载一致性
Status: Downloaded newer image for ubuntu:18.04
docker.io/library/ubuntu:18.04			# 镜像的完整名称
```

### 例子

```bash
$ docker run -it --rm ubuntu:18.04 bash

root@e7009c6ce357:/# cat /etc/os-release
NAME="Ubuntu"
VERSION="18.04.1 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.1 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic
```

- `-it`：这是两个参数，一个是 `-i`：交互式操作，一个是 `-t` 终端。我们这里打算进入 `bash` 执行一些命令并查看返回结果，因此我们需要交互式终端。
- `--rm`：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 `docker rm`。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 `--rm` 可以避免浪费空间。
- `ubuntu:18.04`：这是指用 `ubuntu:18.04` 镜像为基础来启动容器。
- `bash`：放在镜像名后的是 **命令**，这里我们希望有个交互式 Shell，因此用的是 `bash`。

进入容器后，我们可以在 Shell 下操作，执行任何所需的命令。这里，我们执行了 `cat /etc/os-release`，这是 Linux 常用的查看当前系统版本的命令，从返回的结果可以看到容器内是 `Ubuntu 18.04.1 LTS` 系统。

最后我们通过 `exit` 退出了这个容器。

## 三、删除本地镜像

> 格式为：`docker image rm [选项] <镜像1> [<镜像2> ...]`

* `<镜像> `可以是 **镜像短 ID**、**镜像长 ID**、**镜像名(即`<仓库名>:<标签>`)**、**镜像摘要**

如：使用 `镜像摘要` 

```bash
$ docker image ls --digests
REPOSITORY                  TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
node                        slim                sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228   6e0c4c8e3913        3 weeks ago         214 MB

$ docker image rm node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
Untagged: node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
```

**Untagged 和 Deleted**

如果观察上面这几个命令的运行输出信息的话，你会注意到删除行为分为两类，一类是 `Untagged`，另一类是 `Deleted`。我们之前介绍过，镜像的唯一标识是其 ID 和摘要，而一个镜像可以有多个标签。

因此当我们使用上面命令删除镜像的时候，实际上是在要求删除某个标签的镜像。所以首先需要做的是将满足我们要求的所有镜像标签都取消，这就是我们看到的 `Untagged` 的信息。因为一个镜像可以对应多个标签，因此当我们删除了所指定的标签后，可能还有别的标签指向了这个镜像，如果是这种情况，那么 `Delete` 行为就不会发生。所以并非所有的 `docker image rm` 都会产生删除镜像的行为，有可能仅仅是取消了某个标签而已。

当该镜像所有的标签都被取消了，该镜像很可能会失去了存在的意义，因此会触发删除行为。镜像是多层存储结构，因此在删除的时候也是从上层向基础层方向依次进行判断删除。镜像的多层结构让镜像复用变得非常容易，因此很有可能某个其它镜像正依赖于当前镜像的某一层。这种情况，依旧不会触发删除该层的行为。直到没有任何层依赖当前层时，才会真实的删除当前层。这就是为什么，有时候会奇怪，为什么明明没有别的标签指向这个镜像，但是它还是存在的原因，也是为什么有时候会发现所删除的层数和自己 `docker pull` 看到的层数不一样的原因。

除了镜像依赖以外，还需要注意的是容器对镜像的依赖。如果有用这个镜像启动的容器存在（即使容器没有运行），那么同样不可以删除这个镜像。之前讲过，容器是以镜像为基础，再加一层容器存储层，组成这样的多层存储结构去运行的。因此该镜像如果被这个容器所依赖的，那么删除必然会导致故障。如果这些容器是不需要的，应该先将它们删除，然后再来删除镜像。

### 批量删除镜像

> 用 `docker image ls` 命令来配合
>
> 像其它可以承接多个实体的命令一样，可以使用 `docker image ls -q` 来配合使用 `docker image rm`，这样可以成批的删除希望删除的镜像。我们在“镜像列表”章节介绍过很多过滤镜像列表的方式都可以拿过来使用。

* 删除所有仓库名为 `redis` 的镜像：

  ```sh
  $ docker image rm $(docker image ls -q redis)
  ```

* 删除所有在 `mongo:3.2` 之前的镜像：

  ```sh
  $ docker image rm $(docker image ls -q -f before=mongo:3.2)
  ```

  

