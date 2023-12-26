# git config 配置

通过`git config --list`来查看你的`git配置信息`

## 1. 基本设置（初始化）

```shell
#设置用户名
$ git config --global user.name "你的名字"
#查看用户名
$ git config --global user.name

#设置邮箱
$ git config --global user.email "你的邮箱"
#查看邮箱
$ git config --global user.email

#初始化git版本库
$ git init

#创建.gitignore文件
$ touch .gitigonre
# 文件中写入需要忽略的文件名（示例:node_modules /dist .idea ...），如果需要忽略的文件已经提交到仓库，需要删除后，再次提交.gitignore文件才可生效

```



## 2. 配置别名

> 为常用命令创建别名，可以节省你花在终端的时间。比如我们每次都要输入完整的`checkout`、`commit`，很麻烦，我们可以通过设置别名来实现。

```shell
$ git config --global alias.co checkout
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status
```

设置完成后，你在通过`git config --list`查看配置，就会发现：

```shell
alias.co=checkout
alias.br=branch
alias.ci=commit
alias.st=status
```

之后你就可以通过`git co <branch>`来切换分支、`git st`查看分支状态了.

你还可以通过直接修改`~/.gitconfig`文件来编辑这些命令或添加更多命令：

```
[alias]
    co = checkout
    ci = commit
    br = branch
```



## 3. 配置代理

因为一些特殊网络原因，我们很多时候上`github`很不稳定，有时候我们推送一些代码会`403`失败。这时我们就可以通过设置代理来解决。

比如：我们设置一个本地代理。

```shell
git config --global http.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
```

设置完成后，你再通过`git config --list`查看配置，就会发现：

```powershell
http.proxy=http://127.0.0.1:1080
https.proxy=https://127.0.0.1:1080
```

设置代理成功后，某天，你想取消该代理，这时我们可以通过`unset`来取消代理设置。

```powershell
git config --global --unset http.proxy
git config --global --unset https.proxy
```

