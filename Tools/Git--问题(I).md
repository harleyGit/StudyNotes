>#detached HEAD(游离态的HEAD) 切换到其他分支，导致代码丢失

## 科普 HEAD 知识
 
#`HEAD  基础`
>  git checkout 命令实际上是指修改HEAD文件的内容，让其指向不同的branch。
HEAD文件指向的branch就是当前branch 。 
一般来讲，HEAD的内容是指向staging（暂存区）的master文件的。
 ```
  ref: refs/heads/master
 ```

#` detached HEAD`
如果让HEAD文件指向一个commit id(对一个HEAD进行一次代码或文件提交)，那就变成了detached HEAD。git checkout 可以达到这个效果，用下面的命令：
```
//laea8d9是最近的一次commit id，^指的是之前一次，因此上面的操作结果是让HEAD文件包含了倒数第二次提交的id.
git checkout 1aea8d9^
```
 
<br/>  
***
<br/>     
   
##解决方法
 
![游离态的HEAD](https://upload-images.jianshu.io/upload_images/2959789-075021e91564fc38.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`解决办法：`
① 查找提交记录
```
$ git reflog
```
![在游离态的HEAD上的分支提交记录](https://upload-images.jianshu.io/upload_images/2959789-9f3c8d04c232b230.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


② 检出需要的提交记录
```
$ git checkout 408b379
```

③ 创建develop 分支，并切换到该develop 分支 
```
$ git checkout -b develop
```
![切换到develop分支](https://upload-images.jianshu.io/upload_images/2959789-fba37a26f0cc62c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

⑤ git checkout master 切换到master分支 
```
 $ git checkout master
```

⑥git merge develop 合并develop分支
```
$ git merge develop
```

