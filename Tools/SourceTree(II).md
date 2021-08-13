


> <h2 id=''></h2>
- [**指令工作流**](#指令工作流)
- [**工作流指令**](#工作流指令)
	- [新功能分支](#新功能分支)
	- [修复紧急bug](#修复紧急bug)
	- [dev合并到release](#dev合并到release)
	- [版本打Tag](#版本打Tag)
- [**SourceTree工作流**](#SourceTree工作流)
	- [新建分支](#新建分支)
	- [远程检出分支](#远程检出分支)
	- [删除分支](#删除分支)
	- [拉取](#拉取)
	- [合并分支](#合并分支)
	- [代码回滚](#代码回滚)
	- [远程回滚](#远程回滚)
	- [变基](#变基)
	- [打Tag](#打Tag)
- **`参考资料`**
	- [Git 分支阐述](https://www.cnblogs.com/hezhiying/p/9292314.html)
	- [sourceTree合并某次提交](https://www.jianshu.com/p/12da57330ca0)
	- [source tree进行rebase操作](https://www.jianshu.com/p/e54fd2ab8ce8)
	- [合并多次提交](https://github.com/zuopf769/how_to_use_git/blob/master/使用git%20rebase合并多次commit.md)





<br/>

***
<br/>

> <h1 id='指令工作流'>指令工作流</h1>




<br/>

***
<br/>

> <h1 id='工作流指令'>工作流指令</h1>

<br/>


> <h2 id='新功能分支'>新功能分支</h2>


```
///从dev建立特性分支
(dev)$: git checkout -b feature/xxx   

///开发        
(feature/xxx)$: blabla                         
(feature/xxx)$: git add xxx
(feature/xxx)$: git commit -m 'commit comment'

///把特性分支合并到dev
(dev)$: git merge feature/xxx --no-ff          
```

<br/>
<br/>

> <h2 id='修复紧急bug'>修复紧急bug</h2>


```
///从master建立hotfix分支
(master)$: git checkout -b hotfix/xxx   

///开发      
(hotfix/xxx)$: blabla                        
(hotfix/xxx)$: git add xxx
(hotfix/xxx)$: git commit -m 'commit comment'

/// 把hotfix分支合并到master，并上线到生产环境
(master)$: git merge hotfix/xxx --no-ff  

///把hotfix分支合并到dev，同步代码     
(dev)$: git merge hotfix/xxx --no-ff        
```


<br/>
<br/>

> <h2 id='dev合并到release'>dev合并到release</h2>

```
///把dev分支合并到release，然后在测试环境拉取并测试
(release)$: git merge dev --no-ff             
```


<br/>
<br/>

> <h2 id='版本打Tag'>版本打Tag</h2>


```
///把testing测试好的代码合并到master，运维人员操作
(master)$: git merge testing --no-ff   

///给版本命名，打Tag       
(master)$: git tag -a v0.1 -m '部署包版本名' 
```

<br/>

***
<br/>

> <h1 id='SourceTree工作流'>SourceTree工作流</h1>


<br/>

> <h2 id='新建分支'>新建分支</h2>


![创建 Develop 分支 步骤](https://upload-images.jianshu.io/upload_images/2959789-3acf23637634f80f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

创建了Develop分支，如下：

![Develop 分支效果图](https://upload-images.jianshu.io/upload_images/2959789-8d8fad8d521a0d61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
<br/>

**`Demo: 创建功能开发分支feature/Harley`**

![feature/Harley 步骤](https://upload-images.jianshu.io/upload_images/2959789-64cbc2a126a0eab7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

效果：
![feature 效果图](https://upload-images.jianshu.io/upload_images/2959789-30e37ed538ce34ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
<br/>


> <h2 id='远程检出分支'>**`远程检出分支`**</h2>




![远程检出 develop 分支](https://upload-images.jianshu.io/upload_images/2959789-218cb64820cddd6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![检出 develop 分支](https://upload-images.jianshu.io/upload_images/2959789-16157a449cc758a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


效果图：

![远程检出的 develop 分支](https://upload-images.jianshu.io/upload_images/2959789-18f855e632c3cb90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)







<br/>
<br/>


> <h2 id='删除分支'>删除分支</h2>

**`删除远程分支`**

![删除远程分支](https://upload-images.jianshu.io/upload_images/2959789-9a24128fdfd5fbfe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

&emsp;  这是删除远程的分支，本地分支还没有删除，需要进行下一步删除本地分支。

<br/>


**`删除本地分支`**
![删除 Develop 分支步骤图](https://upload-images.jianshu.io/upload_images/2959789-d268e5051128da72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
<br/>


> <h2 id='拉取'>拉取</h2>

&emsp; `master`分支的内容是最新的，`feature/Harley的分支`版本落后于master，从远程拉取内容合并到`feature/Harley的分支`上，如下图步骤：

![拉取 master 分支到 feature/Harley 分支上](https://upload-images.jianshu.io/upload_images/2959789-cda66587a121e177.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
<br/>


> <h2 id='合并分支'>合并分支</h2>



**`第一种方法`**

![合并分支](https://upload-images.jianshu.io/upload_images/2959789-8911ad0da3198a2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

**`第二种方法`**

![合并Huang 分支到 develop 分支](https://upload-images.jianshu.io/upload_images/2959789-cb50d674849ed6dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>
<br/>


> <h2 id='代码回滚'>代码回滚</h2>



**适用于的场景：**
- 提交错代码，想放弃刚刚提交的部分；
- 代码发生冲突，处理比较麻烦，为了代码安全，直接回滚到之前干净的代码。

**回滚分为`本地回滚`和`远程回滚`;**

<br/>

**`本地回滚`**，回滚已经提交的代码，但还未推送到远程仓库。

![选择回滚到哪次提交](https://upload-images.jianshu.io/upload_images/2959789-0eb73cdd32073704.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![选择模式进行回滚](https://upload-images.jianshu.io/upload_images/2959789-fa7a46296f80ce76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**使用模式：**
- `软合并：`保留上第二次提交的修改内容，就等第二次提交的【概述】了。
- `混合合并：`保留上第二次提交的修改内容，就等第二次提交的【概述】了(与软合并没啥区别)。
- `强行合并：`清除第二次提交的所有内容，第一次提交的【概述】也没有了，好像刚刚第一次的提交。

效果图：

![本地回滚完成](https://upload-images.jianshu.io/upload_images/2959789-035fac02454342ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

> <h2 id='远程回滚'>**`远程回滚`**</h2>



&emsp;  `SourceTree`默认是不提供这种操作的，因为存在风险。所以，回滚远程代码，一定要注意：
①. 想要放弃的代码，是所有开发成员都一致同意的；
②. 想要放弃的代码只是自己的，中间没有别人的提交记录，这可以直接回滚。
③. 这个操作过程中，提醒其他成员不要推送代码。

- Frist Stemp

![SourceTree 开启【允许强制推送】权限](https://upload-images.jianshu.io/upload_images/2959789-9d7986d4143aac6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- Second Stemp

![回滚到想提交的位置](https://upload-images.jianshu.io/upload_images/2959789-9074723aeb058c29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- Third Stemp

![强制推送](https://upload-images.jianshu.io/upload_images/2959789-521fb4bccf0c511e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

效果图：
![回到了第一次提交位置](https://upload-images.jianshu.io/upload_images/2959789-be1df4b96dae57d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
<br/>


> <h2 id='变基'>变基</h2>

![变基与合并的区别](https://upload-images.jianshu.io/upload_images/2959789-b9b68a2db95509cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**作用：**
- 变基到其他分支，不会生成新的提交节点；
- 压缩提交消息；

<br/>

**`①变基到其他分支`**将Harley 分支内容接到 develop

![将Harley 分支内容接到 develop](https://upload-images.jianshu.io/upload_images/2959789-f64daa40a1a56d34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


注意：在这里，`先要拉取，再推送`，否则会失败,下面是`推送，如下：`

![推送develop分支到远端](https://upload-images.jianshu.io/upload_images/2959789-7d4323e08d50fc6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




<br/>
<br/>


> <h2 id='打Tag'>打Tag</h2>


**功能:**
-  轻量级的：它其实是一个独立的分支,或者说是一个不可变的分支.指向特定提交对象的引用;
-  带附注的：实际上是存储在仓库中的一个独立对象，它有自身的校验和信息，包含着标签的名字，标签说明，标签本身也允许使用 GNU Privacy Guard (GPG) 来签署或验证,电子邮件地址和日期，一般我们都建议使用含附注型的标签，以便保留相关信息。

![添加Tag步骤](https://upload-images.jianshu.io/upload_images/2959789-5e7ac75a848d9e99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


也可以像下图那样打标签🏷️那样：
![工作流打标签流程](https://upload-images.jianshu.io/upload_images/2959789-f6848154eeae3ee1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


[Tag 的使用](https://www.jianshu.com/p/1bc7c948d019)请看这里。

- **`Tag 和 branch 的区别`:**
	-  tag 对应某次 commit, 是一个点，是不可移动的;
	-  branch 对应一系列 commit，是很多点连成的一根线，有一个HEAD 指针，是可以依靠 HEAD 指针移动的。

&emsp;  所以，两者的区别决定了使用方式，改动代码用 branch ,不改动只查看用 tag。

&emsp;  tag 和 branch 的相互配合使用，有时候起到非常方便的效果，例如 已经发布了 v1.0 v2.0 v3.0 三个版本，这个时候，我突然想不改现有代码的前提下，在 v2.0 的基础上加个新功能，作为 v4.0 发布。就可以 检出 v2.0 的代码作为一个 branch ，然后作为开发分支。


<br/>

***
<br/>


> <h1 id='Git工作流'>Git工作流</h1>



**`1⃣️在develop 分支建立 release分支`**
![建立release 发布版本](https://upload-images.jianshu.io/upload_images/2959789-3994f8abed1cf40f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![建立 release 分支的 1.3.7发布版本](https://upload-images.jianshu.io/upload_images/2959789-6250fec50fd74a80.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

