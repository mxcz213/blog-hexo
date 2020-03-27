---
title: git-rebase-操作总结（合并git-commit的message）
---
第一步：新建一个干净的仓库，做五次修改提交，不要建分支，只做修改提交推送到远程

第二步：想要修改某一次的`commit` 的`message`，比如把`hello git`改成`1`
```
$ git log
commit 6eb562ed87eb5f763bba96eb0eef5e795d8acbaa
Author: chenjuanhe <chenjuanhe@pptv.com>
Date:   Fri Mar 15 15:33:27 2019 +0800

    5

commit 76ea4563339fda46a895c8ccf5fdf8be9de98210
Author: chenjuanhe <chenjuanhe@pptv.com>
Date:   Fri Mar 15 15:32:59 2019 +0800

    4

commit 073583226ceb34ea4f3a144d23e2cc0c4186269a
Author: chenjuanhe <chenjuanhe@pptv.com>
Date:   Fri Mar 15 15:32:38 2019 +0800

    3

commit 99d522288e7e28694b477021ca6556b2ca7ce61f
Author: chenjuanhe <chenjuanhe@pptv.com>
Date:   Fri Mar 15 15:31:44 2019 +0800

    2

commit 43ca7897fa5a1bb56b95729338db6fced7be69c0
Author: chenjuanhe <chenjuanhe@pptv.com>
Date:   Fri Mar 15 15:31:09 2019 +0800

    hello git

commit 8e455b2861b5b28c907f76b080bb0394e461495b
Author: mxcz213 <496182124@qq.com>
Date:   Fri Mar 15 15:29:15 2019 +0800

    Initial commit
```
复制这条`meaasge`的 `hello git`,复制`commitId`：`8e455b2861b5b28c907f76b080bb0394e461495b`，
使用`git rebase -i 8e455b2861b5b28c907f76b080bb0394e461495b`命令
```
$ git rebase -i 8e455b2861b5b28c907f76b080bb0394e461495b
```
出现这个画面：
![](https://upload-images.jianshu.io/upload_images/5541401-b24295102a62220f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

按`i`将`vim`切到插入状态，修改`pick`成`edit`,最后按`Esc`退出键，再按`:wq`保存修改，按`回车键Enter`结束
```
$ git rebase -i 8e455b2861b5b28c907f76b080bb0394e461495b
Stopped at 30432ba... hello git
You can amend the commit now, with

        git commit --amend

Once you are satisfied with your changes, run

        git rebase --continue
```
现在是修改状态就可以修改提交的message了,使用命令`git commit --amend`
```
mxcz@ITA-1401-0047 /E/workCode/git-exercise-2 (master|REBASE-i)
$ git commit --amend
[detached HEAD ea2c59f] 1
 1 file changed, 1 insertion(+)
 create mode 100644 operate_git.txt
```
![](https://upload-images.jianshu.io/upload_images/5541401-aa4376cd279df05e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后按照提示的使用命令`git rebase --continue`
```
mxcz@ITA-1401-0047 /E/workCode/git-exercise-2 (master|REBASE-i)
$ git rebase --continue
Successfully rebased and updated refs/heads/master.
```
最后使用`git push -f`提交到远程
```
mxcz@ITA-1401-0047 /E/workCode/git-exercise-2 (master)
$ git push -f
Counting objects: 16, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (10/10), done.
Writing objects: 100% (15/15), 1.33 KiB, done.
Total 15 (delta 0), reused 10 (delta 0)
To git@github.com:mxcz213/git-exercise-2.git
 + 6eb562e...c210399 master -> master (forced update)
```
在查看git log,文件提交的message已经被修改了。
![](https://upload-images.jianshu.io/upload_images/5541401-cdbec9b0323acf8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
期间如果出现任何错误都使用`git rebase --abort`重来

####合并message
![](https://upload-images.jianshu.io/upload_images/5541401-388c6135ed7f331f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
同样是上面的操作，将`2、3`的提交合并到一起，使用`2`的`commit id`
```
$ git rebase -i ea2c59fb124d821586e9ec55231e746dd781d488
[detached HEAD e015cb4] 23
 1 file changed, 3 insertions(+), 1 deletion(-)
Successfully rebased and updated refs/heads/master.
```
![](https://upload-images.jianshu.io/upload_images/5541401-47cad3231359456a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
只把`3`对应的`pick`改成`s`，然后保存新开了一个`vim`，删除`2/、3`注释，改成`23`，
![](https://upload-images.jianshu.io/upload_images/5541401-80009db61e4fc12e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
最后强制`push`到服务端：
```
$ git push -f
Counting objects: 11, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (9/9), 844 bytes, done.
Total 9 (delta 0), reused 6 (delta 0)
To git@github.com:mxcz213/git-exercise-2.git
 + 8116add...ddb9a50 master -> master (forced update)
```
####再查看`git log，2、3`的提交合并到`2`里面了，`commit id变成2`的了，注意：`4和5的commit id`也被改变了
![](https://upload-images.jianshu.io/upload_images/5541401-278a8c30cec3941d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####注意：
这里使用`git push`是不行的，
必须加强制`push`标志位`-f`

##总结命令

git rebase -i [commit id]
git commit --amend
git rebase --continue
git push -f

参考：https://www.jianshu.com/p/54cd784fc9aa





