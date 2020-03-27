---
title: git-命令行操作
---
这里参考廖雪峰老师的教程，一步步重头操作一遍。

##概念
Git：分布式版本控制系统
和svn 这种集中式的版本控制系统比较

集中式版本控制系统：版本库是存放在中央服务器的，必须联网才能工作；
分布式版本控制系统：每个人的电脑都是一个完整的版本库，工作的时候不需要联网；
分布式的安全性高，一个人的损坏了可以从其他人那里拷贝，而集中式的，中央服务挂掉，所有人都干不了活

##操作
前提安装了git
```
git --version
//git version 1.7.11.msysgit.1
```
#####1.本地创建版本库
```
mkdir git-operate
cd git-operate
```
#####2.把目录变成git管理的仓库
```
git init
```
#####3.新建readme.txt文件添加到git仓库
```
git add readme.txt  //执行上面的命令，没有任何显示，这就对了，Unix的哲学是“没有消息就是好消息”，说明添加成功。
```
#####4.用git commit 提交到仓库
```
git commit -m "add new file"
[master (root-commit) c0b6535] add new file
 1 file changed, 2 insertions(+)
 create mode 100644 readme.txt
```
#####5.通常我们会修改已有的文件，用git status查看当前仓库文件的状态
```
$ git status
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#       modified:   readme.txt
#
no changes added to commit (use "git add" and/or "git commit -a")
```
#####6.git status告诉我们文件的状态，现在看看修改的内容
```
$ git diff readme.txt
diff --git a/readme.txt b/readme.txt
index 013b5bc..d8036c1 100644
--- a/readme.txt
+++ b/readme.txt
@@ -1,2 +1,2 @@
-Git is a distributed version control system.
+Git is a version control system.
 Git is free software.
\ No newline at end of file
```
#####7.提交修改之后的文件（提交修改和提交新文件是一样的两步：git add、git commit）
```
$ git add readme.txt

$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#       modified:   readme.txt

$ git commit -m "modify file"
[master ee01550] modify file
 1 file changed, 1 insertion(+), 1 deletion(-)

$ git status
# On branch master
nothing to commit (working directory clean)
```
#####8.使用git status命令随时掌握工作区的状态

#####9.将本地版本库推到github上

首先：github上建ssk key参考链接：https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001374385852170d9c7adf13c30429b9660d0eb689dd43a000

