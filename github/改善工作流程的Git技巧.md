##  1. Git别名

> 为常用命令创建别名，可以节省你花在终端的时间

```sh
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.br branch
```

无需键入`git checkout master`，你只需键入`git co master`即可。

你还可以通过直接修改`~/.gitconfig`文件来编辑这些命令或添加更多命令：

```
[alias]
    co = checkout
    ci = commit
    br = branch
```

## 2. 版本提交比较

比较同一文件的提交或版本之间差异的一种简单方法是使用`git diff`命令。

如果要比较不同提交之间的相同文件，请运行以下命令：

```shell
git diff $start_commit..$end_commit -- path/to/file
```

如果要比较两次提交之间的更改：

```shell
git diff $start_commit..$end_commit
```

这些命令将在终端内打开差异视图，但如果你更喜欢使用更直观的工具来比较差异的话，不妨试试`git difftool`。Meld是一款不错的查看器/编辑器，可用于直观地比较差异。

配置Meld：

```shell
git config --global diff.tool git-meld
```

查看差异：

```shell
git difftool $start_commit..$end_commit -- path/to/file
```

或者

```shell
git difftool $start_commit..$end_commit
```

## 3. 隐藏未提交的更改

如果你正在开发一个功能并且需要对项目进行紧急修复，那么你可能会碰到这样的情况：既不想提交一个未完成的功能，也不想丢失当前的更改。解决方案是使用`Git stash`命令临时删除这些更改：

```shell
git stash
```

`git stash`命令会隐藏更改，为你提供一个干净的工作目录并能够切换到新的分支进行更新，而无需提交无意义的快照来保存当前状态。

一旦你完成修复工作并想重新访问之前的更改，可以运行：

```shell
git stash pop
```

这样更改就会恢复。

如果你不再需要这些更改并想要清理存储堆栈，可以这样做：

```shell
git stash drop
```

## 4. 设置全局.gitignore

如果你想避免提交像`.DS_Store`或Vim `swp`这样的文件，可以设置一个全局的`.gitignore`文件。

创建文件：

```shell
touch ~/.gitignore
```

然后运行：

```shell
git config --global core.excludesFile ~/.gitignore
```

或者手动将以下内容添加到`~/.gitconfig`中：

```
[core]
  excludesFile = ~/.gitignore
```

你还可以创建一个列表，列出你希望Git忽略的内容。

## 5. 删除已从远程移除的本地分支

本地存储库中可能存在已经失效的分支。要在每次fetch/pull时删除这些远程存储库中已不再存在的分支，请运行：

```shell
git config --global fetch.prune true
```

或者手动将以下内容添加到`~/.gitconfig`中：

```
[fetch]
  prune = true
```

## 6. 更高效地使用Git blame

Git blame是一种发现**<u>谁更改了文件中某一行代码</u>**的便捷方式。你可以根据你要显示的内容，传递不同的标志：

```
git blame -w  # ignores white space

git blame -M  # ignores moving text

git blame -C  # ignores moving text into other files
```

## 7. HEAD的别名

`@`与`HEAD`相同。在rebase期间使用`@`可以帮上大忙：

```shell
git rebase -i @~2
```

## 8. 重置文件

### 方式一

当你在修改代码时，突然意识到所做的更改不是很好，并且想要重置这些更改时，你可以将文件重置为分支的HEAD，而非单击撤消你编辑的所有内容。

```shell
git reset --hard HEAD
```

如果你要重置单个文件：

```shell
git checkout HEAD -- path/to/file
```

如果你已经提交了更改，但仍想恢复，可以使用：

```shell
git reset --soft HEAD~1
```

### 方式二

```shell
git restore [file path] 撤回文件
```

