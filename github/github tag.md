# tag

> 创建一个tag来指向软件开发中的一个关键时期，比如版本号更新的时候可以建一个“v2.0”、“v3.1”之类的标签，这样在以后回顾的时候会比较方便。tag的使用很简单，主要操作有：查看tag、创建tag、验证tag以及共享tag。

## 查看tag

* **列出所有tag**

  这样列出的tag是按字母排序的，和创建时间没关系。

  `git tag`

* **查看某些tag**

  `git tag -l v1.*`

## 创建tag

* **创建轻量级tag**

  `git tag v1.0`

* **创建带信息的tag**

  -m后面带的就是注释信息，这样在日后查看的时候会很有用

  `git tag -a v1.0-m "first version"` 

* **创建带签名的tag**(前提得有GPG私钥)

  `git tag -s v1.0-m "first version"`

* **为以前的commit添加tag**

  1. **查看以前的commit**

     `git log --oneline`

     假如有这样一个`commit：8a5cbc2 updated readme`

  2. **为其添加tag**

     `git tag -a v1.18a5cbc2`

## 删除tag

`git tag -d v1.0`

## 验证tag

在有GPG私钥的前提下验证tag：`git tag -v v1.0`

## 共享tag

在执行git push的时候，tag是不会上传到服务器的，比如创建tag后git push，在github网页上是看不到tag的，为了共享这些tag，必须这样：`git push origin --tags`