然后：在github上新建一个和本地同名的仓库，参考链接：
https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013752340242354807e192f02a44359908df8a5643103a000
![](https://upload-images.jianshu.io/upload_images/5541401-39adc1a02cbcb868.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
最后：根据提示将本地仓库和github仓库关联
```
$ git remote add origin git@github.com:mxcz213/git-operate.git

$ git push -u origin master
Counting objects: 7, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (7/7), 609 bytes, done.
Total 7 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), done.
To git@github.com:mxcz213/git-operate.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.

//把本地库的内容推送到远程，用git push命令，实际上是把当前分支master推送到远程。
//由于远程库是空的，我们第一次推送master分支时，加上了-u参数，
//Git不但会把本地的master分支内容推送的远程新的master分支，
//还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。
//以后再提交远程仓库就是git push origin master
```
#####10.克隆一个远程仓库
```
git clone git@github.com:mxcz213/git-operate.git
```
#####11.多次修改提交之后，想改回某一个版本，版本回退，查看提交日志
```
git log    //显示从最近到最远的提交日志
```
![](https://upload-images.jianshu.io/upload_images/5541401-7e505696cd334f23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
$ git log --pretty=oneline        //单行输出log信息
c0160df418d31c3b4d3792c814cbad2b90b4b613 commit again
1852c62687999bb837ff09831c9898e5faede408 modify file
ee01550aee99f47270cb7c8b9bf08e9ca6431bde modify file
c0b6535d48040e1a26d4cb9cbc1bbef13121b62d add new file
```
把readme.txt回退到上一个版本modify file
在git中HEAD表示当前版本，上一个版本是HEAD^,上上一个版本是HEAD^^，往上很多版本比如100个就是HEAD~100,使用git reset命令回退版本
```
$ git reset --hard HEAD^
HEAD is now at 1852c62 modify file

$ cat readme.txt
Git is a distributed version control system.
Git is free software. 
//文件被还原了
```
```
//通过commit id来回退版本，缩写id，git自己会去找
$ git log --pretty=oneline
1852c62687999bb837ff09831c9898e5faede408 modify file
ee01550aee99f47270cb7c8b9bf08e9ca6431bde modify file
c0b6535d48040e1a26d4cb9cbc1bbef13121b62d add new file

 $ git reset --hard ee01
HEAD is now at ee01550 modify file

$ git log --pretty=oneline
ee01550aee99f47270cb7c8b9bf08e9ca6431bde modify file
c0b6535d48040e1a26d4cb9cbc1bbef13121b62d add new file

$ git reflog     //记录每一次的命令
ee01550 HEAD@{0}: reset: moving to ee01
1852c62 HEAD@{1}: reset: moving to HEAD^
c0160df HEAD@{2}: commit: commit again
1852c62 HEAD@{3}: commit: modify file
ee01550 HEAD@{4}: commit: modify file
c0b6535 HEAD@{5}: commit (initial): add new file
```
##重点--分支管理
master分支只来提交，每次有新功能，每个人都可以建自己的分支，在分支上开发，完成之后合并到master分支，防止代码丢失或者影响到别人的开发进度。
#####
1.创建`dev`分支,并切换到`dev`分支
2.查看当前分支
3.修改`readme.txt`文件内容：`Creating a new branch is quick.`
4.查看当前dev分支里readme.txt文件内容
5.切换并查看master分支里readme.txt文件内容
6.合并dev分支的内容到master分支
```
mxcz@ITA-1401-0047 /E/workCode/git-operate (master)
$ git checkout -b dev      //创建dev分支并切换到dev分支
Switched to a new branch 'dev'

mxcz@ITA-1401-0047 /E/workCode/git-operate (dev)
$ git branch    //查看分支
* dev
  master

//修改dev分支里面readme.txt文件的内容为：`Creating a new branch is quick.`然后提交
mxcz@ITA-1401-0047 /E/workCode/git-operate (dev)
$ git add readme.txt

mxcz@ITA-1401-0047 /E/workCode/git-operate (dev)
$ git commit -m "branch test"
[dev e0d6643] branch test
 1 file changed, 1 insertion(+), 2 deletions(-)

mxcz@ITA-1401-0047 /E/workCode/git-operate (dev)
$ git checkout master    //切换分支到master
Switched to branch 'master'
Your branch is behind 'origin/master' by 1 commit, and can be fast-forwarded.

mxcz@ITA-1401-0047 /E/workCode/git-operate (master)
$ cat readme.txt
Git is a version control system.
Git is free software.
mxcz@ITA-1401-0047 /E/workCode/git-operate (master)
$ git checkout dev
Switched to branch 'dev'

mxcz@ITA-1401-0047 /E/workCode/git-operate (dev)
$ cat readme.txt
Creating a new branch is quick.

mxcz@ITA-1401-0047 /E/workCode/git-operate (dev)
$ git checkout master
Switched to branch 'master'
Your branch is behind 'origin/master' by 1 commit, and can be fast-forwarded.

mxcz@ITA-1401-0047 /E/workCode/git-operate (master)
$ git merge dev      //将dev分支合并到master分支（这种合并分支不会保存分支操作记录）
Updating ee01550..e0d6643
Fast-forward
 readme.txt | 3 +--
 1 file changed, 1 insertion(+), 2 deletions(-)

mxcz@ITA-1401-0047 /E/workCode/git-operate (master)
$ cat readme.txt    //合并成功，文件被修改
Creating a new branch is quick.

mxcz@ITA-1401-0047 /E/workCode/git-operate (master)
$ git branch -d dev     //删除dev分支
Deleted branch dev (was e0d6643).

mxcz@ITA-1401-0047 /E/workCode/git-operate (master)
$ git branch    //查看分支只剩master分支了
* master
```
#####保留分支记录的操作`git merge -m "merge with --no-ff" bug-101` 
```
mxcz@ITA-1401-0047 /E/workCode/git-operate (master)
$ git checkout -b bug-101
Switched to a new branch 'bug-101'

mxcz@ITA-1401-0047 /E/workCode/git-operate (bug-101)
$ cat readme.txt
Creating a new branch is quick.
create a bug-101 branch
mxcz@ITA-1401-0047 /E/workCode/git-operate (bug-101)
$ git add readme.txt

mxcz@ITA-1401-0047 /E/workCode/git-operate (bug-101)
$ git commit -m "create bug-101 branch"
[bug-101 5fc13df] create bug-101 branch
 1 file changed, 1 insertion(+)

mxcz@ITA-1401-0047 /E/workCode/git-operate (bug-101)
$ git branch
* bug-101
  master

mxcz@ITA-1401-0047 /E/workCode/git-operate (bug-101)
$ git checkout master
Switched to branch 'master'
Your branch and 'origin/master' have diverged,
and have 1 and 1 different commit each, respectively.

mxcz@ITA-1401-0047 /E/workCode/git-operate (master)
$ git merge --no-ff -m "merge with no-ff" bug-101
Merge made by the 'recursive' strategy.
 readme.txt | 1 +
 1 file changed, 1 insertion(+)

mxcz@ITA-1401-0047 /E/workCode/git-operate (master)
$ git log --graph --pretty=oneline --abbrev-commit
*   02bbf1a merge with no-ff
|\
| * 5fc13df create bug-101 branch
|/
* e0d6643 branch test
* ee01550 modify file
* c0b6535 add new file
```
此时的分支信息就有保留，使用`--no-ff`，这时master就不再是一条直线了，分叉了。
但是git可以通过`rebase`将`变基`成为一条干净的线；

######先解决冲突
```
mxcz@ITA-1401-0047 /E/workCode/git-operate (master)
$ git push origin master
To git@github.com:mxcz213/git-operate.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'git@github.com:mxcz213/git-operate.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Merge the remote changes (e.g. 'git pull')
hint: before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.

mxcz@ITA-1401-0047 /E/workCode/git-operate (master)
$ git pull
Auto-merging readme.txt
CONFLICT (content): Merge conflict in readme.txt
Automatic merge failed; fix conflicts and then commit the result.

mxcz@ITA-1401-0047 /E/workCode/git-operate (master|MERGING)
$ cat readme.txt
<<<<<<< HEAD
Creating a new branch is quick.
create a bug-101 branch
=======
Git is a distributed version control system.
Git is free software.
>>>>>>> 1852c62687999bb837ff09831c9898e5faede408
```
打开readme.txt文件，手动删掉冲突`<<<<<<<=======>>>>>>>`的部分，然后提交
```
mxcz@ITA-1401-0047 /E/workCode/git-operate (master)
$ git push origin master
Counting objects: 14, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (10/10), 994 bytes, done.
Total 10 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), done.
To git@github.com:mxcz213/git-operate.git
   1852c62..6ac178c  master -> master

mxcz@ITA-1401-0047 /E/workCode/git-operate (master)
$ git log --graph --pretty=oneline --abbrev-commit
*   6ac178c fixed conflict
|\
| * 1852c62 modify file
* |   02bbf1a merge with no-ff
|\ \
| * | 5fc13df create bug-101 branch
|/ /
* | e0d6643 branch test
|/
* ee01550 modify file
* c0b6535 add new file
```
#####如果commit的注释写错了，要修改掉，怎么办？下面来操作修改最后一次提交的注释`change file`改成`change file xxx`
```
git log
commit 9219151d7352b0561ae2d1d5c5c20a4c6dfbd4d1
Author: mxcz<chenjuanhe@pptv.com>
Date:   Fri Mar 15 13:33:54 2019 +0800

    change file

commit 6ac178c4d6672ac7de86ed9f7775da908cd02668
Merge: 02bbf1a 1852c62
Author: mxcz<chenjuanhe@pptv.com>
Date:   Thu Mar 14 17:17:02 2019 +0800

    fixed conflict
```
```
$ git commit --amend   //进入修改commit里
```
![](https://upload-images.jianshu.io/upload_images/5541401-2ab2184c178a770c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
改成`change file xxx`，然后按Esc，再按:wq保存退出编辑：
```
mxcz@ITA-1401-0047 /E/workCode/git-operate (master|REBASE-i)
$ git commit --amend
[master 464ab48] change file xxx
 1 file changed, 5 deletions(-)
```
然后强制推到服务器
```
$ git push -f
Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 294 bytes, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@github.com:mxcz213/git-operate.git
 + 9219151...464ab48 master -> master (forced update)
```
此时再查看`git log`提交历史，`commit`的`message`已经被修改过来
```
$ git log
commit 464ab488219208a9e30bcd5b7cb6e5b37c7a43a3
Author: mcxz<chenjuanhe@pptv.com>
Date:   Fri Mar 15 13:33:54 2019 +0800

    change file xxx

commit 6ac178c4d6672ac7de86ed9f7775da908cd02668
Merge: 02bbf1a 1852c62
Author: mxcz<chenjuanhe@pptv.com>
Date:   Thu Mar 14 17:17:02 2019 +0800

    fixed conflict
```
#####`git`打标签
```
$ git tag test_20190315_html

$ git push origin test_20190315_html
Total 0 (delta 0), reused 0 (delta 0)
To git@github.com:mxcz213/git-operate.git
 * [new tag]         test_20190315_html -> test_20190315_html

$ git tag
test_20190315_html
```
![](https://upload-images.jianshu.io/upload_images/5541401-911537487ec48180.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##总结
#####添加文件
git add .     将所有文件添加到暂存区


#####分支管理
Git鼓励大量使用分支：
查看分支：git branch
切换分支：git checkout <branch name>
创建本地分支：git branch <branch name>
创建+切换本地分支：git checkout -b <branch name>
将新建的分支推到远程：git push origin <branch name>

合并某分支到当前分支：git merge <branch name>    （这种合并分支不会保存分支操作记录）

强制删除本地分支：git branch -D <branch name>
删除远程分支：git push origin  --delete <branch name>

#####标签
git tag [tag name]    #给当前分支的最后一次commit打上标签
git tag [tag name] [commit id]    #给指定的commit打上标签
git tag -a [tag name] -m "[comment]" [commit id]    #创建有说明的标签
git tag    #显示所有标签
git show [tag name]    #显示指定标签的提交信息
git tag -d [tag name]    #删除标签
git push origin [tag name]    #推送指定标签到远程
git push origin --tags    #推送所有本地标签到远程

参考：https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000
           https://www.jianshu.com/p/c4e66d70e858
           https://www.jianshu.com/p/54cd784fc9aa
           

