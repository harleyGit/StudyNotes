


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
- [**分支种类**](#分支种类)
- [**Git安装**](#Git安装)
- [Gitee配置](#Gitee配置)
- 
- **错误解决**
	-  [LibreSSL SSL_connect：443](#LibreSSLSSL_connect443) 
	-  [游离态Head解决](#游离态Head解决)
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

> **1.拉取指定分支代码**

> git clone -b dev https://github.com/xxx/xxx.git


<br/>


> **2.初始化版本库**

> git init


<br/>

> **3.添加文件到版本库（只是添加到缓存区） .代表添加文件夹下所有文件**

> git add .


<br/>


> **4.把添加的文件提交到版本库，并填写提交备注**

> git commit -m "提交记录"

<br/>

> **5.把本地库与远程库关联**

> git remote add origin 你的远程库地址


<br/>

> **6.推送代码到远程仓库（ 第一次推送时）**

> git push -u origin master




<br/>

> **7.推送代码到远程仓库（第一次推送后，直接使用该命令即可推送修改）**

> git push origin master 


<br/>

> **8.拉取远程代码到本地**



当远程是在master分支，本地开发也是在分支，可以使用下面命令：
> [git pull --rebase ](https://www.jianshu.com/p/b0a4d0c1e66f)






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

**1⃣️创建了Develop分支，如下：**

![Develop 分支效果图](https://upload-images.jianshu.io/upload_images/2959789-8d8fad8d521a0d61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>



- **`2⃣️在develop 分支建立 release分支`**

![建立release 发布版本](https://upload-images.jianshu.io/upload_images/2959789-3994f8abed1cf40f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![建立 release 分支的 1.3.7发布版本](https://upload-images.jianshu.io/upload_images/2959789-6250fec50fd74a80.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



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



># <h1 id='分支种类'>[分支种类](https://juejin.cn/post/6844903634006720526)</h1>


![分支合入、提交、检出关系图](https://upload-images.jianshu.io/upload_images/2959789-7993f537c48f6cd8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**`master 分支:`** 
-   用于部署生产环境的分支，要确保master分支稳定性；
-   一般由`develop`、`release` 以及`hotfix`分支合并，任何时间都不能直接修改代码;


**`develop 开发分支:`**
-  开发分支，始终保持最新完成以及bug修复后的代码;
-  一般开发的新功能时，`feature` 分支都是基于 `develop` 分支下创建的;

**`feature 功能分支:`** 
-  开发新功能时，`以develop` 为基础创建 `feature` 分支
-  分支命名: `feature/` 开头的为特性分支， 命名规则: `feature/user_module、 feature/cart_module`;

**`release 发布分支:`**  
 -  发布分支为预上线分支，发布提测阶段；

&emsp;  当有一组`feature`开发完成，首先会合并到`develop`分支，进入提测时，会创建`release`分支。如果测试过程中若存在bug需要修复，则直接由开发者在release分支修复并提交。

&emsp;  当测试完成之后，合并`release`分支到`master`和`develop`分支，此时`master`为最新代码，用作上线。


**`hotfix 补丁分支: `**
- 分支命名: hotfix/ 开头的为修复分支，它的命名规则与 feature 分支类似;
- 线上出现紧急问题时，需要及时修复，以master分支为基线，创建hotfix分支，修复完成后，需要合并到master分支和develop分支;

**`version1.0 版本标签分支:`**  

&emsp;  Branch是GitFolw的核心。主要分为两大类 Main Branchs 和 Supporting branches, 其中 Main Branchs 中又包含了 Master 和 Develop，而 Supporting branches 中包含了 Feature 、Release、Hotfix 以及其他自定义分支。

<br/>
**`Master`**
描述：
&emsp;  master分支上存放的是最稳定的正式版的代码，并且该分支的代码应该是随时可在开发环境中使用的代码（Production Ready state）。当一个版本开发完毕后，产生了一份新的稳定的可供发布的代码时，master分支上的代码要被更新，同时，每一次更新，都需要在master上打上对应的版本号(tag)。

生成及销毁：
&emsp; 任何人不允许在master上进行代码的直接提交，只接受合入，Master上的代码必须是要从经过多轮测试且已经发布一段时间(根据DAU以及项目实际情况来定，个人建议K歌国际版可以定为一周)且线上已经稳定的 release 分支合并进去，然后在Master 上生成tag(通常就是对应的版本号)



<br/>


**`Develop`**

描述：
&emsp;  develop分支是保存当前最新版本开发成果的分支。该分支上的代码允许有BUG，但是必须保证编译通过，且该分支可以作为每天夜间测试的分支(如果有夜间测试的话)所以该分支也叫做Nightly build。当develop分支上的代码已实现了软件需求说明书中所有的功能(必须经过开发自测，但是不必经过QA)且相对稳定时候，就可以基于此分支来拉出新的release分支交付QA进行测试。


生成及销毁:
&emsp;  Develop分支是由一个人(通常是Team Leader)从Master中拉出，任何人不得在Develop上进行代码提交，只接受合入。Develop上所有代码一定都是由 Supporting branches 中的Branch合并进来，且合入Develop的分支必须保证功能完整，可以独立运行，可允许包含一些BUG(但是最好经过自测，不要有太大或者太明显的BUG，比如一启动就crash之类的)。


![Develop分支提交图](https://upload-images.jianshu.io/upload_images/2959789-e6309ad1180ae0ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




<br/>




**`Feature`**

描述：
&emsp;  Feature分支通常叫做功能分支，也可以叫做个人分支，一般命名为 feature/XXXX,该分支就是每一个开发人员进行开发的分支，比如做一些功能、需求之类的东西，这个分支上的代码变更最终合并回develop分支或者干脆被抛弃掉（例如实验性且效果不好的代码变更）。一般而言，feature分支代码可以保存在开发者自己的代码库中而不强制提交到主代码库里。


生成及销毁:
&emsp;  每个开发者从通常会Develop分支中拉取自己的feature，且开发者可以随意的在自己的feature上进行操作 包括但不限于 提交、回滚、删除。如果最终需要合并入develop那就要保证功能的完整性以及代码的稳定新，比如我在feature上做了3个需求但是由于时间关系我只做了两个，那也可以将feature合并入develop，然后剩下的那一个需求等有时间了再去feature上做完之后再合入develop。所以这里说的功能的完整性并不是值得要做完所有的功能，而是要保证你所要做的所有需求中的某一个或者某几个功能已经做完，不允许把做到一半的功合并入develop。合并入develop尽量上删除远端的feature分支，本地的feature可以视情况而取舍。


![Feature 和 Develop关系图](https://upload-images.jianshu.io/upload_images/2959789-1e729c7c5fbdba99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>

**`Release`**

描述：
&emsp;  Release分支通常叫做发布分支，也可以叫做测试-发布分支，一般命名为 Release/1.2.3（后面是版本号）,该分支是为测试-发布新的产品版本而开辟的。因为包含测试流程，所以在这个分支上的代码允许做小的缺陷修正、准备发布版本所需的各项说明信息（版本号、发布时间、编译时间等等）。通过在release分支上进行这些工作可以让develop分支空闲出来以接受新的feature分支上的代码提交，进入新的软件开发迭代周期。注意：该分支上的代码一定是可编译可运行的，允许包含小BUG


生成及销毁:
&emsp;  当develop分支上的代码已经包含了该版本所有即将发布的功能和需求，并且已通过自测且已基本稳定，我们就可以考虑准备基于develop拉取release分支了。而所有在当前即将发布的版本之外的业务需求一定要确保不能混到release分支之内（避免由此引入一些不可控的系统缺陷）。成功的派生了release分支之后，develop分支就可以为“下一个版本”服务了。所谓的“下一个版本”是在当前即将发布的版本之后发布的版本。开发人员可以在此分支上修改BUG，进行提交、回滚等操作，但是与feature不同的是release分支是被多人操作的，不像feature，所以一定要小心避免冲突。当现在QA测试没有问题，便从release上发布上线，且经过一段时间的验证没有问题后合入master，并且删除release分支，其实根据release分支的特性我们可以使用Git Hook触发软件自动测试以及生产环境代码的自动更新工作。这些自动化操作将有利于减少新代码发布之后的一些事务性工作。



<br/>



**`Hotfix`**

描述：
&emsp;  Hotfix叫热修复分支，除了是计划外创建的以外，hotfix分支与release分支十分相似，当已经发布的版本（Master上代码）遇到了异常情况或者发现了严重到必须立即修复的软件缺陷的时候，就需要从master分支上指定的tag版本拉取hotfix分支来组织代码的紧急修复工作。这样做的显而易见的好处是不会打断正在进行的develop分支的开发工作，能够让团队中负责新功能开发的人与负责代码紧急修复的人并行、独立的开展工作。


生成及销毁:
&emsp;  由Master上拉取，进行修复，负责修改BUG的同事可以进行提交及其它操作，后续的热修复测试也在此分支上进行。通过测试验证没问题后有一个人(通常为teamleader)合并入Master分支，且同时也要合并入Develop分支。

![Hotfix 分支关系图](https://upload-images.jianshu.io/upload_images/2959789-fc29934a4531b97d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




<br/>

***
<br/>




> <h1 id='Git安装'>Git安装</h1>

查看电脑是否安装Git，终端输入：

```
git
```

安装过则会输出：

![安装 Git 后的终端截图](https://upload-images.jianshu.io/upload_images/2959789-5f498c4eb71ee8ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>
<br/>

**`创建ssh key、配置git`**

**`①. 设置username和email（github每次commit都会记录他们）`**

```
//用户名
git config --global user.name "Harley"

//邮箱
git config --global user.email "harley@qq.com"
```

<br/>

**`②. 终端命令创建ssh key`**

```

ssh-keygen -t rsa -C "harelysoa@qq.com" 

Generating public/private rsa key pair.

Enter file in which to save the key (/Users/harleyhuang/.ssh/id_rsa): 

```

创建过的，这里我选**n**，没有创建过的，会要求确认路径和输入密码，我们这使用默认的一路回车就行。成功的话会在**~/**下生成.ssh文件夹，进去，打开id_rsa.pub，复制里面的key

```

Created directory '/Users/harleyhuang/.ssh'.

Enter passphrase (empty for no passphrase): 

Enter same passphrase again: 

```

要求输入管理员密码，然后会提示：

![输入管理员密码](https://upload-images.jianshu.io/upload_images/2959789-55f9618631bcdb44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


复制：`/Users/harleyhuang/.ssh/id_rsa.pub.` 获取公钥，打开` Finder`，组合键祭起： `Command + shift +G`，然后粘贴。打开` id_rsa.pub` 文件，复制里面的全部内容。

登录[GitHub]([https://github.com/settings/keys](https://github.com/settings/keys)),配置公钥：

![配置公钥](https://upload-images.jianshu.io/upload_images/2959789-fb34fa99856d0c36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


填写：

![填写标题和公钥](https://upload-images.jianshu.io/upload_images/2959789-ef4e6828e643bcc7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


完成后终端输入：

```
ssh -T git@github.com 
```

终端输出：

```

The authenticity of host 'github.com (52.74.223.119)' can't be established.

RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.

Are you sure you want to continue connecting (yes/no)? 
```
输入：yes

终端输出：

```
Warning: Permanently added 'github.com,52.74.223.119' (RSA) to the list of known hosts.

Enter passphrase for key '/Users/harleyhuang/.ssh/id_rsa': 

```
输入管理员密码，然后回车,终端输出：

```

Hi harleyGit! You've successfully authenticated, but GitHub does not provide shell access.
```

Git 安装完成!



<br/>

***
<br/>

> <h1 id='Gitee配置'>Gitee配置</h1>
**`问题：这是一个无效的源路径URL`**

原因：在[**码云**]()没有配置公钥导致的，只要配置公钥即可解决这个问题，亲测！

```

//打开终端，输入：
$ ssh-keygen -t rsa -C "自己的邮箱号.com"
//回车
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/用户名/.ssh/id_rsa): 
//遇到上面的代码，不要管，再点击回车
/Users/用户名/.ssh/id_rsa already exists.
Overwrite (y/n)?
//这个文件已存在，是否覆盖？输入：n 即可
n
//拷贝终端的文件路径：/Users/用户名/.ssh
//打开 Finder，Command + shift +G，用文本编辑打开 id_rsa.pub 这个文件
//复制里面的公钥；

```

前往[码云]()，打开设置，找到如下界面进行粘贴即可！
![粘贴公钥](https://upload-images.jianshu.io/upload_images/2959789-db13c19b3f55e023.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


[远程仓库以及代码管理工具使用](https://www.jianshu.com/p/3685cbf9ab47)



<br/>

***
<br/>

># <h1 id="GitBoo">[GitBoo](https://www.gitbook.com)</h1>


- **安装Node.js**

在[这里](https://nodejs.org/en/)下载[Node.js](https://nodejs.org/en/),进入下载页面后下载**`Current 区域稳定版`**。

![Node.js 官网下载页面](https://upload-images.jianshu.io/upload_images/2959789-523420b312e36c34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 点击下载的安装包

- 在终端输入

```
 npm -v
6.14.4
node -v
v13.12.0

```

说明安装成功。

<br/>

- **安装GitBook**

[GitBook 安装步骤](https://www.jianshu.com/p/421cc442f06c)

[sss](https://www.jianshu.com/p/421cc442f06c)




<br/>

***
<br/>

># <h1 id="错误解决">错误解决</h2>


<br/>


># <h2 id="LibreSSLSSL_connect443">LibreSSL SSL_connect：443</h2>

**`LibreSSL SSL_connect: SSL_ERROR_SYSCALL in connection to github.com:443`**

```
//查看git配置
git config --global --list  

// 发现其中有 http.https.XXXXXX.proxy 和 https.https.XXXXXX.proxy配置

//删除代理配置， 运行后，git恢复正常。
git config --global --unset http.https://github.com.proxy
git config --global --unset https.https://github.com.proxy


//
 git config --global --unset http.proxy
 git config --global --unset https.proxy
 
 networksetup -setv6off Wi-Fi

```



<br/>
<br/>

> <h2 id='游离态Head解决'>游离态Head解决</h2>

**detached HEAD(游离态的HEAD) 切换到其他分支，导致代码丢失**

 
**`HEAD  基础`**

>  git checkout 命令实际上是指修改HEAD文件的内容，让其指向不同的branch。

HEAD文件指向的branch就是当前branch 。 

一般来讲，HEAD的内容是指向staging（暂存区）的master文件的。

 ```
  ref: refs/heads/master
 ```

**` detached HEAD`**
如果让HEAD文件指向一个commit id(对一个HEAD进行一次代码或文件提交)，那就变成了detached HEAD。git checkout 可以达到这个效果，用下面的命令：

```
//laea8d9是最近的一次commit id，^指的是之前一次，因此上面的操作结果是让HEAD文件包含了倒数第二次提交的id.
git checkout 1aea8d9^
```
 
<br/> 
<br/>     
   
**问题解决**
 
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



