## 1. 版本提交比较

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

## 2. 删除已从远程移除的本地分支

本地存储库中可能存在已经失效的分支。要在每次`fetch/pull`时删除这些远程存储库中已不再存在的分支，请运行：

```shell
$ git config --global fetch.prune true
```

或者手动将以下内容添加到`~/.gitconfig`中：

```
[fetch]
  prune = true
```

## 4. `HEAD `的别名

`@`与`HEAD`相同。在`rebase`期间使用`@`可以帮上大忙：

```shell
$ git rebase -i @~2
```

## 5. 重置文件

### 方式一

> 当你在修改代码时，突然意识到所做的更改不是很好，并且想要重置这些更改时，你可以将文件重置为分支的`HEAD`，而非单击撤消你编辑的所有内容。
>

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

> 撤回文件

```shell
git restore [file path]
```

