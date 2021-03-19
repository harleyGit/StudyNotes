- **Git提交代码**
- **错误解决**
- **Git配置**



<br/>

***
<br/>

># Git提交代码


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

># 错误解决

<br/>

> 1. **LibreSSL SSL_connect: SSL_ERROR_SYSCALL in connection to github.com:443** 

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

***
<br/>

># Git配置

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
