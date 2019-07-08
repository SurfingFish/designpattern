#      Git实现从本地添加项目到远程仓库



1、首先，要有git的账号，[点击查看怎么注册？](http://blog.csdn.net/u011043843/article/details/30356071)

2、注册成功之后，登陆GitHub，然后，在右上角找到“Create a new repo”按钮，创建一个新的仓库：



在Repository name填入仓库名，其他保持默认设置，点击“Create repository”按钮，就成功地创建了一个新的Git仓库：



3、在Git bash下创建并初始化本地仓库

```
mkdir 仓库名 -- 创建本地仓库
```



```
cd 仓库名    --进入本地仓库
```



```
pwd     --查看本地仓库路径
```



```
git init --初始化本地仓库
```



将需要上传的本地文件放置到文件名为仓库名的路径下



```
git remote add origin git@github.com:SurFingFish/designpattern.git  --将远程仓库与本地库关联
```

SurfingFish： 用户名   

designpattern 远程仓库名



```
git add 文件夹名    --将文件夹添加到暂存区
```



```
git commit -m 'xxx'
```

-m 之后的部分相当于注释



```
git push -u origin master
```

当远程仓库为空的时候，第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送到远程新的mster分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或拉取时可以简化命令。    

master 代表是哪个分支



