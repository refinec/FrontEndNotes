> Compose 是用于定义和运行多容器 Docker 应用程序的工具。通过 Compose，您可以使用 YML 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务。

想了解 YML 文件配置，可以先阅读 [YAML 入门教程](https://www.runoob.com/w3cnote/yaml-intro.html)。

`Compose` 使用的三个步骤：

- 使用 `Dockerfile` 定义应用程序的环境。
- 使用 `docker-compose.yml` 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。
- 最后，执行 `docker-compose up` 命令来启动并运行整个应用程序。

## 一、yml 配置指令参考

### 1. `version` 指定本 yml 依从的 compose 哪个版本制定

```yaml
version: '3.8'
```

### 2. `services` 服务

以下指令都在`services`下具体的服务中使用。

### 3. `build` 指定构建镜像上下文路径

如 webapp 服务，指定为从上下文路径 `./dir/Dockerfile` 所构建的镜像：

```yaml
version: "3.7"
services:
  webapp:
    build: ./dir
```

或者，作为具有在上下文指定的路径的对象，以及可选的 `Dockerfile` 和 `args`：

```yaml
version: "3.7"
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
      labels:
        - "com.example.description=Accounting webapp"
        - "com.example.department=Finance"
        - "com.example.label-with-empty-value"
      target: prod
```

- `context`：上下文路径。
- `dockerfile`：指定构建镜像的 Dockerfile 文件名。
- `args`：添加构建参数，这是只能在构建过程中访问的环境变量。
- `labels`：设置构建镜像的标签。
- `target`：多层构建，可以指定构建哪一层。

### 4. `container_name` 定义容器名称

指定自定义容器名称，而不是生成的默认名称。

```yaml
container_name: my-web-container
```

### 5. `depends_on` 设置各`service`之间的依赖关系

- `docker-compose up` ：以依赖性顺序启动服务。在以下示例中，先启动 db 和 redis ，才会启动 web。
- `docker-compose up <SERVICE>` ：自动包含 SERVICE 的依赖项。在以下示例中，docker-compose up web 还将创建并启动 db 和 redis。
- `docker-compose stop` ：按依赖关系顺序停止服务。在以下示例中，web 在 db 和 redis 之前停止。

```yaml
version: "3.7"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

📢注意：web 服务不会等待 redis db 完全启动 之后才启动。

### 6. 添加环境变量

#### i.`environment` 添加环境变量

支持数组、字典、任何布尔值（布尔值需要用引号引起来，以确保 YML 解析器不会将其转换为 True 或 False）。

```yaml
environment:
  RACK_ENV: development
  SHOW: 'true'

# 或者
environment:
  - NODE_ENV=production
  - PORT=3000
```

#### ii. `env_file` 从文件添加环境变量

支持单个值 或 列表的多个值：

```yaml
env_file: .env
```

```yaml
env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```

### 7. 服务 端口配置

#### i. `expose` 暴露端口，但不映射到宿主机，只被连接的服务访问

仅可以指定内部端口为参数：

```yaml
expose:
 - "3000"
 - "8000"
```

#### ii.`ports` 端口映射

```yaml
ports:
  - "3000:3000"
```

### 8. `volumes` 卷挂载

将主机的数据卷或者文件挂载到容器里

```yaml
volumes:
  # 挂载日志目录
  - ./logs:/app/logs
  # 挂载文件
  - "/localhost/postgres.sock:/var/run/postgres/postgres.sock"
```

### 9. `image` 指定容器运行的镜像

以下格式都可以：

```yaml
image: redis
image: ubuntu:14.04
image: tutum/influxdb
image: example-registry.com:4000/postgresql
image: a4bc65fd # 镜像id
```

### 10. 网络配置

#### i. `network_mode` 设置网络模式

```yaml
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```

#### ii. `networks` 配置容器连接的网络

配置容器连接的网络，引用顶级 networks 下的条目 。

```yaml
services:
  some-service:
    networks:
      some-network:
        aliases:
         - alias1
      other-network:
        aliases:
         - alias2
networks:
  some-network:
    # Use a custom driver
    driver: custom-driver-1
  other-network:
    # Use a custom driver which takes special options
    driver: custom-driver-2
```

* **aliases** ：同一网络上的其他容器可以使用**服务名称 **或 此**别名**来连接到对应容器的服务。

### 11. `restart` 重启策略

```yaml
restart: "no"
restart: always
restart: on-failure
restart: unless-stopped
```

- `no`：是默认的重启策略，在任何情况下都不会重启容器。
- `unless-stopped`：在容器退出时总是重启容器，但是不考虑在Docker守护进程启动时就已经停止了的容器
- `always`：容器总是重新启动。
- `on-failure`：在容器非正常退出时（退出状态非0），才会重启容器。

📢注意：swarm 集群模式，请改用 `restart_policy`。

### 12. `command` 覆盖容器启动的默认命令 

```yaml
command: ["bundle", "exec", "thin", "-p", "3000"]
```

### 13. `healthcheck` 健康检查

用于检测 docker 服务是否健康运行

```yaml
healthcheck:
  test: ["CMD", "node", "-e", "require('http').get('http://localhost:3000/generatePdf', (r) => {process.exit(r.statusCode === 404 ? 0 : 1)})"] # 设置检测程序
  interval: 1m30s # 设置检测间隔
  timeout: 10s # 设置检测超时时间
  retries: 3 # 设置重试次数
  start_period: 40s # 启动后，多少秒开始启动检测程序
```

### 14. `logging` 日志记录配置

```yaml
logging:
	# driver：指定服务容器的日志记录驱动程序。有以下三个选项
  driver: "json-file" # 默认值
  driver: "syslog"
  driver: "none"
```

* 仅在 `json-file` 驱动程序下，可以使用以下参数，限制日志得数量和大小。

  ```yaml
  logging:
    driver: json-file
    options:
      max-size: "200k" # 单个文件大小为200k
      max-file: "10" # 最多10个文件
  ```

  当达到文件限制上限，会自动删除旧的文件。

* `syslog` 驱动程序下，可以使用 `syslog-address` 指定日志接收地址

  ```yaml
  logging:
    driver: syslog
    options:
      syslog-address: "tcp://192.168.0.42:123"
  ```

### 15. `cap_add，cap_drop` 添加、删除容器拥有的宿主机的内核功能

```yaml
cap_add:
  - ALL # 开启全部权限

cap_drop:
  - SYS_PTRACE # 关闭 ptrace权限
```

### 16. `cgroup_parent` 指定父 cgroup 组

为容器指定父 cgroup 组，意味着将继承该组的资源限制。

```yaml
cgroup_parent: m-executor-abcd
```

### 17. `deploy` 指定与service 的部署、运行相关的配置。只在 `swarm`集群模式下才会有用

**注**：仅支持 V3.4 及更高版本。

```yaml
version: "3.8"
services:
  redis:
    image: redis:alpine
    deploy:
      mode：replicated
      replicas: 6
      endpoint_mode: dnsrr
      labels: 
        description: "This redis service label"
      resources:
        limits:
          cpus: '0.50'
          memory: 50M
        reservations:
          cpus: '0.25'
          memory: 20M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
```

#### **可选参数**

* **`endpoint_mode`**：访问集群服务的方式

  ```yaml
  # 1. Docker 集群服务一个对外的虚拟 ip。所有的请求都会通过这个虚拟 ip 到达集群服务内部的机器。
  endpoint_mode: vip 
  
  # 2. DNS 轮询（DNSRR）。所有的请求会自动轮询获取到集群 ip 列表中的一个 ip 地址。
  endpoint_mode: dnsrr
  ```

* **`labels`**：在service上设置标签。可以用容器上的 `labels`（跟 `deploy` 同级的配置） 覆盖 `deploy` 下的 `labels`

* **`mode`**：指定service提供的模式

  - **`replicated`**：复制服务，复制指定服务到集群的机器上。
  - **`global`**：全局服务，服务将部署至集群的每个节点。

  图解：下图中黄色的方块是 `replicated` 模式的运行情况，灰色方块是 `global` 模式的运行情况。

  <img src="https://www.runoob.com/wp-content/uploads/2019/11/docker-composex.png" alt="img" style="zoom:50%;" />

* **replicas**：运行的节点数量

  在设置`mode: replicated` 时，需要使用此参数配置具体运行的节点数量。

* **`resources`**：配置服务器资源使用的限制

  例如该例子，配置 redis 集群运行需要的 cpu 的百分比 和 内存的占用。避免占用资源过高出现异常。

* **`restart_policy`**：配置容器重启策略

  - `condition`：可选 `none`、`on-failure`、 `any`（默认值：`any`）。
  - `delay`：设置延迟多久之后重启（默认值：`0`）。
  - `max_attempts`：最大容器重启次数（默认值：一直重试）。
  - `window`：设置容器重启超时时间（默认值：`0`）。

* **`rollback_config`**：配置更新失败情况下如何回滚服务

  - `parallelism`：一次要回滚的容器数。如果设置为`0`，则所有容器将同时回滚。
  - `delay`：每个容器组回滚之间等待的时间（默认为`0s`）。
  - `failure_action`：如果回滚失败，该怎么办。值包括：`continue`、`pause`（默认`pause`）。
  - `monitor`：每个容器更新后，持续观察是否失败了的时间 (ns|us|ms|s|m|h)（默认为0s）。
  - `max_failure_ratio`：在回滚期间可以容忍的故障率（默认为0）。
  - `order`：回滚期间的操作顺序。值包括：`stop-first`（串行回滚）、 `start-first`（并行回滚）（默认 `stop-first` ）。

* **`update_config`**：配置应如何更新服务，对于配置滚动更新很有用。

  * `parallelism`：一次更新的容器数。
  * `delay`：每个容器组回滚之间等待的时间（默认为`0s`）。
  * `failure_action`：如果回滚失败，该怎么办。值包括：`continue`、`pause`（默认`pause`）。
  * `monitor`：每个容器更新后，持续观察是否失败了的时间 (ns|us|ms|s|m|h)（默认为0s）。
  * `max_failure_ratio`：在回滚期间可以容忍的故障率（默认为0）。
  * `order`：回滚期间的操作顺序。值包括：`stop-first`（串行回滚）、 `start-first`（并行回滚）（默认 `stop-first` ）。

#### 单机时资源限制

  如果需要在单机部署时限制资源，建议使用 Docker 原生参数：

```yaml
services:
    kary-bff-server:
      # 替代方案：使用 mem_limit 和 cpus（已废弃但单机仍可用）
      mem_limit: 1g
      memswap_limit: 1g
      cpu_count: 1

      # 或者使用 cgroup 版本兼容的写法
      deploy:
        resources:
          limits:
            cpus: '1'
            memory: 1G
```

### 18. `devices` 指定设备映射列表

```yaml
devices:
  - "/dev/ttyUSB0:/dev/ttyUSB0"
```

### 19. `extra_hosts` 添加主机名映射

类似 `docker client --add-host`。

```yaml
extra_hosts:
 - "somehost:162.242.195.82"
 - "otherhost:50.31.209.229"
```

以上会在此服务的内部容器中 `/etc/hosts` 创建一个具有 ip 地址和主机名的映射关系：

```
162.242.195.82  somehost
50.31.209.229   otherhost
```

### 20. `dns` 自定义 DNS 服务器

支持单个值 或 列表的多个值。

```yaml
dns: 8.8.8.8

dns:
  - 8.8.8.8
  - 9.9.9.9
```

### 21. `dns_search` 自定义 DNS 搜索域

支持单个值 或 列表。

```yaml
dns_search: example.com

dns_search:
  - dc1.example.com
  - dc2.example.com
```

### 22. `entrypoint` 覆盖容器默认的 `entrypoint`

```yaml
entrypoint: /code/entrypoint.sh
```

也可以是以下格式：

```yaml
entrypoint:
    - php
    - -d
    - zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20100525/xdebug.so
    - -d
    - memory_limit=-1
    - vendor/bin/phpunit
```

### 23. `secrets` 存储敏感数据

存储敏感数据，例如密码：

```yaml
version: "3.1"
services:

mysql:
  image: mysql
  environment:
    MYSQL_ROOT_PASSWORD_FILE: /run/secrets/my_secret
  secrets:
    - my_secret

secrets:
  my_secret:
    file: ./my_secret.txt
```

### 24. `security_opt` 修改容器默认的 schema 标签

```yaml
security-opt：
  - label:user:USER   # 设置容器的用户标签
  - label:role:ROLE   # 设置容器的角色标签
  - label:type:TYPE   # 设置容器的安全策略标签
  - label:level:LEVEL  # 设置容器的安全等级标签
```

### 25. `stop_grace_period` SIGKILL 信号发送延迟时间

指定在容器无法处理 `SIGTERM` (或者任何 `stop_signal` 的信号)，等待多久后发送 SIGKILL 信号关闭容器。

```yaml
stop_grace_period: 1s # 等待 1 秒
stop_grace_period: 1m30s # 等待 1 分 30 秒 
```

默认的等待时间是 10 秒。

### 26. `stop_signal` 设置停止容器的替代信号

设置停止容器的替代信号，默认情况下使用 `SIGTERM` 。

```yaml
stop_signal: SIGUSR1 # 使用 SIGUSR1 替代信号 SIGTERM 来停止容器
```

### 27. sysctls 设置容器中的内核参数

支持数组、字典两种格式。

```yaml
sysctls:
  net.core.somaxconn: 1024
  net.ipv4.tcp_syncookies: 0

sysctls:
  - net.core.somaxconn=1024
  - net.ipv4.tcp_syncookies=0
```

### 28. `tmpfs` 在容器内安装一个临时文件系统

支持单个值 或 列表的多个值。

```yaml
tmpfs: /run

tmpfs:
  - /run
  - /tmp
```

### 29. `ulimits` 覆盖容器默认的 `ulimit`

```yaml
ulimits:
  nproc: 65535
  nofile:
    soft: 20000
    hard: 40000
```

## 二、使用示例

下面用 `Python` 来建立一个能够记录页面访问次数的 web 网站

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
   
   WORKDIR /code
   
   ENV FLASK_APP app.py
   ENV FLASK_RUN_HOST 0.0.0.0
   
   # 安装 gcc，以便诸如 MarkupSafe 和 SQLAlchemy 之类的 Python 包可以编译加速。
   RUN apk add --no-cache gcc musl-dev linux-headers
   COPY requirements.txt requirements.txt
   RUN pip install -r requirements.txt
   
   COPY . .
   CMD ["flask", "run"]
   ```

3. 编写 `docker-compose.yml` 文件

   ```yaml
   version: '3.8'
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
