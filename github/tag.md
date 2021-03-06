# tag

> 创建一个tag来指向软件开发中的一个关键时期，比如版本号更新的时候可以建一个“v2.0”、“v3.1”之类的标签，这样在以后回顾的时候会比较方便。tag的使用很简单，主要操作有：查看tag、创建tag、验证tag以及共享tag。

## 查看tag

* **列出所有tag**

  这样列出的tag是按字母排序的，和创建时间没关系。

  `git tag`

* **列出满足条件的 tag **

  `git tag -l v1.*`

## 创建tag

* **创建轻量级tag**

  `git tag v1.0`

* **创建带信息的tag**

  -m后面带的就是注释信息，这样在日后查看的时候会很有用

  `git tag -a v1.0 -m "first version"` 

  `git show tagName`  查看 tag 相关信息

* **创建带签名的tag**(前提得有GPG私钥)

  `git tag -s v1.0 -m "first version"`

  `git tag -v tagName `  验证签署标签

* **为以前的commit添加tag**

  1. **查看以前的commit**

     `git log --oneline`

     假如有这样一个`commit：8a5cbc2 updated readme`

  2. **为其添加tag**

     `git tag -a v1.18a5cbc2`

  3. **后期补标签**

  ```sql
  git log --pretty=oneline  #查看提交记录
  git tag -a tagName  SHA-1 # 在指定的提交指针处添加 标签
  ```

## 删除tag

`git tag -d v1.0`

## 验证tag

在有GPG私钥的前提下验证tag：`git tag -v v1.0`

## 共享tag

在执行git push的时候，tag是不会上传到服务器的，比如创建tag后git push，在github网页上是看不到tag的，为了共享这些tag:

* **git push origin v1.0`**   提交指定 tag
* **git push origin --tags**  提交所有未提交的 tag