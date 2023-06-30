# git常用操作

```bash
# 配置ssh
git config --global user.name "用户名"　
git config --global user.email "邮箱地址" 
ssh-keygen
# 拉取代码
git clone +仓库地址
# 查看远程分支
git branch -r
# 查看当前分支
git branch
#切换分支
git checkout +分支名(注意不要带origin)
# 在主分支上提交代码到其他分支
git push origim master：分支名(注意不要带origin)
# 在其他分支上提交本分支代码到远程
git push #当前在哪个分支就将代码提交到哪个分支上
# 在本地将其他分支合并到主分支
# 1.确保您当前在主分支上：
git checkout (main or master)
# 2. 获取远程最新主分支的代码：
git fetch origin (main or master 取决于主分支名叫什么)
# 3. 将其他分支合并到主分支中：
git merge <other-branch-name> 其中，<other-branch-name>是要合并到主分支的其他分支的分支名。
# 查看commit 日志
git log



```
