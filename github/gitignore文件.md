## .gitignore文件

> 在 Git bash中使用`touch .gitignore`命令生成 .gitignore文件

**在该文件中，在每一行指定一个忽略规则。**

| 规则           | 说明                                                         |
| :------------- | ------------------------------------------------------------ |
| `.a`           | 忽略所有 `.a` 结尾的文件                                     |
| `!lib.a`       | `!lib.a`除外                                                 |
| `/TODO`        | `/TODO`仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO  |
| `build/`       | 忽略 `build/`目录下的所有文件，过滤整个build文件夹           |
| `doc/.txt`     | 忽略`doc/notes.txt`但不包括 `doc/server/arch.txt`            |
| `bin/`         | 忽略当前路径下的bin文件夹，该文件夹下的所有内容都会被忽略，**不忽略 bin 文件** |
| `/bin`         | 表示忽略根目录下的**bin文件**                                |
| `/.c`          | 表示忽略`cat.c`，不忽略` build/cat.c`                        |
| `debug/.obj`   | 忽略`debug/io.obj`，不忽略 `debug/common/io.obj`和`tools/debug/io.obj` |
| `/foo`         | 忽略/foo, a/foo, a/b/foo等                                   |
| `a//b`         | 忽略a/b, a/x/b, a/x/y/b等                                    |
| `!/bin/run.sh` | 不忽略bin目录下的run.sh文件                                  |
| `*.log`        | 忽略所有 .log 文件                                           |
| `config.php`   | 忽略当前路径的 `config.php` 文件                             |
| `/mtk/`        | 过滤整个文件夹                                               |
| `*.zip`        | 过滤所有.zip文件                                             |
| /mtk/do.c      | 过滤某个具体文件                                             |
| fd1/*          | 忽略目录 fd1 下的全部内容；不管是根目录下的 /fd1/ 目录，还是某个子目录 /child/fd1/ 目录，都会被忽略 |
| /fd1/*         | 忽略根目录下的 /fd1/ 目录的全部内容                          |

```
/*
!.gitignore
!/fw/
/fw/*
!/fw/bin/
!/fw/sf/
```

**忽略全部内容，但是不忽略 .gitignore 文件、根目录下的 /fw/bin/ 和 /fw/sf/ 目录；注意要先对bin/的父目录使用!规则，使其不被排除。**

**需要注意的是，gitignore还可以指定要将哪些文件添加到版本管理中** 如：

* !*.zip
* !/mtk/one.txt

唯一的区别就是规则开头多了一个感叹号

假如我们只需要管理/mtk/目录中的one.txt文件，这个目录中的其他文件都不需要管理，那么.gitignore规则应写为：：
/mtk/*
!/mtk/one.txt

**注意：**如果不慎在创建.gitignore文件之前就push了项目，那么即使你在.gitignore文件中写入新的过滤规则，这些规则也不会起作用，Git仍然会对所有文件进行版本管理。简单来说出现这种问题的原因就是Git已经开始管理这些文件了，所以你无法再通过过滤规则过滤它们。当然，**解决办法是清理git 缓存**

