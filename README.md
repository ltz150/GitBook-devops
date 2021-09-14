# git常用命令

![git.png](https://upload-images.jianshu.io/upload_images/10797253-04e201a2a8cb1e30.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 常用命令

* git init 把当前目录初始化为仓库，多了一个.git的目录，这个目录是Git来跟踪管理版本库的 也不一定必须在空目录下创建Git仓库，选择一个已经有东西的目录也是可以的
* git add 文件名 告诉Git，把文件添加到仓库
* git commit -m "这里写注释"  

  告诉Git，把文件提交到仓库，-m后面输入的是本次提交的说明，可以输入任意内容，当然最好是有意义的

* git status   

  git status命令可以让我们时刻掌握仓库当前的状态

* git diff  

  查看difference，显示的格式正是Unix通用的diff格式

* git log 命令显示从最近到最远的提交日志,版本号是一个SHA1计算出来的一个非常大的数字，用十六进制表示 如果嫌输出信息太多，看得眼花缭乱的，可以试试加上--pretty=oneline参数，后面显示commit时的备注
* 用==HEAD==表示当前版本 上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100

  \`\`\` li.tianzeng@litianzeng MINGW64 /f/git \(master\) $ git log commit 9b378c6671d5d975d80e621f047140660f8be617 \(HEAD -&gt; master\) Author: ltz150 [ltz150@163.com](mailto:ltz150@163.com) Date: Sun Oct 28 16:38:53 2018 +0800

  版本3

commit 60f710a986e854da76fe71561c8bfd59979b00e5 Author: ltz150 [ltz150@163.com](mailto:ltz150@163.com) Date: Sun Oct 28 16:37:54 2018 +0800

```text
版本2
```

commit b73a42c36aac49acc8eb9bfe4d0d29aba2fd093a Author: ltz150 [ltz150@163.com](mailto:ltz150@163.com) Date: Sun Oct 28 16:35:20 2018 +0800

```text
版本1
```

```text
- git reset --hard HEAD^    恢复到上一个版本
从最近的一个commit恢复
上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100

- git reset --hard 9b378（版本号前几位）    恢复到指定版本
版本号没必要写全，前几位就可以了，Git会自动去找。当然也不能只写前一两位，因为Git可能会找到多个版本号，就无法确定是哪一个了。
- git reset HEAD <file>  
可以把暂存区的修改撤销掉（unstage），重新放回工作区
用HEAD时，表示最新的版本

- git reflog   
记录你的每一次历史命令

> Git跟踪并管理的是修改，而非文件

- git checkout --文件名  
把文件在工作区的修改全部撤销，这里有两种情况：

一种是文件自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

一种是文件已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

总之，就是让这个文件回到最近一次git commit或git add时的状态。
- git rm 文件名   
删除版本库里面的文件，git checkout -- 文件名，把版本库 的文件恢复到工作区

- git remote add origin   git@github.com:id/仓库名.git
把一个已有的本地仓库与github关联
- git push  
把当前分支推送

- git clone github地址  
从地址克隆仓库

> master和其他分支名字 才是指向提交的，，HEAD指向的就是当前分支

- git checkout -b 分支名字  
命令加上-b参数表示创建并切换，相当于以下两条命令：
git branch 分支名   #创建分支
git checkout 分支名  #切换分支
- git branch  
命令会列出所有分支，当前分支前面会标一个*号。
```

li.tianzeng@litianzeng MINGW64 /f/git \(master\) $ git checkout -b fenzhi Switched to a new branch 'fenzhi'

li.tianzeng@litianzeng MINGW64 /f/git \(fenzhi\) $ git branch

* fenzhi

  master

```text
- git checkout master   
切换到主分支
- git merge   
命令用于合并指定分支到当前分支
当前在master上，指定fenzhi合并到mastershang
```

li.tianzeng@litianzeng MINGW64 /f/git \(fenzhi\) $ git checkout master Switched to branch 'master' Your branch is up to date with 'origin/master'.

li.tianzeng@litianzeng MINGW64 /f/git \(master\) $ git merge fenzhi Updating a8baa73..49f9e1b Fast-forward number.txt \| 3 ++- 1 file changed, 2 insertions\(+\), 1 deletion\(-\)

\`\`\` ![2018-10-28-22-03-13.png](https://upload-images.jianshu.io/upload_images/10797253-84aa6565ba94c5e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* git branch -d fenzhi

  git branch -D 强行删除。

  删除fenzhi，建议合并后删除

> 当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容，再提交。

用git log --graph命令可以看到分支合并图。

## 分支策略

1. master分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活
2. 干活都在dev分支上，也就是说，dev分支是不稳定的
3. 每个人都有自己的分支，时不时地往dev分支上合并就可以了
4. 合并分支时，加上--no-ff参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而fast forward合并就看不出来曾经做过合并

![2018-10-28-22-41-48.png](https://upload-images.jianshu.io/upload_images/10797253-b8c9e80afe7f0748.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* git stash 把当前工作现场“储藏”起来，等以后恢复现场后继续工作
* git stash list 显示保存的工作是现场
* 恢复现场3种方式
  * git stash apply恢复，但是恢复后，stash内容并不删除
  * git stash pop，恢复的同时把stash内容也删了
* git remote git remote -v显示更详细的信息 要查看远程库的信息 ,远程仓库的默认名称是origin

## 推送分支

* git push origin master  

  git push origin dev  

> * master分支是主分支，因此要时刻与远程同步；
> * dev分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；
>   * bug分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；
> * feature分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发。

* git checkout -b dev origin/dev  

  创建远程origin的dev分支到本地,这个命令创建本地dev分支.

* git branch --set-upstream-to=origin/dev dev 本地dev分支与远程origin/dev分支的链接，根据提示，设置dev和origin/dev的链接
* git rebase rebase操作可以把本地未push的分叉提交历史整理成直线； rebase的目的是使得我们在查看历史提交的变化时更容易，因为分叉的提交需要三方对比。

## 标签tag

* git tag 用于新建一个标签，默认为HEAD，也可以指定一个commit id；
* git tag -a  -m "blablabla..."可以指定标签信息；
* git tag可以查看所有标签。
* git push origin 可以推送一个本地标签；
* git push origin --tags可以推送全部未推送过的本地标签；
* git tag -d 可以删除一个本地标签；
* git push origin :refs/tags/可以删除一个远程标签。

