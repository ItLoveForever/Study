## 全局配置
### 设置用户名和邮箱,只是自身标明使用不影响后续的任何事项
```git
git config --global user.name "tianwei" # 自定义用户名
git config --global user.email "386033340@qq.com" # 自定义邮箱
git config --list 查看仓库的配置
git remote add origin 链接 # 增加远程地址
ssh -keygen -t rsa _C "注册网站的用户名"

```

### git常用操作
```git
git add . # 把目录下所有增加本地仓库
git commit -m "描述" # 提交并且增加描述
git init (地址) # 初始化仓库
git push # 将本地推送到远程,如果第一次可以加上 -u origin master 设置推送的目的地地址
git diff 文件 # 比较文件修改前后
git log # 历史提交记录
git check # 切换分支
git branch # 查看当前所有分支
git merge 分支名 # 在当前分支上合并分支
git reset --hard HEAD~ # 回退上一个版本(gti reset --hard~100 回退100个版本)
gti pull origin master --allow-unrelated-histories 冲突强制合并
```

```text
不要用git pull，用git fetch和git merge代替它。
git pull的问题是它把过程的细节都隐藏了起来，以至于你不用去了解git中各种类型分支的区别和使用方法。
当然，多数时候这是没问题的，但一旦代码有问题，你很难找到出错的地方。看起来git pull的用法会使你
吃惊，简单看一下git的使用文档应该就能说服你。

将下载（fetch）和合并（merge）放到一个命令里的另外一个弊端是，你的本地工作目录在未经确认的情况下
就会被远程分支更新。当然，除非你关闭所有的安全选项，否则git pull在你本地工作目录还不至于造成
不可挽回的损失，但很多时候我们宁愿做的慢一些，也不愿意返工重来。
```
*对于需要忽视的文件可以创建.ignore放进去*
### 分支相关
git branch 分支名字~~用于创建分支~~
git branch 查看分支
git check 分支名 切换分支
git commit -a -m同时提交和add
git branch -d 删除分支
git branch -D 强制删除
git branch check -b 分支明创建并切换分支
git merge  合并分支