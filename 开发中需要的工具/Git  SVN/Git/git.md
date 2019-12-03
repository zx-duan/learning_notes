#### GIT语法

```cmd
//添加项目到缓存中
$ git add .


//设置git的账户名
$ git config --global user.name "zx_duan"

//设置git的邮箱
$ git config --global user.email "489305739@qq.com"

//初始化项目
$ git init

//绑定码云上面的地址
$ git remote add origin https://gitee.com/zx_duan/crm.git

//将项目推送到码云上面去
$ git push -u origin master



```







#### 操作执行步骤

经理方 : 

创建项目→初始化 `本地` 项目→项目上传`远程`   -

----->后续 : 收集组员每日提交的代码 , 然后整合 ----> 添加缓存 → 提交项目

> 提交项目使用的分支是主分支  master
>
> 提交或者修改之前都需要先备份一下







组员方 : 

下载托管中心的代码(最新) ----->拷贝一份------> 自定义一个支线用来编写代码 ------->检查bug无误后提交代码到Master中 ----> 然后使用Master提交代码 , 如果检查没有问题就发布到远程上面

>  一定不要使用主分支来进行开发
>
> 在新创建或者修改代码后一定要记得拷贝一份原来的



```cmd

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo
//创建账户的名称
Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo
$ git config --global user.name "zhangxin"
//创建账户的邮箱
Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo
$ git config --global user.email "490305739@qq.com"
Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo
//初始化本地仓库
$ git init
Initialized empty Git repository in G:/学习资料/demo/.git/
Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)

//添加文件到缓存区
$ git add readme.txT
Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)

//提交文件到本地仓库
$ git commit -m "这是一个读取文件很重要"
[master (root-commit) 7ac66a7] #这是一个读取文件很重要
 1 file changed, 2 insertions(+)
 create mode 100644 readme.txt

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)

//查看提交的状态
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)


//获取未add时文件与已经提交文件的区别
$ git diff
diff --git a/readme.txt b/readme.txt
index d8036c1..013b5bc 100644
--- a/readme.txt
+++ b/readme.txt
@@ -1,2 +1,2 @@
-Git is a version control system.
+Git is a distributed version control system.
 Git is free software.
\ No newline at end of file
Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)


//获取文件更改的内容
$ git diff readme.txt
diff --git a/readme.txt b/readme.txt
index d8036c1..013b5bc 100644
--- a/readme.txt
+++ b/readme.txt
@@ -1,2 +1,2 @@
-Git is a version control system.
+Git is a distributed version control system.
 Git is free software.
\ No newline at end of file

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)
$ git add readme.txt

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   readme.txt


Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)
$ git commit -m "add distributed"
[master 426ce2a] add distributed
 1 file changed, 1 insertion(+), 1 deletion(-)

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)
$ git status
On branch master
nothing to commit, working directory clean

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)
$ git add readme.txt

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)
$ git commit -m readme.txt
[master d825fc3] readme.txt
 1 file changed, 1 insertion(+), 1 deletion(-)

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)
$ git status
On branch master
nothing to commit, working directory clean

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)
$ git diff

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)

//获取修改后的日志文件
$ git log
commit d825fc334b67226c30bae9f89476b2c73cb71d92
Author: zhangxin <490305739@qq.com>
Date:   Sun Nov 3 21:47:49 2019 +0800

    readme.txt

commit 426ce2afef91f321c479091855d9f2197920901f
Author: zhangxin <490305739@qq.com>
Date:   Sun Nov 3 21:46:34 2019 +0800

    add distributed

commit 7ac66a7756106cc1a2383a7abbcc27604232e8d8
Author: zhangxin <490305739@qq.com>
Date:   Sun Nov 3 21:41:40 2019 +0800

    这是一个读取文件很重要

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)

//获取简短版的日志文件
$ git log --pretty=oneline
d825fc334b67226c30bae9f89476b2c73cb71d92 readme.txt
426ce2afef91f321c479091855d9f2197920901f add distributed
7ac66a7756106cc1a2383a7abbcc27604232e8d8 这是一个读取文件很重要

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)

//回退到上一个版本
$ git reset --hard HEAD^
HEAD is now at 426ce2a add distributed

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)
$ git diff

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)
$ git log
commit 426ce2afef91f321c479091855d9f2197920901f
Author: zhangxin <490305739@qq.com>
Date:   Sun Nov 3 21:46:34 2019 +0800

    add distributed

commit 7ac66a7756106cc1a2383a7abbcc27604232e8d8
Author: zhangxin <490305739@qq.com>
Date:   Sun Nov 3 21:41:40 2019 +0800

    这是一个读取文件很重要

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)
$ cat readme.txt
Git is a distributed version control system.
Git is free software.
Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)

//修改文件的版本
$ git reset --hard d825f
HEAD is now at d825fc3 readme.txt

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)
//获取文本的内容
$ cat readme.txt
Git is a distributed version control system.
Git is free software distributed under the GPL.
Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)

//获取使用git的操作命令
$ git reflog
d825fc3 HEAD@{0}: reset: moving to d825f
426ce2a HEAD@{1}: reset: moving to HEAD^
d825fc3 HEAD@{2}: commit: readme.txt
426ce2a HEAD@{3}: commit: add distributed
7ac66a7 HEAD@{4}: commit (initial): 这是一个读取文件很重要

Admin@DESKTOP-O68KVFO MINGW64 /g/学习资料/demo (master)
```









注意事项











