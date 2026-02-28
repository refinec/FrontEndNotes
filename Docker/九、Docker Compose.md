## docker compose

> `Compose` 允许用户通过一个单独的 `docker-compose.yml` 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（**project**）。

`Compose` 中有两个重要的概念：

- 服务 (`service`)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
- 项目 (`project`)：由一组关联的应用容器组成的一个完整业务单元，在 `docker-compose.yml` 文件中定义。

## 一、安装

`Docker Desktop for Mac/Windows` 自带 `docker-compose` 二进制文件，安装 Docker 之后可以直接使用。

```sh
$ docker-compose --version
docker-compose version 1.27.4, build 40524192
```

### 二进制包

在 Linux 上安装可以从 [官方 GitHub Release](https://github.com/docker/compose/releases) 处直接下载编译好的二进制文件

例如，在 Linux 64 位系统上直接下载对应的二进制包

```sh
$ sudo curl -L https://github.com/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 国内用户可以使用以下方式加快下载
$ sudo curl -L https://download.fastgit.org/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose
```

### PIP 安装

*注：* `x86_64` 架构的 Linux 建议按照上边的方法下载二进制包进行安装，如果您计算机的架构是 `ARM` (例如，树莓派)，再使用 `pip` 安装。

这种方式是将 Compose 当作一个 Python 应用来从 pip 源中安装。

执行安装命令：

```sh
$ sudo pip install -U docker-compose

Collecting docker-compose
  Downloading docker-compose-1.27.4.tar.gz (149kB): 149kB downloaded
...
Successfully installed docker-compose cached-property requests texttable websocket-client docker-py dockerpty six enum34 backports.ssl-match-hostname ipaddress
```

### bash 补全命令

```sh
$ curl -L https://raw.githubusercontent.com/docker/compose/1.27.4/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose
```

## 二、卸载

如果是二进制包方式安装的，删除二进制文件即可。

```sh
$ sudo rm /usr/local/bin/docker-compose
```

如果是通过 `pip` 安装的，则执行如下命令即可删除。

```sh
$ sudo pip uninstall docker-compose
```

## 三、使用

> 下面用 `Python` 来建立一个能够记录页面访问次数的 web 网站

1. 在web 项目目录中编写 `app.py` 文件

   ```python
   from flask import Flask
   from redis import Redis
   
   app = Flask(__name__)
   redis = Redis(host='redis', port=6379)
   
   @app.route('/')
   def hello():
       count = redis.incr('hits')
       return 'Hello World! 该页面已被访问 {} 次。\n'.format(count)
   
   if __name__ == "__main__":
       app.run(host="0.0.0.0", debug=True)
   ```

2. 编写 `Dockerfile` 文件

   ```dockerfile
   FROM python:3.6-alpine
   ADD . /code
   WORKDIR /code
   RUN pip install redis flask
   CMD ["python", "app.py"]
   ```

3. 编写 `docker-compose.yml` 文件

   ```dockerfile
   version: '3'
   services:
   
     web:
       build: .
       ports:
        - "5000:5000"
   
     redis:
       image: "redis:alpine"
   ```

4. 运行 compose 项目

   ```sh
   $ docker-compose up
   ```

   此时访问本地 `5000` 端口，每次刷新页面，计数就会加 1。

## 四、命令说明

参考链接：https://yeasy.gitbook.io/docker_practice/compose/commands

## 五、Compose 模板文件

参考链接：https://yeasy.gitbook.io/docker_practice/compose/compose_file

## 六、实战

1. [实战 Django](https://yeasy.gitbook.io/docker_practice/compose/django)
2. [实战 Rails](https://yeasy.gitbook.io/docker_practice/compose/rails)
3. [实战 WordPress](https://yeasy.gitbook.io/docker_practice/compose/wordpress)
4. [实战 LNMP](https://yeasy.gitbook.io/docker_practice/compose/lnmp)