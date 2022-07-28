---
sidebar_position: 1
---

作者：@金宏辉

Git官网：[https://git-scm.com/](https://git-scm.com/)

首先，打开Git官网下载安装程序，然后按照默认选项安装即可。

我们需要设置一下默认给一个Git的账号：


```
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"

```

#### 初始化仓库

在需要初始化仓库的目录，右键打开  ，通过`git init`命令在其目录下，生成.git.文件夹（默认隐藏），此时还只是本地仓库，我们使用还需要连接远程仓库。

#### 将本地仓库关联到远程仓库

我们本地有一个仓库，我们想把它推送到远程上去，很简单，我们只需要使用`git remote add origin`命令就可以了，`ongin`是`github / coding`上的仓库名称，意思是远程仓库的意思。

#### 克隆仓库（常用）

当我们远程有仓库时，想要关联到本地只需要使用`git clone`就可以了，新建一个空目录，不要`git init`。使用`git clone`会自动帮我们初始化

鉴于刚刚的，我们上传的代码在远程仓库中有一个默认的`main`和`master`，由于我们最初上传的分支是`master`，所以`github`给我们创建了一个新的分支叫`master`，并没有关联到`mian`中，我们拉取时，默认拉取的是`main`分支。

所以我们可以使用`git clone -b分支名 仓库地址`来指定分支

#### 切换分支（常用）

首先我们查看本地分支`git branch `，`git branch -a `查看项目所有分支，`git checkout -b` 切换分支。

例如我切换到 king: `git checkout -b king origin/king`

#### 上传操作（常用）

首先添加文件进入暂存区，`git add .`  .代表着所有文件

 然后提交 文件 `git commit -m[提交信息]`  

最后直接` git push` 进入远程仓库 

#### 拉取操作（常用）

将远程指定分支 拉取到 本地指定分支上：`git pull origin <远程分支名>:<本地分支名>`

将远程指定分支 拉取到 本地当前分支上：`git pull origin <远程分支名>` 

将与本地当前分支同名的远程分支 拉取到 本地当前分支上(需先关联远程分支，方法见文章末尾，只需关联一次) :`git pull`

#### Git本地回退版本并且更新远程仓库

首先根据提交记录找到你想要回到的`commit id`（版本号）
然后回退到这个版本
`git reset --hard 3628164`
提交到仓库
`git push –force `

#### git删除分支

删除本地分支 `git branch -d 本地分支名`
删除远程分支 `git push origin --delete 远程分支名`
推送空分支到远程（删除远程分支另一种实现）`git push origin :远程分支 `



#### 删除本地分支区别

`git branch -d `会在删除前检查merge状态（其与上游分支或者与head）。
`git branch -D` 是git branch --delete --force的简写，它会直接删除。
**共同点：**
都是删除本地分支的方法（与删除远程分支命令相独立，要想本地和远程都删除，必须得运行两个命令）。 

#### Git查看分支

查看本地分支 `git branch`
查看远程分支 `git branch -r`
查看本地和远程分支 `git branch -a `
