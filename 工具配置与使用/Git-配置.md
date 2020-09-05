查看电脑是否安装Git，终端输入：

```
git
```

安装过则会输出：

![安装 Git 后的终端截图](https://upload-images.jianshu.io/upload_images/2959789-5f498c4eb71ee8ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br/>
***
<br/>

># Git 配置

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
