># 分支的分类


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
**`参考资料`**


