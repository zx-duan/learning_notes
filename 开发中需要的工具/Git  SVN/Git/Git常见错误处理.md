#### 								仓库路径已经存在

![img](https://img2018.cnblogs.com/blog/658377/201905/658377-20190522094101029-1897696483.png)

> 解决办法如下：
>
> ​    1、先输入$ git remote rm origin
>
> ​    2、再输入$ git remote add origin***.git 就不会报错了！
>
> ​    3、如果输入$ git remote rm origin 还是报错的话，error: Could not remove config section 'remote.origin'. 我们需要修改gitconfig文件的内容
>
> ​    4、找到你的github的安装路径，我的是C:\Users\ASUS\AppData\Local\GitHub\PortableGit_ca477551eeb4aea0e4ae9fcd3358bd96720bb5c8\etc
>
> ​    5、找到一个名为gitconfig的文件，打开它把里面的[remote "origin"]那一行删掉就好了！

