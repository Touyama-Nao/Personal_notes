# Git使用的错误的地方
报错1：
Git 报错git pull 失败 ,提示：fatal: refusing to merge unrelated histories
> 失败原因：这是因为远程仓库origin上的分支master和本地分支master被Git认为是不同的仓库，所以不能直接合并。

> 解决方案：添加--allow-unrelated-histories
附：push和pull命令格式：

1
<code><code><code><code>git push <远程主机名> <本地分支名>[:<远程分支名>]    e.g., git push oschina master， 省略远程分支名，代表推送到有追踪关系的远程分支（一般为同名）</code></code></code></code>

1
<code><code><code><code>git pull <远程主机名> <远程分支>:<本地分支>       e.g.

git pull --allow-unrelated-histories