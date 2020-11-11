- **`打包`**
	- 通过Xcode打包企业ipa包，选择`Enterprise`
<br/>
	
	![z10](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z10.png)
<br/>

- **`勾选Include manifest for over-the-air installation`**

<br/>

![z11](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z11.png)
<br/>

- **`生成一个info.plist文件`**

	暂时随意输入地址，比如`https://www.github.com`，后面要对其进行修改，地址要`https`。
		<br/>
		![z12](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z12.png)
	
<br/>	

- **`github上传`**
	- 首先在github上创建一个库，然后克隆到本地;
	- 把`.ipa info.plist 57x57.png 512x512.png`四个文件上传到提交到github上;
<br/>

- **`获取.ipa包的链接`**
<br/>
![z13](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z13.png)
<br/>

- **`修改info.plist 文件，点击Raw，然后进入新的页面，在浏览器中复制地址即可`**
<br/>

![z15](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z15.png)
<br/>

- **`info.plist修改，把获取到的地址，按位置然后填入到info.plist文件中`**
<br/>

![z14](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/z14.png)
<br/>

- **`安装路径`**
<br/>
`itms-services:///?action=download-manifest&url= info.plist文件的url地址`
<br/>
<br/>
`itms-services:///?action=download-manifest&url=https://github.com/harleyGit/StudyNotes/raw/master/Others/BRPlay/info.plist`

<br/>

- **`下载注意`**
<br/>
&emsp; 这些做好后，需要打开 vpn代理，进行翻墙否则无法下载。
描述文件[下载地址](https://airportal.cn/680874)

[![GitHubBRPlay](https://raw.githubusercontent.com/harleyGit/StudyNotes/master/Pictures/GitHubBRPlay.png)](https://www.liantu.com/)

