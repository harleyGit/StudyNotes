

- [**查看提交记录**](#查看提交记录)
-  [**提交**](#提交)
-  [**rebase变基**](#ebase变基)
	- [单分支合并多次commit](#单分支合并多次commit)
	- [一分支提交合并到另一分支上](#一分支提交合并到另一分支上)
- [**git branch**](#gitbranch)
- [**git checkout**](#gitcheckout)
- **[Git 简易指南](https://www.bootcss.com/p/git-guide/)**
- **[Git 参考手册](http://gitref.justjavac.com/index.html)**
- **[Pro Git(中文版)](https://gitee.com/progit/)**
- **[Git教程 廖雪峰](https://www.liaoxuefeng.com/wiki/896043488029600/896202815778784)**
- [Git&GitLab](https://blog.csdn.net/csfreebird/article/category/1224512)



<br/>

***
<br/>

># <h1 id="查看提交记录">查看提交记录</h1>

- 终端命令
  - `git log `:列出历史提交记录;

![记录查看](https://upload-images.jianshu.io/upload_images/2959789-d67dd6a2895bfa4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


  -  `git log --oneline`:查看历史记录的简洁的版本;
![简洁记录查看](https://upload-images.jianshu.io/upload_images/2959789-3199a538813e96d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>

***
<br/>


> <h1 id="提交">提交</h1>

**` 提交规范`**
- feat: 新功能
- fix: 修复问题
- docs: 修改文档
- style: 修改代码格式，不影响代码逻辑
- refactor: 重构代码，理论上不影响现有功能
- perf: 提升性能
- test: 增加修改测试用例
- chore: 修改工具相关（包括但不限于文档、代码生成等）
- deps: 升级依赖
**`例如`**

```
git commit -m 'fix:修复xxxbug'
```

<br/>
<br/>

- 第一次提交

```
cd /Users/harleyhuang/Documents/Gitee/SourceTreeTest
git add -A
git commit -m "第一次提交"
git push
//第一次提交需要输入用户名和密码，这个用户名和密码对应着GitHub、Gitee的用户名和密码
Username for 'https://gitee.com': [xxx@qq.com(用户名)]
Password for 'https://harelysoa@qq.com@gitee.com': [xxxxxx(密码)]
```


<br/>
<br/>

- 第二次提交

```
//第二次提交
git add -A
git commit -m "第二次提交"
git push
```

![SourceTree 效果图](https://upload-images.jianshu.io/upload_images/2959789-e09e82921f7be101.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>

![工作流](https://upload-images.jianshu.io/upload_images/2959789-9af4cd14dbe74644.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
<br/>

- 第三次提交

```
//添加到缓存区
git add -A
```

![由未缓存文件变为缓存文件](https://upload-images.jianshu.io/upload_images/2959789-8c02c392f056c509.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
//改动已经提交到了 HEAD，但是还没到你的远端仓库
git commit -m "第三次提交"
```

![已经提交到了 HEAD，但是还没到你的远端仓库](https://upload-images.jianshu.io/upload_images/2959789-baa584d84d73524f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
//状态改变，没有需要推送到远端的
 git push
```

![推送到远端](https://upload-images.jianshu.io/upload_images/2959789-9ab7eed32e0ef2a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>

***
<br/>

> <h1 id="rebas变基">rebas变基</h1>

<br/>

> <h2 id="单分支合并多次commit">单分支合并多次commit</h2>

注意：不要通过rebase对任何已经提交到公共仓库中的commit进行修改。

&emsp;当我们在本地仓库中提交了多次，在我们把本地提交push到公共仓库中之前，为了让提交记录更简洁明了，我们希望把如下分支B、C、D三个提交记录合并为一个完整的提交，然后再push到公共仓库。

<br/>

**`终端命令变基`**

![rebase 流程图](https://upload-images.jianshu.io/upload_images/2959789-3b64c3c484df85b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![rebase 之前状态](https://upload-images.jianshu.io/upload_images/2959789-cd589bdf7e52fb16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


现在我们在master分支上添加了5次提交，我们的目标是把最后三个提交合并为一个提交：
![终端6次提交，第一次提交是master的初始化](https://upload-images.jianshu.io/upload_images/2959789-0b880e7f8d3c7bb5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br/>
<br/>

变基命令：

`git rebase -i  [startpoint]  [endpoint]`

- ` -i`: 意思是`--interactive`，即弹出交互式的界面让用户编辑完成合并操作;
-  `[startpoint]  [endpoint]`则指定了一个编辑区间，如果不指定`[endpoint]`，则该区间的终点默认是当前分支`HEAD`所指向的`commit`(注：该区间指定的是一个前开后闭的区间)。

变基步骤：
-  输入命令：

```
git rebase -i 073ea5e //第三次提交ID

//上面等同如下命令
git rebase -i HEAD~3
```

![rebase 交互图](https://upload-images.jianshu.io/upload_images/2959789-d0e17f859da0cb13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**`命令说明`**
>pick：保留该commit（缩写:p）

>reword：保留该commit，但我需要修改该commit的注释（缩写:r）

>edit：保留该commit, 但我要停下来修改该提交(不仅仅修改注释)（缩写:e）

>squash：将该commit和前一个commit合并（缩写:s）

>fixup：将该commit和前一个commit合并，但我不要保留该提交的注释信息（缩写:f）

>exec：执行shell命令（缩写:x）

>drop：我要丢弃该commit（缩写:d）


![SourceTree 状态变化，为变基状态](https://upload-images.jianshu.io/upload_images/2959789-f4e730b8a3d9190c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  **修改提交内容**
![编辑提交内容](https://upload-images.jianshu.io/upload_images/2959789-594a246f597911e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  先 `esc` 键，然后输入`:wq` 进行保存，变成如下图：
![SourceTree 状态改变](https://upload-images.jianshu.io/upload_images/2959789-cf1e16d7d02d6d70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![终端修改信息](https://upload-images.jianshu.io/upload_images/2959789-90b01484bfcfb024.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  然后 `esc` 键，然后输入`:wq` 进行保存，变成如下图：

![SourceTree 状态改变](https://upload-images.jianshu.io/upload_images/2959789-3dc3bf0807a52069.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


终端修改提交信息
![修改注释信息](https://upload-images.jianshu.io/upload_images/2959789-0dbf2df70b3e267a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 最后 `esc` 键，然后输入`:wq` 进行保存，如下图:
![提交记录合并完成](https://upload-images.jianshu.io/upload_images/2959789-c07bf3916a3ee98c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**单分支上变基完成**
![变基完成](https://upload-images.jianshu.io/upload_images/2959789-6bffeca860a178c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
<br/>



**`SourceTree 单分支变基`**
- 选中变基到的开区间位置
![选中区间变基](https://upload-images.jianshu.io/upload_images/2959789-5f7dcd3417bf7b20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 变基信息修改
![变基信息修改](https://upload-images.jianshu.io/upload_images/2959789-abb4fdd265531534.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  完成变基
![完成变基后并进行了提交](https://upload-images.jianshu.io/upload_images/2959789-ff1350749daa4bad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)






<br/>
<br/>


> <h2 id="一分支提交合并到另一分支上">一分支提交合并到另一分支上</h2>

&emsp;  当我们项目中存在多个分支，有时候我们需要将某一个分支中的一段提交同时应用到其他分支中，就像下图：
![一分支提交合并到另一分支](https://upload-images.jianshu.io/upload_images/2959789-0bd8273f1edafa19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


-  初始SourceTree状态图：
![分支变基图](https://upload-images.jianshu.io/upload_images/2959789-fdc349f3faa88d04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



我们希望将develop分支中的cc、dd、ee部分复制到master分支中，这时我们就可以通过rebase命令来实现（如果只是复制某一两个提交到其他分支，建议使用更简单的命令:git cherry-pick）。

-  在实际模拟中，我们创建了【master】和【develop】两个分支:

`【master】 分支部分提交记录`
![master 部分分支提交记录](https://upload-images.jianshu.io/upload_images/2959789-7066884f4b3892ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


`【develop】分支提交记录`
![【develop】分支提交记录](https://upload-images.jianshu.io/upload_images/2959789-11e6f07d13c7694c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- rebase 命令格式

`git rebase   [startpoint]   [endpoint]  --onto  [branchName]`
&emsp;  `[startpoint]  [endpoint]`仍然和上一个命令一样指定了一个编辑区间(前开后闭)，`--onto`的意思是要将该指定的提交复制到哪个分支上。
所以，在找到`cc 提交(a69e8fbb1)`和`ee 提交(3e75b2d52)`的提交id后，我们运行以下命令：

![变基前的分支状态](https://upload-images.jianshu.io/upload_images/2959789-fca9c1d5e2db41f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


`git rebase a69e8fbb1 3e75b2d52 --onto master `
出现冲突了：
![变基出现冲突了](https://upload-images.jianshu.io/upload_images/2959789-7533ad3525266209.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  解决冲突，输入：

`git rebase --continue`
又出现了冲突，如下图：
![变基出现冲突](https://upload-images.jianshu.io/upload_images/2959789-d373312f9deeec61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 继续解决冲突并提交，终端输入：

`git rebase --continue`
` git add -A`
`git commit -m "解决变基冲突"`
![变基继续并提交，注意：此时是游离态 HEAD](https://upload-images.jianshu.io/upload_images/2959789-ba2b08aa86426869.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**`注意`**：当前HEAD处于游离状态，实际上，此时所有分支的状态应该是这样:
![处于游离态](https://upload-images.jianshu.io/upload_images/2959789-eeaf846c59bbf389.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 查看master分支提交日志：
![变基后的日志](https://upload-images.jianshu.io/upload_images/2959789-c07660c929619157.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


所以，虽然此时HEAD所指向的内容正是我们所需要的，但是master分支是没有任何变化的，git只是将`cc 提交~ee 提交`部分的提交内容复制一份粘贴到了`maste`r所指向的提交后面，我们需要做的就是将master所指向的提交id设置为当前HEAD所指向的提交id就可以了，即:

- 切换到master 分支

`git checkout master`

![切换到master分支](https://upload-images.jianshu.io/upload_images/2959789-341a80bc1c4f28bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这时你会发现，变基的内容不见了，怎么办？别急，看下面：

- 修改提交ID

`git reset --hard c8bb979`
![这时变基彻底完成了](https://upload-images.jianshu.io/upload_images/2959789-ab311f74d3711220.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)































<br/>

***
<br/>

> <h1 id="gitcheckout">git checkout</h1>

**`解析：`**

①  git checkout 实际上是修改HEAD文件的内容，让其指向不同的branch。HEAD文件指向的branch就是当前branch。


②  git checkout最简单的用法，显示工作区，暂存区和HEAD的差异

```
$ git checkout
M	x
Your branch is ahead of 'origin/master' by 1 commit.//意思是我本地仓库比远程仓库领先一个提交操作。

```

<br/>

***
<br/>

>#  <h1 id="gitbranch">git branch</h1>

① git branch用-a 参数，可以看到很多branch，包括远程的branch。

```
git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/develop
  remotes/origin/issue_193
  remotes/origin/issue_210
  remotes/origin/master
```






