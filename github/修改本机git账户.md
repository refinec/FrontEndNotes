# 修改本机git账户

```sh
git config --global -l:查看全局配置
git config --system -l：查看本地配置

git config user.name：查看用户名
git config user.email：查看邮箱
git config user.name "你的用户名"：修改你本地一个仓库的用户名
git config user.email"你的邮箱"：修改你本地一个仓库的邮箱
git config --global user.name"你的用户名"：修改全局仓库的用户名
git config --global user.email"你的邮箱"：修改全局仓库的邮箱

cd ~/.ssh // 如果是Mac 先进这里
ssh-keygen -t rsa -C "邮箱"
```

## github后台配置ssh key之后本地无法git clone的问题 Permission denied (publickey).

当你在github后台添加了ssh keys之后，如果你在本地 git clone git://www.somesite.com/test.git 的时候出现了一些问题，不如access denied，那么你要在本地这么测试一下：

ssh -T git@github.com

如果返回是：

Permission denied (publickey).

那么你可能要在本地ssh-add一下，当然在这之前你可以使用 ssh -vT git@github.com 查看一下到底是因为什么原因导致的失败。

ssh-add ~/.ssh/youraccount_rsa

然后会返回如下：

Enter passphrase for /Users/andy/.ssh/youraccount_rsa:
Identity added: /Users/andy/.ssh/youraccount_rsa (/Users/andy/.ssh/youraccount_rsa)

之后再使用 ssh -T git@github.com

会返回成功：

Hi youraccount! You've successfully authenticated, but GitHub does not provide shell access.

说明你目前本地的ssh已经切换到了youraccount这个账号，

之后便可以进行git clone到本地：

git clone git@github.com:youraccount/yourproject.git

另：如果你有使用github官方的那个小猫app，那么当你git -T git@github.com的时候会使用你在github这个程序中设置的github账号进行登录，而不是ssh的rsa账号登录。



