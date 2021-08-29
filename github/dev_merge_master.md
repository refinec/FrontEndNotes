# 将master分支内容合并到dev分支

1. 将分支切换到master

   `git checkout master`

2. 将代码pull到本地

   `git pull`

3. 修改冲突

4. 提交到本地

   `git add .
   git commit -m "merge"`

5. 切换到你所在分支dev

   `git checkout dev`

6. merge

   `git merge master`

7. 将本地内容push到dev分支

   `git push`

