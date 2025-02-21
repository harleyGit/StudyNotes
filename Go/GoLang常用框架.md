></h2>
- [**动手实现一个库**](#动手实现一个库)
	- [TravisCI](#TravisCI) 
	- [GitHub Actions](#GitHubActions) 
	- [DaoCloud](#DaoCloud)
- [**‌go-colly框架**](#go-colly框架)
	- [go-colly框架的特性](#go-colly框架的特性)
	- [go-colly框架使用](#go-colly框架使用)
	- [将抓取的网页内容存储在文件中](#将抓取的网页内容存储在文件中)
- [**‌gin框架**](#‌gin框架)
	- [把爬虫程序设置成Web服务](#把爬虫程序设置成Web服务)
- [**‌cellnet网络库**](#cellnet网络库)
- [**‌图表库——go-chart**](#图表库——go-chart)
- [**图表库go-echarts**](#图表库go-echarts)
- [**packr库处理模板引擎内的文件**](#packr库处理模板引擎内的文件)
- [**‌GJSON解析JSON数据**](#GJSON解析JSON数据)
- [**IrisWeb框架**](#IrisWeb框架)
	- [IrisWeb框架详解](#IrisWeb框架详解) 
	- [快速入门](#快速入门) 
	- [中间件](#中间件) 
	- [MVC模式](#MVC模式) 
	- [解析JSON请求](#解析JSON请求) 
	- [集成GORM](#集成GORM) 
	- [部署](#部署)
- **数据库框架**
	- [Redis](#Redis)
	- [GORM和XORM详介](#GORM和XORM详介)
		- [GORM使用](#GORM使用)
			- [GORM定义模型](#GORM定义模型) 
			- [GORM初始化数据库](#GORM初始化数据库) 
			- [GORM插入数据](#GORM插入数据) 
			- [GORM查询数据](#GORM查询数据) 
			- [GORM更新数据](#GORM更新数据) 
			- [GORM删除数据](#GORM删除数据)
	- [XORM使用](#XORM使用)
		- [XORM定义模型](#XORM定义模型) 
		- [XORM初始化数据库](#XORM初始化数据库) 
		- [XORM插入数据](#XORM插入数据) 
		- [XORM查询数据](#XORM查询数据)
			- [项目实践1-JOIN(select)](#项目实践1-JOIN(select))
			-  [项目实践2-where(select)](#项目实践2-where(select))
		- [XORM更新数据](#XORM更新数据) 
		- [XORM删除数据](#XORM删除数据)
	- [主要区别](#主要区别)
		- [什么时候选用？](#什么时候选用？)
	- [model序列化和继承](#model序列化和继承)
- [**安全**](#安全)
	- [crypto库](#crypto库)
		- [crypto哈希](#crypto哈希)


<br/>

***
<br/><br/><br/>
> <h1 id="动手实现一个库">动手实现一个库</h1>


- **Golang 源代码持续集成工具的用途和功能**

<br/>

- 这些工具主要用于**持续集成（CI）和持续交付/部署（CD）**，在开发过程中自动化以下任务：

	- **自动化构建**：检测代码更改后，自动编译和构建代码。
	- **自动化测试**：运行单元测试、集成测试，确保代码质量。
	- **代码质量检查**：执行代码静态分析（如 lint）以检测潜在问题。
	- **自动化部署**：在代码通过验证后，将其部署到生产或其他环境。
	- **团队协作**：在代码合并或提交后，快速获得构建和测试结果，提升开发效率。

<br/>

以下是 **TravisCI**、**GitHub Actions** 和 **DaoCloud** 的具体介绍和用途。


- **工具对比总结**

| 工具         | 特点                                      | 适用场景                          | 优势                          |
|--------------|------------------------------------------|-----------------------------------|-------------------------------|
| **TravisCI** | 经典 CI 服务，轻量级，开源项目友好         | 小型或中型项目的持续集成          | 配置简单，文档丰富            |
| **GitHub Actions** | 深度集成 GitHub，功能强大且灵活          | GitHub 托管的项目持续集成          | 社区资源丰富，强大的触发机制   |
| **DaoCloud** | 国内服务，支持容器化、云原生               | 国内项目，依赖 Docker 或 Kubernetes | 图形化界面，国内服务稳定       |

根据你的项目托管平台、需求和团队协作方式选择合适的工具：  
- **开源项目**：推荐 TravisCI 或 GitHub Actions。
- **国内私有项目**：推荐 DaoCloud。



<br/><br/><br/>

> <h2 id="TravisCI">TravisCI</h2>

- **用途**

TravisCI 是一款流行的持续集成服务，支持多个编程语言，包括 Go。主要用于 GitHub 仓库的持续集成。它会在代码推送或提交 Pull Request 时触发自动化流程。

<br/>

- **功能**
	- 自动化构建和测试。
	- 支持多种操作系统（Linux、macOS、Windows）。
	- 集成 Docker 支持复杂的构建环境。
	- 与 GitHub 深度集成，可在 PR 中显示测试状态。

<br/>

- **使用示例**
在项目中添加 `.travis.yml` 文件：

```yaml
language: go
go:
  - 1.19 # 指定 Go 版本
script:
  - go build -v ./...  # 编译项目
  - go test -v ./...   # 运行测试
```

<br/>

**用途场景**：
- 快速设置和运行持续集成。
- 检测 Go 项目中代码的构建和测试状态。


<br/><br/><br/>
> <h2 id="GitHubActions">GitHub Actions</h2>

- **用途**

GitHub Actions 是 GitHub 自带的 CI/CD 工具，专门为 GitHub 仓库设计。通过定义 **workflow** 文件，你可以自动化项目的整个开发周期，包括构建、测试、部署等。

<br/>
- **功能**
	- 使用 YAML 文件定义构建流程（`.github/workflows/*.yml`）。
	- 丰富的社区 Actions（复用其他开发者的任务模板）。
	- 支持事件触发（如 Push、PR 或 Schedule 定时任务）。
	- 提供多平台支持（Linux、Windows、macOS）。

<br/>

- **使用示例**

在项目中添加 `.github/workflows/ci.yml` 文件：

```yaml
name: Go CI
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Go
        uses: actions/setup-go@v4
        with:
          go-version: 1.19
      - name: Install dependencies
        run: go mod tidy
      - name: Build
        run: go build ./...
      - name: Test
        run: go test ./...
```

<br/>

- **用途场景**：
	- **开源项目**：适合 GitHub 托管的开源项目，社区用户可以快速贡献。
	- **团队协作**：在代码合并前确保代码质量。

<br/><br/><br/>
> <h2 id="DaoCloud">DaoCloud</h2>

- **用途**
DaoCloud 是中国开发者常用的一体化云原生工具平台，支持 CI/CD、容器化部署、DevOps 管理等功能。特别适合中国的开发者和企业使用。

<br/>

- **功能**
	- 图形化界面，操作简单易用。
	- 支持从源码构建、测试到 Docker 镜像生成的全流程。
	- 与国内代码托管平台（如 Gitee）深度集成。
	- 提供更快的国内镜像加速服务。

<br/>

- **使用示例**

通过图形化界面完成以下配置：
	1. 选择代码源（如 GitHub 或 Gitee）。
	2. 添加构建任务，例如运行以下命令：

```bash
go build ./...
go test ./...
docker build -t myapp .
```

3. 配置自动化部署到 Kubernetes 集群或其他环境。

- **用途场景**：
	- 国内项目的持续集成和交付。
	- 快速实现 CI/CD 和容器化交付。

<br/>




<br/>

***
<br/><br/><br/>
> <h1 id="go-colly框架">go-colly框架</h1>

go-colly是使用Go语言实现的网络爬虫框架。go-colly以回调函数的形式提供了一组接口，通过这些接口能够实现任意类型的爬虫。开发者使用go-colly框架可以轻松地从Web页面中爬取结构化数据。

<br/>
**通过如下命令下载到项目中：**

```
cd cd /Users/ganghuang/HGFiles/GitHub/GoProject/MLC_GO
go get -u github.com/gocolly/colly/...
```


<br/><br/><br/>
> <h2 id="go-colly框架的特性">go-colly框架的特性</h2>
- **go-colly框架具有如下特性。**
	- 清晰的API。快速（单核>1k请求/s）​。
	- 管理每个域的请求延迟和最大并发性。
	- 自动cookie和会话处理。
	- 同步／异步／并行抓取。高速缓存。
	- 自动处理非Unicode的编码。
	- 支持Robots.txt定制Agent信息。
	- 定制抓取频次。



<br/><br/><br/>
> <h2 id="go-colly框架使用">go-colly框架使用</h2>

**导入框架**

```
import "github.com/gocolly/colly"
```

<br/>

&emsp; 使用go-colly框架的关键是创建Collector对象（即“收集器”​）​，该对象的作用是管理网络通信，并负责在收集任务运行时执行附加的回调函数。通过调用colly库中的NewCollector()函数，即可创建Collector对象。其语法格式如下。

```
colly.NewCollector()
```

<br/>

![go.0.0.55.png](./../Pictures/go.0.0.55.png)


- **这些回调函数在收集任务运行时被有序调用，调用顺序如下。**
	- OnRequest()函数在请求发出前被调用。
	- OnError()函数在请求过程中发生错误时被调用。
	- OnResponse()函数在收到响应后被调用。
		- 如果响应消息的内容是HTML，则在OnResponse()函数执行完毕后调用OnHTML()函数。
		- 如果响应消息的内容是HTML或XML，则在OnHTML()函数执行完毕后调用OnXML()函数。
		- OnXML()函数执行完毕后调用OnScraped()函数。

<br/><br/>

运行程序后，程序根据http://news.baidu.com开始抓取页面结果。通过回调函数OnHTML()，能够分析页面中的新闻标题及其链接。代码如下:

```
package main

import (
	"fmt"

	"github.com/gocolly/colly"
)

func main() {
	// 根据http://news.baidu.com开始抓取页面结果
	testCrawlerBaidu()
}

/// 爬虫百度网页
func testCrawlerBaidu(){
	c := colly.NewCollector(
		// colly.AllowedDomains("news.baidu.com"),
		colly.UserAgent("Opera/9.80(Windows NT 6.1; U; zh-cn) Presto/2.9.168 Version/11.50"))
	// 发出请求时向 Collector 对象附加的回调函数
	c.OnRequest(func(r *colly.Request){
		// 设置请求头
		r.Headers.Set("Host", "baidu.com")
		r.Headers.Set("Connection", "keep-alive")
		r.Headers.Set("Accept", "*/*")
		r.Headers.Set("Origin", "")
		r.Headers.Set("Referer", "http://www.baidu.com")
		r.Headers.Set("Accept-Encoding", "gzip, deflate")
		r.Headers.Set("Accept-Language", "zh-CN, zh;q=0.9")
		fmt.Println("正在访问：", r.URL)
	})

	// 处理HTML中的文档标题
	c.OnHTML("title", func (e *colly.HTMLElement)  {
		fmt.Println("文档标题：", e.Text)
	})

	// 处理HTML中的文档内容
	c.OnHTML("body", func (e *colly.HTMLElement)  {
		e.ForEach(".hotnews a", func (i int, el *colly.HTMLElement)  {
			band := el.Attr("href")
			title := el.Text
			fmt.Printf("文档内容 %d: %s\n%s\n", i+1, title, band)
		})
	})

	// 获取收到的响应内容的数量
	c.OnResponse(func(r *colly.Response){
		fmt.Println("收到响应内容的数量： ", r.StatusCode)
	})
	
	// 限制 visit 的线程数， visit 可以同时运行多个
	c.Limit(&colly.LimitRule{
		Parallelism: 2,
	})

	c.Visit("http://news.baidu.com")
}
```

**Log:**

```
ganghuang@GangHuangs-MacBook-Pro TestCrawlerBaidu % go run test_crawler_baidu.go
正在访问： http://news.baidu.com
收到响应内容的数量：  200
文档标题： 百度新闻——海量中文资讯平台
文档内容 1: 习近平向全国各族人民致以新春祝福
https://content-static.cctvnews.cctv.com/snow-book/index.html?item_id=5200598709432592760&toc_style_id=feeds_default&track_id=34D9D8BD-D879-4708-BF3E-A8D868C6C660_759409923605&share_to=copy_url
文档内容 2: 习近平春节前夕赴辽宁看望慰问基层干部群众
https://news.cctv.com/2025/01/24/ARTI6hnAgPkfUOy7vO9AAjFB250124.shtml
文档内容 3: 总书记的关怀让我们感到幸福温暖
https://h.xinhuaxmt.com/vh512/share/12380734?newstype=1001&homeshow=1
文档内容 4: 转型升级 “智”造强企 传统钢企焕发青春
https://content-static.cctvnews.cctv.com/snow-book/index.html?item_id=16973667513549662031&toc_style_id=feeds_default&track_id=3C71E229-8F36-461A-8F0A-AAD40C7030ED_759408113589&share_to=copy_url
文档内容 5: 习近平辽宁行｜烟火气里年味浓——走进沈阳大东副食品商场
https://content-static.cctvnews.cctv.com/snow-book/index.html?item_id=1274157371348255293&toc_style_id=feeds_default&track_id=25D644CD-6C11-463A-921D-8BC0F1C6B019_759329323922&share_to=copy_url
文档内容 6: 向“新”发力 逐“质”而行
https://content-static.cctvnews.cctv.com/snow-book/index.html?item_id=16224630468348415654&toc_style_id=feeds_default&share_to=copy_url&track_id=24e45dbe-d36d-4e2a-82b5-1d3700f1f9a2
文档内容 7: 时政Vlog｜上联：年货置办中 下联：今日宜“大东” 横批：老香了
https://content-static.cctvnews.cctv.com/snow-book/index.html?item_id=95295998088496574&channelId=1119&toc_style_id=feeds_default&track_id=D9D2124E-0D13-4872-A820-380C0AF85403_759396097754&share_to=copy_url
文档内容 8: 延安苹果产业里的新质生产力（新春走基层）
https://www.peopleapp.com/column/30048094126-500006064834
文档内容 9: 新春走基层｜“隧道探险家”出海记
https://app.people.cn/h5/detail/normal/6157882818937856
文档内容 10: 大市场看中国年｜逛百年大集 寻乡愁记忆
https://www.cnr.cn/tj/tjtp/20250124/t20250124_527050755.shtml
```

<br/><br/>
> <h2 id="抓取指定连接的网页内容"> 抓取指定连接的网页内容 </h2>

控制台打印的抓取结果是爬虫程序通过限制域名、设置抓取深度、过滤URL后得到的。通过访问http://news.baidu.com发现该链接的网页内容很丰富，涉及方方面面的领域。那么，如何才能把http://news.baidu.com的网页内容全部抓取下来呢？代码如下。

```
// 访问指定网址
testCrawlerAppointWebpage("https://news.baidu.com")


/// 抓取指定连接的网页内容 urls: 指定链接
func testCrawlerAppointWebpage(urls string) {
	// 创建 Collector 对象
	c := colly.NewCollector()
	/* 
	* 是否抓取指定链接的网页内容
	* 初始设置为不抓取指定链接的网页内容
	 */
	visited := false
	// 使用 Collector 对象抓取 URL
	c.OnResponse(func (r *colly.Response)  {
		if !visited {
			visited = true
			r.Request.Visit("/get?q=2")
		}
	})

	// 对指定链接网页内容进行处理
	c.OnHTML("a[href]", func (e *colly.HTMLElement)  {
		href := e.Text	// 获取指定链接的网页内容
		fmt.Println("🍎 网页内容：",href)	// 打印指定链接的网页内容
	})
	c.Visit(urls)	//访问指定链接
}
```

**Log**

```
ganghuang@GangHuangs-MacBook-Pro TestCrawlerBaidu % go run test_crawler_baidu.go
🍎 网页内容： 网页
🍎 网页内容： 贴吧
🍎 网页内容： 知道
🍎 网页内容： 音乐
🍎 网页内容： 图片
🍎 网页内容： 视频
🍎 网页内容： 地图
🍎 网页内容： 文库
🍎 网页内容： 




🍎 网页内容： 帮助
🍎 网页内容： 首页
🍎 网页内容： 国内
🍎 网页内容： 国际
🍎 网页内容： 军事
🍎 网页内容： 财经
🍎 网页内容： 娱乐
🍎 网页内容： 体育
🍎 网页内容： 互联网
🍎 网页内容： 科技
🍎 网页内容： 游戏
🍎 网页内容： 女人
🍎 网页内容： 汽车
🍎 网页内容： 房产
🍎 网页内容： 首页
🍎 网页内容： 国内
🍎 网页内容： 国际
🍎 网页内容： 军事
🍎 网页内容： 财经
🍎 网页内容： 娱乐
🍎 网页内容： 体育
🍎 网页内容： 互联网
🍎 网页内容： 科技
🍎 网页内容： 游戏
🍎 网页内容： 女人
🍎 网页内容： 汽车
🍎 网页内容： 房产
🍎 网页内容： 


点击刷新，将会有未读推荐


🍎 网页内容： 热点要闻
🍎 网页内容： 
🍎 网页内容： 习近平向全国各族人民致以新春祝福
🍎 网页内容： 习近平春节前夕赴辽宁看望慰问基层干部群众
🍎 网页内容： 总书记的关怀让我们感到幸福温暖
🍎 网页内容： 转型升级 “智”造强企 传统钢企焕发青春
🍎 网页内容： 习近平辽宁行｜烟火气里年味浓——走进沈阳大东副食品商场
🍎 网页内容： 向“新”发力 逐“质”而行
🍎 网页内容： 时政Vlog｜上联：年货置办中 下联：今日宜“大东” 横批：老香了
🍎 网页内容： 延安苹果产业里的新质生产力（新春走基层）
🍎 网页内容： 新春走基层｜“隧道探险家”出海记
🍎 网页内容： 大市场看中国年｜逛百年大集 寻乡愁记忆
🍎 网页内容： 中经评论：1.4万亿斤！中国粮“藏”在哪？
🍎 网页内容： 共享中国发展机遇 美国企业持续看好中国市场
🍎 网页内容： 我在中国过大年｜当“非遗版”春节遇上非遗彝绣
🍎 网页内容： 香港点亮锦鲤醒狮彩灯迎新春
🍎 网页内容： 专题：京彩春节的N种打开方式
🍎 网页内容： 今日辟谣
🍎 网页内容： 北京网站辟谣平台
🍎 网页内容： 互联网联合辟谣平台
🍎 网页内容： 多地“禁改限”明确春节可燃放烟花爆竹
🍎 网页内容： 暴雪、寒潮、大雾三预警齐发！多地高速封闭，列车停运
🍎 网页内容： 传统习俗迎春节 来看这些地方的浓浓年味
🍎 网页内容： 铁路桥隧工孩子年前坐火车“看爸爸” 父亲在崖壁喊到飙泪
🍎 网页内容： 国家药监局药审中心回应个别品种数据重复：编辑错误导致
🍎 网页内容： 美国总统特朗普称将再次和朝鲜领导人取得联系
🍎 网页内容： 

点击刷新，将会有未读推荐

🍎 网页内容： 更多个性推荐新闻
🍎 网页内容： 


🍎 网页内容： 


🍎 网页内容： 
🍎 网页内容： 8
🍎 网页内容： 7
🍎 网页内容： 6
🍎 网页内容： 5
🍎 网页内容： 4
🍎 网页内容： 3
🍎 网页内容： 2
🍎 网页内容： 1
🍎 网页内容： 
🍎 网页内容： 
🍎 网页内容： 年的味道
🍎 网页内容： 飞天逐梦探苍穹
🍎 网页内容： 今冬以来最强雨雪来袭
🍎 网页内容： 午睡可以修复全身多个器官
🍎 网页内容： 老君山景区紧急闭园
🍎 网页内容： 医生说这6种病是衰老的表现
🍎 网页内容： 监控拍下火箭残骸掉落瞬间
🍎 网页内容： 河里钓起整箱现金?谣言
🍎 网页内容： 第一批外国网友已到中国“赶春运”
🍎 网页内容： 美拨款25亿美元应对洛杉矶山火
🍎 网页内容： 
🍎 网页内容： 
🍎 网页内容： 
🍎 网页内容： 辟谣
🍎 网页内容： 举报
🍎 网页内容： 二维码
🍎 网页内容： 收藏本站
🍎 网页内容： 搜索
🍎 网页内容： 用户反馈
🍎 网页内容： 
🍎 网页内容： Android版下载
🍎 网页内容： iPhone版下载
🍎 网页内容： 用户协议
🍎 网页内容： 隐私策略
🍎 网页内容： 企业推广
🍎 网页内容： 投诉中心
🍎 网页内容： 营业执照
🍎 网页内容： 《互联网新闻信息服务许可》编号：11220180008
🍎 网页内容： 《互联网宗教信息服务许可证》编号：京（2022）0000043
🍎 网页内容： 使用百度前必读
🍎 网页内容： 

🍎 网页内容： 

🍎 网页内容： 
```


<br/><br/>
> <h2 id="将抓取的网页内容存储在文件中">将抓取的网页内容存储在文件中</h2>

通过19.4节的操作已经把http://news.baidu.com的网页内容全部抓取下来，并把这些内容打印在控制台上。但是，如何才能把抓取下来的网页内容存储在文件中呢？代码如下。

```
// 存储爬虫文件
func testCrawlerAppointWebpageAndSaveTxt(urls string) {
	// 创建 Collector 对象
	c := colly.NewCollector()
	/* 
	* 是否抓取指定链接的网页内容
	* 初始设置为不抓取指定链接的网页内容
	 */
	visited := false
	// 使用 Collector 对象抓取 URL
	c.OnResponse(func (r *colly.Response)  {
		if !visited {
			visited = true
			r.Request.Visit("/get?q=2")
		}
	})

	// 文件的创建和持续写入
	filename := "news.txt"	// 文件名
	var f *os.File
	var fileErr error 
	if testCheckFileExist(filename) {	// 如果文件存在
		f, fileErr = os.OpenFile(filename, os.O_APPEND | os.O_WRONLY, 0666)	// 打开文件
		if fileErr != nil {
			fmt.Printf("❌ 打开文件 news.txt 失败：%v\n", fileErr)
		}
	} else {
		f,_ = os.Create(filename)	// 创建文件
		fmt.Println("🍎 新建文件 news.txt")

	}
	defer f.Close()	// 关闭文件
	write := bufio.NewWriter(f)
	defer write.Flush()

	// 对指定链接网页内容进行处理
	c.OnHTML("a[href]", func (e *colly.HTMLElement)  {
		href := e.Text	// 获取指定链接的网页内容
		write.WriteString("🍒 "+href+"\n")
		fmt.Println("🍎 网页内容：",href)	// 打印指定链接的网页内容
	})


	c.Visit(urls)	//访问指定链接
}
/* 检测文件是否存在
* fileName 文件名
 */
func testCheckFileExist(filename string) bool {
	var exist = true
	if _, err := os.Stat(filename); os.IsNotExist(err) {
		exist = false
	}
	return exist
}
```

效果图：

![go.0.0.56.png](./../Pictures/go.0.0.56.png)

<br/>

***
<br/><br/><br/>
> <h1 id="‌gin框架">‌gin框架</h1>

获取gin框架的包：

```
go get github.com/gin-gonic/gin
```

<br/><br/><br/>
> <h2 id="把爬虫程序设置成Web服务">把爬虫程序设置成Web服务</h2>

把爬虫程序设置成Web服务。具体修改如下：运行修改后的爬虫程序，打开浏览器，访问“本机IP：端口”​；当在网页上看到提示信息（​“已经把http://news.baidu.com的网页内容全部抓取下来，请查看当前项目目录下的news.txt文件！”​）时，说明已经把http://news.baidu.com的网页内容全部抓取下来，并存储在当前项目目录下的news.txt文件中。

为了满足上述的修改要求，需要下载第三方包gin。如图19.8所示，下载gin包的步骤如下。

```
// 把爬虫程序设置成Web服务
func testCrawlerAppointWebpageAndSaveTxtV2() {
	// 初始化引擎
	engine := gin.Default()
	// 注册一个路由和处理函数
	engine.Any("/", WebRoot)
	// 绑定端口
	engine.Run(":9200")
}
// 处理路由函数
func WebRoot(context *gin.Context){
	// 调用 testCrawlerAppointWebpageAndSaveTxt()函数
	testCrawlerAppointWebpageAndSaveTxt("https://news.baidu.com")
	// 设置网页上的文本内容
	context.String(http.StatusOK, "已经把http://news.baidu.com 的网页内容全部抓取下来\n 请查看当前项目目录下的news.txt文本文件！")
}

// 存储爬虫文件
func testCrawlerAppointWebpageAndSaveTxt(urls string) {
	// 创建 Collector 对象
	c := colly.NewCollector()
	/* 
	* 是否抓取指定链接的网页内容
	* 初始设置为不抓取指定链接的网页内容
	 */
	visited := false
	// 使用 Collector 对象抓取 URL
	c.OnResponse(func (r *colly.Response)  {
		if !visited {
			visited = true
			r.Request.Visit("/get?q=2")
		}
	})

	// 文件的创建和持续写入
	filename := "news.txt"	// 文件名
	var f *os.File
	var fileErr error 
	if testCheckFileExist(filename) {	// 如果文件存在
		f, fileErr = os.OpenFile(filename, os.O_APPEND | os.O_WRONLY, 0666)	// 打开文件
		if fileErr != nil {
			fmt.Printf("❌ 打开文件 news.txt 失败：%v\n", fileErr)
		}
	} else {
		f,_ = os.Create(filename)	// 创建文件
		fmt.Println("🍎 新建文件 news.txt")

	}
	defer f.Close()	// 关闭文件
	write := bufio.NewWriter(f)
	defer write.Flush()

	// 对指定链接网页内容进行处理
	c.OnHTML("a[href]", func (e *colly.HTMLElement)  {
		href := e.Text	// 获取指定链接的网页内容
		write.WriteString("🍒 "+href+"\n")
		fmt.Println("🍎 网页内容：",href)	// 打印指定链接的网页内容
	})


	c.Visit(urls)	//访问指定链接
}
/* 检测文件是否存在
* fileName 文件名
 */
func testCheckFileExist(filename string) bool {
	var exist = true
	if _, err := os.Stat(filename); os.IsNotExist(err) {
		exist = false
	}
	return exist
}
```

**Log:**

```
ganghuang@GangHuangs-MacBook-Pro TestCrawlerBaidu % go run test_crawler_baidu.go
ganghuang@GangHuangs-MacBook-Pro TestCrawlerBaidu % go run test_crawler_baidu.go
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /                         --> main.WebRoot (3 handlers)
[GIN-debug] POST   /                         --> main.WebRoot (3 handlers)
[GIN-debug] PUT    /                         --> main.WebRoot (3 handlers)
[GIN-debug] PATCH  /                         --> main.WebRoot (3 handlers)
[GIN-debug] HEAD   /                         --> main.WebRoot (3 handlers)
[GIN-debug] OPTIONS /                         --> main.WebRoot (3 handlers)
[GIN-debug] DELETE /                         --> main.WebRoot (3 handlers)
[GIN-debug] CONNECT /                         --> main.WebRoot (3 handlers)
[GIN-debug] TRACE  /                         --> main.WebRoot (3 handlers)
[GIN-debug] [WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.
Please check https://pkg.go.dev/github.com/gin-gonic/gin#readme-don-t-trust-all-proxies for details.
[GIN-debug] Listening and serving HTTP on :9200
```

<br/>

此时，Mac系统弹出如图19.9所示的Mac安全警报。选中“公用网络”复选框，单击“允许访问”按钮。

打开默认浏览器（比如：Safari浏览器），访问“本机IP：端口”​（即127.0.0.1:9200）​，即可看到如图19.10所示的提示信息。这段程序在当前项目目录下生成news.txt文件。单击news.txt，即可看到抓取的网页内容，如图所示。

![go.0.0.57.png](./../Pictures/go.0.0.57.png)

<br/>

***
<br/><br/><br/>
># <h1 id="cellnet网络库">[cellnet网络库](https://github.com/davyxu/cellnet)</h1>

cellnet的设计理念是：高性能、简单、方便、开箱即用，希望开发者能使用cellnet迅速开展业务开发，而无须为底层性能调优及架构扩展而担忧。

[cellnet网络库](https://github.com/davyxu/cellnet)多个版本的迭代，无论是作为初学者学习的范例，还是作为私用、商用项目的基础构建乃至核心技术层已经在业内广受了解及使用。

- **主要使用领域：**
	- 游戏服务器
	- 方便定制私有协议，快速构建逻辑服务器、网关服务器、服务器间互联互通、对接第三方SDK、转换编码协议等
	- ARM设备
	- 设备间网络通讯
	- 证券软件
	- 内部RPC

<br/><br/><br/>
> <h2 id="cellnet网络库的流程及架构">cellnet网络库的流程及架构</h2>

- cellnet的主要处理流程和组件由下面几个部分组成。
	- Socket连接管理：cellnet网络库使用连接器和接受器(Connector、Acceptor)管理Socket连接。
	- 会话(Session)：客户端和服务器连接使用会话(Session)处理收发包流程。收发包的流程将事件通过事件回调(cellnet.EventFunc)派发。
	-  包处理(packet)：cellnet中的packet包处理会话收发流程派发的事件，实现变长封包的解析、处理和收发。
	-  编码器：用户的封包使用编码器(Codec)负责原始封包的字节数组和用户消息间的转换。
	-  消息队列：可以将收到的消息按顺序排队并提供给用户进行处理。
	-  消息元信息(MessageMeta)：为所有的系统提供静态的消息扩展信息，如消息的ID、编码器、创建方法等。

![go.0.0.58.png](./../Pictures/go.0.0.58.png)

<br/>

***
<br/><br/><br/>
> <h1 id="图表库——go-chart">图表库——go-chart</h1>

市面上有诸多的开源图表库，比如百度开源的ECharts、阿里开源的BizCharts、Chart.js、HighCharts、G2、D3、Google出品的Google Charts等，这些开源的库在兼容性、拓展性、支持图表的种类、交互等层面各有优缺点，具体的选择要结合具体的情况（比如开发者选择的技术栈、需要支持的图表种类等各方面）​。


参照之前的思想，本节将基于chart.js构建一个完整的图表库go-chart，支持折线图、柱状图、饼图、雷达图、散点图、气泡图等。

[完整代码地址](https://github.com/goecharts/go-chart)


<br/>

***
<br/><br/><br/>
># <h1 id="图表库go-echarts">[图表库go-echarts](https://github.com/go-echarts/go-echarts)</h1>

基于ECharts图表库逐步讲解如何构建图表，整体的核心思想是：

- 明确ECharts如何构建图表。
- 延伸到使用模板引擎加载动态数据的方式构建图表。
- 抽取公共的结构体，方便组合复用。
- 各图表类型对象实现定义的接口(Interface)。

当然，还有很多细节没有涉及，但是整体思想皆是如此，如果想基于ECharts构建更为丰富的图表类型，那么可以查看[go-echarts](https://github.com/go-echarts/go-echarts)，其中采用的思维方式是类似的，而实现的功能更加完善。


<br/>

***
<br/><br/><br/>
> <h1 id="packr库处理模板引擎内的文件">packr库处理模板引擎内的文件</h1>

packr库能够比较优雅地处理模板引擎内的文件。

**下载命令如下：**

```
go get -u github.com/gobuffalo/packr
```

<br/>

***
<br/><br/><br/>
> <h1 id="GJSON解析JSON数据">GJSON解析JSON数据</h1>

GJSON是一个更便捷的解析JSON数据的第三方库，支持链式操作，能让读者非常方便地获取任意数据。

**下载**

```
go get -u github.com/tidwall/gjson
```


<br/>

***
<br/><br/><br/>
> <h1 id="IrisWeb框架">Iris Web框架</h1>

<br/><br/><br/>
> <h2 id="IrisWeb框架详解">Iris Web框架详解</h2>

- **1.什么是 Iris？**

[Iris](https://www.iris-go.com/) 是 Go 语言中的一个高性能、功能丰富的 Web 框架，提供了强大的路由、MVC 支持、中间件机制和 WebSocket 处理能力。它是 Go 生态中**最快**的 Web 框架之一，并支持扩展性强的功能，如依赖注入、JWT 认证、多种模板引擎、数据库集成等。

<br/>

- **2.Iris 的特点**
	- **高性能**：Iris 号称是 **最快的 Go Web 框架**，比 `gin` 更快（官方 Benchmark 测试）。
	- **简洁易用**：API 设计清晰，代码风格简洁。
	- **MVC 设计模式**：支持 `Model-View-Controller` 架构，适用于大型项目。
	- **内置中间件**：支持 `JWT`、`CORS`、`日志`、`Gzip` 压缩等。
	- **支持 WebSocket**：轻松构建实时应用，如聊天室、推送服务等。
	- **多种视图引擎**：支持 HTML、Pug、Django、Markdown、React、Vue SSR 等。
	- **依赖注入**：支持 `DI（Dependency Injection）`，方便管理服务。

<br/>

- **3. 安装 Iris**

可以使用 `go get` 安装：

```sh
go get github.com/kataras/iris/v12
```

或者使用 `go mod`：

```sh
go mod init myproject
go get github.com/kataras/iris/v12
```

<br/><br/><br/>
> <h2 id="快速入门">快速入门</h2>

```go
package main

import (
	"fmt"
	"github.com/kataras/iris/v12"
)

func main() {
	app := iris.New()

	// 定义路由
	app.Get("/", func(ctx iris.Context) {
		ctx.WriteString("Hello, Iris!")
	})

	// 启动服务器
	err := app.Listen(":8080")
	if err != nil {
		fmt.Println("服务器启动失败:", err)
	}
}
```

**运行**

```sh
go run main.go
```

浏览器访问 `http://localhost:8080/`，可以看到：

```
Hello, Iris!
```

<br/><br/>
或者如下：

```
package main

import "github.com/kataras/iris/v12"

func main() {
	practiceIrisWebLibV0_get()
}

func practiceIrisWebLibV0_get() {
	// 默认服务引擎
	app := iris.Default()
	// 路由： /ping
	// 控制逻辑： 返回JSON格式内容
	app.Get("/ping", func (ctx iris.Context)  {
		ctx.JSON(iris.Map{
			"message": "pong",
		})
	})
	// listen and serve on http://0.0.0.0:8080
	app.Run(iris.Addr(":8080"))
}
```

控制台运行：

```
go run PLrisWebLibV0.go
[DBUG] 2025/02/02 12:02 Log level set to "debug"
[DBUG] 2025/02/02 12:02 Using <UUID4> to identify requests
[DBUG] 2025/02/02 12:02 API: 1 registered route (1 GET)
GET: /ping (./PLrisWebLibV0.go:23)

Iris Version: 12.2.11

Now listening on: 
> Network:  http://192.168.0.105:8080
> Local:    http://localhost:8080
Application started. Press CTRL+C to shut down.
```

在浏览器输入：

```
http://localhost:8080/ping
```

出现：

```
{"message":"pong"}
```


<br/><br/>

如何获取到路径中的参数？通过iris.Context。

```
// 默认服务引擎
app := iris.Default()
// 路由： /ping
// 控制逻辑： 返回JSON格式内容
app.Get("/ping", func (ctx iris.Context)  {
	// 其他的类型都提供了相应的方法
	ctx.Params().GetInt("id")
})
```


<br/><br/><br/>
> <h2 id="路由">路由</h2>

- **基本路由**

```go
app.Get("/hello", func(ctx iris.Context) {
	ctx.WriteString("Hello, World!")
})
```

**动态路由**

```go
app.Get("/user/{id:int}", func(ctx iris.Context) {
	id := ctx.Params().GetIntDefault("id", 0)
	ctx.WriteString(fmt.Sprintf("User ID: %d", id))
})
```

- 访问 `http://localhost:8080/user/123` 返回 `User ID: 123`

<br/><br/>

**通配符路由**

```go
app.Get("/files/{filepath:path}", func(ctx iris.Context) {
	filepath := ctx.Params().Get("filepath")
	ctx.WriteString("File path: " + filepath)
})
```

- 访问 `http://localhost:8080/files/images/photo.jpg` 返回 `File path: images/photo.jpg`

<br/><br/><br/>
> <h2 id="中间件">中间件</h2>

- **全局中间件**

```go
app.Use(func(ctx iris.Context) {
	ctx.Application().Logger().Infof("请求路径: %s", ctx.Path())
	ctx.Next()
})
```

<br/>

**局部中间件**

```go
app.Get("/admin", authMiddleware, func(ctx iris.Context) {
	ctx.WriteString("Welcome to Admin Panel")
})

func authMiddleware(ctx iris.Context) {
	token := ctx.GetHeader("Authorization")
	if token != "valid-token" {
		ctx.StatusCode(iris.StatusForbidden)
		ctx.WriteString("Unauthorized")
		return
	}
	ctx.Next()
}
```

- 访问 `http://localhost:8080/admin`，需要携带 `Authorization: valid-token` 头，否则返回 `Unauthorized`。

<br/><br/><br/>
> <h2 id="MVC模式">MVC 模式</h2>

Iris 支持 MVC 结构，让代码更清晰。

**创建 MVC 控制器**

```go
package main

import (
	"github.com/kataras/iris/v12"
)

type UserController struct{}

func (c *UserController) Get() string {
	return "Get All Users"
}

func (c *UserController) GetBy(id int) string {
	return fmt.Sprintf("User ID: %d", id)
}

func main() {
	app := iris.New()
	mvc.New(app.Party("/users")).Handle(new(UserController))
	app.Listen(":8080")
}
```

- 访问 `http://localhost:8080/users` 返回 `Get All Users`

- 访问 `http://localhost:8080/users/10` 返回 `User ID: 10`

<br/><br/><br/>
> <h2 id="解析JSON请求">解析JSON请求</h2>

```go
app.Post("/json", func(ctx iris.Context) {
	var user struct {
		Name  string `json:"name"`
		Email string `json:"email"`
	}

	if err := ctx.ReadJSON(&user); err != nil {
		ctx.StatusCode(iris.StatusBadRequest)
		ctx.WriteString("Invalid JSON")
		return
	}

	ctx.JSON(iris.Map{
		"message": "User Created",
		"user":    user,
	})
})
```
**测试**

```sh
curl -X POST "http://localhost:8080/json" \
     -H "Content-Type: application/json" \
     -d '{"name": "Alice", "email": "alice@example.com"}'
```

返回：

```json
{
    "message": "User Created",
    "user": {
        "name": "Alice",
        "email": "alice@example.com"
    }
}
```

<br/><br/><br/>
> <h2 id="集成GORM">集成 GORM</h2>

```go
package main

import (
	"fmt"
	"github.com/kataras/iris/v12"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
)

type User struct {
	ID    uint   `gorm:"primaryKey"`
	Name  string `json:"name"`
	Email string `json:"email"`
}

var db *gorm.DB

func main() {
	var err error
	dsn := "root:password@tcp(127.0.0.1:3306)/test_db?charset=utf8mb4&parseTime=True&loc=Local"
	db, err = gorm.Open(mysql.Open(dsn), &gorm.Config{})
	if err != nil {
		panic("数据库连接失败: " + err.Error())
	}

	db.AutoMigrate(&User{})

	app := iris.New()

	app.Post("/users", func(ctx iris.Context) {
		var user User
		if err := ctx.ReadJSON(&user); err != nil {
			ctx.StatusCode(iris.StatusBadRequest)
			return
		}
		db.Create(&user)
		ctx.JSON(user)
	})

	app.Get("/users", func(ctx iris.Context) {
		var users []User
		db.Find(&users)
		ctx.JSON(users)
	})

	app.Listen(":8080")
}
```

- `POST /users` 创建用户
- `GET /users` 获取所有用户

<br/><br/><br/>
> <h2 id="部署">部署</h2>

**编译**

```sh
go build -o app main.go
```

**使用 `systemd` 运行**

创建 `/etc/systemd/system/iris.service`：

```ini
[Unit]
Description=Iris Web Server
After=network.target

[Service]
ExecStart=/path/to/app
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```
```sh
systemctl start iris
systemctl enable iris
```

<br/><br/>

### **总结**

| **功能** | **Iris 支持** |
|----------|--------------|
| 高性能 | ✅ |
| 路由 | ✅ |
| MVC | ✅ |
| WebSocket | ✅ |
| 中间件 | ✅ |
| 视图模板 | ✅ |
| 依赖注入 | ✅ |
| 数据库集成 | ✅ |

Iris 适用于高性能 Web 应用，是 `Gin` 的有力竞争者。如果你需要更强大的 MVC 结构，Iris 可能是更好的选择！🚀


<br/>

***
<br/><br/><br/>
> <h1 id=""></h1>

要使用好ORM，其核心还是掌握关系型数据库的原理、优化、索引等。受自身项目所限，读者使用基本的SQL功能就能完成任务，如果数据量大了，那么势必会遇到查询缓慢的问题；如果服务失联，那么势必会遇到数据丢失的问题。这些才是关系型数据库使用的核心，读者需要懂得索引的原理、索引的优化，才能设计更加健壮的模型，完成任务的同时使系统具备持续迭代的能力。

建议读者熟练掌握编程语言的同时多研究基础原理，这些原理的掌握才是读者应对繁复变化的核心竞争力。

> <h1 id="Redis">Redis</h1>

Redis（Remote Dictionary Service，远程字典服务）是互联网领域使用广泛的存储中间件，以其高性能和丰富的客户端库在软件领域大放异彩。几乎所有的互联网应用都可以使用Redis，还是以百度热点事件排行榜为例，其后台就可以使用一个有序集合zset来实现，再按分数排名，分数高的热点事件排名靠前；再比如微博的点赞、转发、评论等计数的功能，都可以使用字符串类型的INCR自增功能。

- Redis核心的基本数据结构有5种：
	- 分别为string（字符串）​、list（列表）​、set（集合）​、hash（哈希）和zset（有序集合）​。
	- 熟练掌握这5种基本数据结构的使用至关重要。

Redis提供了一套操作这5种数据结构的命令，包含进行数据的新增、删除、更改、查询等操作。

使用Redis通常采用客户端/服务器端架构的模式，客户端可以连接本地的Redis服务，也可以连接远程的Redis服务，不过我们仍然推荐Redis以容器的形式运行。

<br/>

**下载：**

```
go get github.com/gomodule/redigo/redis
```

<br/><br/><br/>
> <h2 id="GORM和XORM详介">GORM和XORM详介</h2>

**GORM** 和 **XORM** 是 Go 语言中常用的 ORM（对象关系映射）框架，它们都用于操作数据库，但在设计理念、易用性、性能等方面有所不同。

***

- **1. GORM vs XORM 概览**

| 特性         | GORM | XORM |
|-------------|------|------|
| **活跃度** | 🔥 非常活跃（GitHub 49k+ Star） | 🚀 活跃（GitHub 8k+ Star） |
| **使用难度** | 🌟🌟🌟🌟（功能强大，但学习曲线较陡） | 🌟🌟🌟（API 设计更简洁，易上手） |
| **性能** | 🟡 相对较慢（功能丰富，反射较多） | 🟢 更快（基于代码生成优化，较少反射） |
| **数据库支持** | ✅ MySQL、PostgreSQL、SQLite、SQL Server 等 | ✅ 同样支持 MySQL、PostgreSQL、SQLite、SQL Server |
| **自动迁移** | ✅ 内置 `AutoMigrate`（适合快速开发） | ❌ 没有自动迁移 |
| **预加载** | ✅ `Preload()`、`Joins()` | ✅ `Join()` |
| **事务** | ✅ `Begin()`, `Commit()`, `Rollback()` | ✅ `Transaction()` |
| **软删除** | ✅ `gorm.DeletedAt` | ✅ `deleted` 方式 |
| **结构体标签** | `gorm:"column:id;primaryKey"` | `xorm:"'id' pk"` |
| **支持 JSON 结构** | ✅（通过 `json` 解析） | ✅（支持 `JSON` 字段存储） |


<br/><br/>
> <h3 id="GORM使用">GORM使用</h4>

<br/><br/>
> <h3 id="GORM定义模型"> GORM定义模型</h3>

```go
type User struct {
    gorm.Model  // 提供 ID、CreatedAt、UpdatedAt、DeletedAt
    Name  string `gorm:"size:100"`
    Age   int
}
```

<br/><br/>
> <h3 id="GORM初始化数据库"> GORM 初始化数据库 </h3>

```go
db, err := gorm.Open(mysql.Open("root:password@tcp(127.0.0.1:3306)/test"), &gorm.Config{})
if err != nil {
    panic(err)
}

// 自动迁移
db.AutoMigrate(&User{})
```

<br/><br/>
> <h3 id="GORM插入数据"> GORM插入数据 </h3>

```go
user := User{Name: "Alice", Age: 25}
db.Create(&user)
```


<br/><br/>
> <h3 id="GORM查询数据"> GORM查询数据 </h3>

```go
var user User
db.First(&user, "name = ?", "Alice")
```

<br/>

**多条查询**

```go
var users []User
db.Where("age > ?", 20).Find(&users)
```

<br/><br/>
> <h3 id="GORM更新数据">GORM更新数据</h3>

```go
db.Model(&user).Update("age", 30)
```

<br/><br/>
> <h3 id="GORM删除数据">GORM删除数据</h3>

```go
db.Delete(&user)
```


<br/><br/>
> <h3 id="XORM使用"> XORM使用</h4>

<br/><br/>
> <h3 id="XORM定义模型"> XORM定义模型</h3>

```go
type User struct {
    ID    int64  `xorm:"pk autoincr"` // 主键 & 自增
    Name  string `xorm:"varchar(100)"`
    Age   int
}
```

<br/><br/>
> <h3 id="XORM初始化数据库">XORM初始化数据库 </h3>

```go
engine, err := xorm.NewEngine("mysql", "root:password@tcp(127.0.0.1:3306)/test")
if err != nil {
    panic(err)
}

// 创建表
err = engine.Sync2(new(User))
```

<br/><br/>
> <h3 id="XORM插入数据"> XORM插入数据 </h3>

```go
user := User{Name: "Alice", Age: 25}
_, err := engine.Insert(&user)
```


<br/><br/>
> <h3 id="XORM查询数据"> XORM查询数据 </h3>

```go
var user User
_, err := engine.Where("name = ?", "Alice").Get(&user)
```

<br/>

**多条数据查询**

```go
var users []User
err := engine.Where("age > ?", 20).Find(&users)
```

<br/><br/>
> <h2 id="项目实践1-JOIN(select))"> 项目实践1-JOIN(select)) </h2>

```
func (engine *Engine) Join(joinOperator string, tablename interface{}, condition string, args ...interface{}) *Session {
	session := engine.NewSession()
	session.isAutoClose = true
	return session.Join(joinOperator, tablename, condition, args...)
}
```

**上述代码如何使用？详解**

<br/><br/>

- **📌 `Join` 方法详解**

这个 `Join` 方法属于 **`Engine` 结构体**，用于 **执行 SQL JOIN 语句**。它通常用于 ORM（对象关系映射）库，例如 `xorm`。

<br/>

- **🔹 代码解析**

```go
func (engine *Engine) Join(joinOperator string, tablename interface{}, condition string, args ...interface{}) *Session {
	session := engine.NewSession() // 创建一个新的数据库会话
	session.isAutoClose = true     // 设置会话自动关闭
	return session.Join(joinOperator, tablename, condition, args...) // 调用 Session 的 Join 方法
}
```

- **1️⃣ `engine.NewSession()`**
	- **创建一个新的数据库会话**，用于执行 SQL 语句。
	- `engine` 可能是 **ORM 的核心对象**，用于管理数据库连接。

<br/>

- **2️⃣ `session.isAutoClose = true`**
	- **自动关闭数据库会话**，防止连接泄漏。

<br/>

- **3️⃣ `session.Join(joinOperator, tablename, condition, args...)`**
	- **调用 `Session` 的 `Join` 方法**，用于执行 SQL `JOIN` 查询。
	- `joinOperator`：指定 JOIN 类型（如 `"INNER"`, `"LEFT"`, `"RIGHT"`）。
	- `tablename`：要 JOIN 的表名（可能是 `string` 或 `struct`）。
	- `condition`：JOIN 条件，例如 `"a.id = b.user_id"`。
	- `args...`：条件参数（可选）。

<br/>

- **🔹 来自 ORM 库**（一个 Go 语言的 ORM 库）。

在 `xorm` 中，`Join` 方法用于 **执行 SQL JOIN 语句**：

```go
func (session *Session) Join(joinOperator string, tablename interface{}, condition string, args ...interface{}) *Session
```

来自

📌 **示例：使用 `xorm` 执行 `JOIN`**

```go
engine := xorm.NewEngine("mysql", "user:password@/dbname")

var users []User
err := engine.Join("INNER", "order", "user.id = order.user_id").
              Where("user.age > ?", 18).
              Find(&users)

if err != nil {
    log.Fatal(err)
}
```
🔹 **相当于执行的 SQL**：

```sql
SELECT * FROM user
INNER JOIN order ON user.id = order.user_id
WHERE user.age > 18;
```

---

## **📌 作用**
1. **封装 `JOIN` 逻辑**，方便 ORM 进行关联查询。
2. **创建会话并自动关闭**，避免连接泄漏。
3. **用于 SQL 关联查询**（`INNER JOIN`、`LEFT JOIN`、`RIGHT JOIN` 等）。

---

## **📌 总结**
- 这个方法属于 **`xorm` 或类似 ORM 库**。
- 主要作用是 **执行 SQL JOIN 语句**，封装 `Session.Join()` 逻辑。
- **自动管理数据库会话**，防止连接泄漏。
- 适用于 **关联查询**（如 `user` 和 `order` 表的关联）。

🚀 **适合用在 ORM 查询时，简化 SQL JOIN 逻辑！**


<br/><br/>
> <h2 id="项目实践2-where(select)">项目实践2-where(select)</h2>

- **先看项目中的一段代码：**

**database_v1.go文件**

```
var BeeQuickDatabase *xorm.Engine
```

<br/>

**controller.go文件使用**

```go

var vipMember model_v1.VipMember

_, dbErr := database_v1.BeeQuickDatabase.Where("level_name = ?", strings.ToUpper("v0")).Get(&vipMember); dbErr != nil {
		return account, error_v1.ErrorV1{
			Code:    http.StatusBadRequest,
			Message: dbErr.Error(),
			Detail:  "会员等级未存在",
		}
	}
```

**`作用：`xorm ORM** 进行数据库查询，主要用于查询 vipMember 是否存在。

<br/>


- **🔹 代码解析**
	- **📌 `database_v1.go`**

```go
var BeeQuickDatabase *xorm.Engine
```
- 📌 **定义 `BeeQuickDatabase` 作为全局数据库引擎**
	- **`xorm.Engine`** 是 `xorm` ORM 的数据库连接对象。
	- 该变量在 **应用启动时** 进行初始化，并在其他文件中 **直接使用**。

<br/>

- **📌 `controller.go`**

```go
var vipMember model_v1.VipMember

_, dbErr := database_v1.BeeQuickDatabase.Where("level_name = ?", strings.ToUpper("v0")).Get(&vipMember)
if dbErr != nil {
    return account, error_v1.ErrorV1{
        Code:    http.StatusBadRequest,
        Message: dbErr.Error(),
        Detail:  "会员等级未存在",
    }
}
```

- **1️⃣ `database_v1.BeeQuickDatabase.Where("level_name = ?", strings.ToUpper("v0")).Get(&vipMember)`**
	- **查询 `vipMember` 是否存在**：
	  - **`Where("level_name = ?", strings.ToUpper("v0"))`**：查询 `level_name` 是否等于 `"V0"`（转大写）。
	  - **`Get(&vipMember)`**：
	    - **`vipMember` 结构体**会被填充查询结果。
	    - **返回 `true/false`**：如果查询到数据，则返回 `true`，否则返回 `false`。
	
- **2️⃣ `if dbErr != nil {}`**
	- **检查是否发生数据库错误**：
	  - `dbErr != nil` 表示 SQL 执行出错（例如数据库连接失败）。
	  - 返回 `error_v1.ErrorV1` 结构体，表示 **HTTP 400 错误（Bad Request）**。

<br/>

- **🔹 代码示例**
	- 📌 **完整的 `vipMember` 查询示例**

```go
type VipMember struct {
	base      `xorm:"extends"`
	LevelName string `xorm:"varchar(2) notnull unique 'level_name'" json:"level_name"`
	Start     int    `json:"start"`
	End       int    `json:"end"`
	Points    float64
	Comment   string `xorm:"varchar(128) notnull" json:"comment"`
	Period    int    `json:"period"`
	ToValue   int    `json:"to_value"`
}

var vipMember VipMember

// 查询 level_name 为 "V0" 的会员
_, dbErr := database_v1.BeeQuickDatabase.Where("level_name = ?", "V0").Get(&vipMember)

if dbErr != nil {
    fmt.Println("查询失败:", dbErr)
} else {
    fmt.Println("查询成功:", vipMember)
}
```

<br/>

- **📌 代码作用**
	1. **查询 `vipMember` 是否存在**（根据 `level_name`）。
	2. **检查数据库查询是否报错**（`dbErr`）。
	3. **如果出错，则返回 HTTP 400 + 错误信息**。

<br/>

- **📌 总结**
	- **`BeeQuickDatabase`** 是 `xorm.Engine`，用于数据库操作。
	- **`Where(...).Get(&vipMember)`** 查询 `level_name = "V0"` 的数据。
	- **如果查询出错**，返回 `error_v1.ErrorV1`，HTTP 状态码 `400`。
	- 适用于 **会员等级校验** 场景。 🚀


<br/><br/>
> <h2 id="源码Where方法使用">源码Where方法使用</h2>

在 `xorm` 库中，📌 `Where` 方法用于构建 **`SQL WHERE` 查询条件**。它通常与其他查询方法一起使用，以便在数据库中检索符合条件的数据。

```
func (engine *Engine) Where(query interface{}, args ...interface{}) *Session {
	session := engine.NewSession()
	session.isAutoClose = true
	return session.Where(query, args...)
}
```

<br/>

- **1️⃣ `engine.NewSession()`**
	- **`NewSession()`** 创建一个新的数据库会话。
	- 在 `xorm` 中，`Session` 对象代表与数据库的连接，用于执行 SQL 查询和事务。

- **2️⃣ `session.isAutoClose = true`**
	- 设置 `isAutoClose` 为 `true`，表示这个会话在使用完后会自动关闭，以避免数据库连接泄漏。

- **3️⃣ `session.Where(query, args...)`**
	- **调用 `session.Where` 方法**，用于构建 **`WHERE` 子句**。
	- **`query`**：是 `WHERE` 条件的字符串（如 `"age > ?"`），或者可以是带有条件字段的结构体。
	- **`args...`**：是查询条件的参数（如 `"30"`，代表查询大于 30 的记录）。

- **4️⃣ 返回值：** `*Session`
	- 返回一个 **`Session` 对象**，你可以继续调用 `Find`、`Get` 等方法进行查询。

<br/>

- **🔹 代码示例**
	- **查询大于某个年龄的会员**

```go
type Member struct {
    ID   int64  `xorm:"id"`
    Name string `xorm:"name"`
    Age  int    `xorm:"age"`
}

var members []Member

// 查询年龄大于 30 的会员
err := engine.Where("age > ?", 30).Find(&members)
if err != nil {
    log.Fatal(err)
}
fmt.Println(members)
```

📌 **相当于执行的 SQL**：

```sql
SELECT * FROM member WHERE age > 30;
```

<br/>

- **使用结构体进行查询**

```go
var member Member

// 根据 ID 查询会员
_, err := engine.Where("id = ?", 1).Get(&member)
if err != nil {
    log.Fatal(err)
}
fmt.Println(member)
```

---

- **🔹 作用**
	1. **构建 WHERE 查询条件**，用于数据库查询。
	2. **自动创建会话**，并确保在查询完成后关闭会话，避免连接泄漏。
	3. **支持参数化查询**，通过 `args...` 可以传递查询条件的值。
	4. **链式调用**，可以继续对 `Session` 对象进行进一步的查询或修改。

<br/>

- **🔹 总结**
	- `Where` 方法用于 **构建 SQL 查询中的 WHERE 条件**。
	- 通过 **`NewSession()` 创建会话**，并通过 `session.Where` 进行查询。
	- **自动关闭会话**，避免数据库连接泄漏。
	- **支持结构体和参数化查询**，简化 SQL 的构造过程。




<br/><br/>
> <h3 id="XORM更新数据">XORM更新数据</h3>

```go
user.Age = 30
_, err := engine.ID(user.ID).Update(&user)
```


<br/><br/>
> <h3 id="XORM删除数据">XORM删除数据</h3>

```go
_, err := engine.ID(user.ID).Delete(&user)
```


<br/><br/><br/>
> <h2 id="主要区别">主要区别</h2>

- **1.GORM 更加面向对象**
	- GORM 允许基于 **模型结构体** 自动迁移，使用 **链式调用**（如 `db.Model().Where().Update()`）。
	- 适合 DDD（领域驱动设计）和复杂查询。

- **2.XORM 更轻量**
	- XORM 使用 **函数式 API**，类似 SQL，写法直观，支持 **数据库反向生成结构体**。
	- 适合 **简单 CRUD**，适合 **高性能应用**。

- **3.性能**
	- **XORM** 的性能更好，因为它减少了 **反射** 和 **自动迁移** 带来的开销。
	- **GORM** 提供了更多特性，但牺牲了一定的性能（如 `AutoMigrate` 会影响数据库查询速度）。

<br/><br/><br/>
> <h2 id="什么时候选用？">什么时候选用？</h2>

| 使用场景 | 选择 GORM | 选择 XORM |
|----------|-----------|-----------|
| 需要复杂查询 | ✅ | 🚫 |
| 需要自动迁移 | ✅ | 🚫 |
| 追求极致性能 | 🚫 | ✅ |
| 快速开发 MVP | ✅ | ✅ |
| 适合新手 | 🚫（较多概念） | ✅（API 直观） |
| 需要 JSON 数据存储 | ✅ | ✅ |

---

### **. 总结**
- **GORM** 适合 **功能丰富** 的场景，比如 Web 框架（Gin、Fiber），尤其是需要 **自动迁移、事务管理、关联查询**。
- **XORM** 适合 **高性能、轻量级** 的场景，比如需要直接映射数据库查询，或者 Golang 微服务。

如果你是 **新手**，XORM 的上手难度低，但如果你需要 **复杂业务逻辑**，GORM 是更好的选择。🚀


<br/><br/><br/>
> <h2 id="model序列化和继承">model序列化和继承</h2>

**base_model.go文件如下：**

```type base struct {
	ID        uint      `xorm:"pk autoincr notnull 'id'" json:"id"`
	CreatedAt time.Time `xorm:"created" json:"created_at"`
	UpdatedAt time.Time `xorm:"updated" json:"updated_at"`
	DeletedAt time.Time `xorm:"deleted" json:"deleted_at"`
}
```

<br/>

**account_model.go文件中有如下代码：**

```
type Account struct {
	base     `xorm:"extends"`
	Phone    string    `xorm:"varchar(11) notnull unique 'phone'" json:"phone"`
	Password string    `xorm:"varchar(128)" json:"password"`
	Token    string    `xorm:"varchar(128) 'token'" json:"token"`
	Avatar   string    `xorm:"varchar(128) 'avatar'" json:"avatar"`
	Gender   string    `xorm:"varchar(1) 'gender'" json:"gender"`
	Birthday time.Time `json:"birthday"`

	Points      int       `json:"points"`
	VipMemberID uint      `xorm:"index"`
	VipMember   VipMember `xorm:"-"`
	VipTime     time.Time `json:"vip_time"`
}

type VipMember struct {
	base      `xorm:"extends"`
	LevelName string `xorm:"varchar(2) notnull unique 'level_name'" json:"level_name"`
	Start     int    `json:"start"`
	End       int    `json:"end"`
	Points    float64
	Comment   string `xorm:"varchar(128) notnull" json:"comment"`
	Period    int    `json:"period"`
	ToValue   int    `json:"to_value"`
}

type AccountGroupVip struct {
	Account   `xorm:"extends"`
	VipMember `xorm:"extends"`
}



func (AccountGroupVip) TableName() string {
	return "beeQuick_account"
}

func (a AccountGroupVip) SerializerForGroup() 

AccountSerializer {
	result := a.Account.Serializer()
	result.VipMember = a.VipMember.Serializer()
	return result
}
```

上述代码使用 **Go 语言** 结合 **XORM** ORM 库定义了数据库模型，其中 `base_model.go` 定义了基础模型，`account_model.go` 主要定义了账户相关的结构体。  

<br/>

- **1.`base_model.go` (基础模型)**

```go
type base struct {
	ID        uint      `xorm:"pk autoincr notnull 'id'" json:"id"`
	CreatedAt time.Time `xorm:"created" json:"created_at"`
	UpdatedAt time.Time `xorm:"updated" json:"updated_at"`
	DeletedAt time.Time `xorm:"deleted" json:"deleted_at"`
}
```
- **解析：**
	- `ID`：主键，自动递增，不允许为空 (`pk autoincr notnull`)，XORM 会将其映射到 `id` 列。
	- `CreatedAt`：记录创建时间 (`xorm:"created"`，XORM 会自动填充)。
	- `UpdatedAt`：记录更新时间 (`xorm:"updated"`，XORM 会在更新时自动填充)。
	- `DeletedAt`：记录删除时间 (`xorm:"deleted"`，用于软删除，即不会真正删除记录)。

<br/>

- **2.`account_model.go` 账户模型**
	- **2.1`Account` (用户账户)**

```go
type Account struct {
	base     `xorm:"extends"`
	Phone    string    `xorm:"varchar(11) notnull unique 'phone'" json:"phone"`
	Password string    `xorm:"varchar(128)" json:"password"`
	Token    string    `xorm:"varchar(128) 'token'" json:"token"`
	Avatar   string    `xorm:"varchar(128) 'avatar'" json:"avatar"`
	Gender   string    `xorm:"varchar(1) 'gender'" json:"gender"`
	Birthday time.Time `json:"birthday"`

	Points      int       `json:"points"`
	VipMemberID uint      `xorm:"index"`
	VipMember   VipMember `xorm:"-"`  // 仅在 Go 代码中存在，不映射到数据库
	VipTime     time.Time `json:"vip_time"`
}
```
- **解析：**
	- **继承 `base`**：
		  - 通过 `xorm:"extends"`，`Account` 结构体继承了 `base` 结构体中的字段，如 `ID`、`CreatedAt`、`UpdatedAt`、`DeletedAt`。
	- **账户信息字段**：
		  - `Phone`：手机号，`varchar(11)`，必须唯一。
		  - `Password`：密码，`varchar(128)`，存储加密后的密码。
		  - `Token`：认证 Token。
		  - `Avatar`：头像 URL。
		  - `Gender`：性别，`varchar(1)`，通常存储 `M`（男）或 `F`（女）。
		  - `Birthday`：生日，时间类型。
	- **会员信息字段**：
		  - `Points`：积分，`int`。
		  - `VipMemberID`：外键，指向 `VipMember` 的 `ID`，并创建索引 (`xorm:"index"`)。
		  - `VipMember`：会员等级信息，`xorm:"-"` 说明它不在数据库中，而是通过 `VipMemberID` 关联后填充。
		  - `VipTime`：VIP 到期时间。
	
<br/>

- **2.2`VipMember` (会员等级)**

```go
type VipMember struct {
	base      `xorm:"extends"`
	LevelName string  `xorm:"varchar(2) notnull unique 'level_name'" json:"level_name"`
	Start     int     `json:"start"`
	End       int     `json:"end"`
	Points    float64
	Comment   string  `xorm:"varchar(128) notnull" json:"comment"`
	Period    int     `json:"period"`
	ToValue   int     `json:"to_value"`
}
```
- **解析：**
	- **继承 `base`**：同样继承了 `ID`、`CreatedAt`、`UpdatedAt`、`DeletedAt`。
	- **会员等级字段**：
		  - `LevelName`：等级名称（如 `V1`、`V2`）。
		  - `Start`、`End`：该会员等级的积分区间，例如 `V1` 可能是 `[0, 100]`，`V2` 是 `[101, 500]`。
		  - `Points`：该等级默认积分加成（如消费 1 元增加的积分）。
		  - `Comment`：描述信息，如 "黄金会员"、"白金会员"。
		  - `Period`：会员有效期（单位：天）。
		  - `ToValue`：达到此值后自动升级下一级。

<br/>
- **2.3 `AccountGroupVip` (账户和会员的组合)**

```go
type AccountGroupVip struct {
	Account   `xorm:"extends"`
	VipMember `xorm:"extends"`
}
```

- **解析：**
	- 这个结构体**继承**了 `Account` 和 `VipMember`，它是一个虚拟结构体，方便查询时直接获取 `Account` 和 `VipMember` 的所有字段。
	- 适用于联合查询，例如：

```go
var results []AccountGroupVip
err := engine.Table("beeQuick_account").Find(&results)
```
  
这样，查询出的 `results` 既包含 `Account` 也包含 `VipMember` 相关信息。

<br/>

- **2.4`TableName()` 方法**

```go
func (AccountGroupVip) TableName() string {
	return "beeQuick_account"
}
```

- 这个方法告诉 XORM **表名不是 `account_group_vip`，而是 `beeQuick_account`**，避免自动生成不合适的表名。

<br/>

- **2.5`SerializerForGroup()` 方法**

```go
func (a AccountGroupVip) SerializerForGroup() AccountSerializer {
	result := a.Account.Serializer()
	result.VipMember = a.VipMember.Serializer()
	return result
}
```

- `SerializerForGroup()` 用于序列化 `AccountGroupVip`，其中：
	  - `a.Account.Serializer()` 处理 `Account` 结构的序列化。
	  - `a.VipMember.Serializer()` 处理 `VipMember` 结构的序列化。
	  - 返回 `AccountSerializer`（假设 `AccountSerializer` 是一个结构体，专门用于 API 响应）。

<br/>

- **总结**
	- **`base` 结构体** 提供通用的 ID 和时间戳字段，多个模型都可以继承它。
	- **`Account` 结构体** 代表用户账户，包括手机号、密码、Token、会员信息等。
	- **`VipMember` 结构体** 代表会员等级，包括等级名称、积分范围、特权等。
	- **`AccountGroupVip` 结构体** 是 `Account` 和 `VipMember` 的组合，适用于查询联合信息。
	- **`SerializerForGroup()`** 用于格式化查询结果，适用于 API 返回 JSON 数据。

<br/>

- **这个设计方式：**
	1. **提高了代码复用性**（通过 `xorm:"extends"` 继承 `base`）。
	2. **避免手动维护时间字段**（`xorm:"created"`、`xorm:"updated"` 自动填充）。
	3. **支持软删除**（`xorm:"deleted"`，不会真正删除数据）。
	4. **使用 `xorm:"-"` 让 `VipMember` 仅用于 Go 代码，不影响数据库结构**。
	5. **便于 JSON 序列化**，API 可以直接返回格式化的数据。

这样，你可以很方便地进行 CRUD 操作，例如：

```go
var account Account
engine.Where("phone = ?", "12345678901").Get(&account)
```


<br/>

***
<br/><br/><br/>
> <h1 id="crypto库"> crypto库 </h1>

<br/><br/><br/>
> <h2 id="crypto哈希">crypto哈希</h2>

- **📌 `GenerateFromPassword` 方法详解**

这是 Go 语言中 `crypto` 库中的一个常用方法，通常用于 **密码的哈希加密**。在 Go 标准库中，它属于 `bcrypt` 密码加密算法的一部分。此方法的目的是通过一个密码和工作因子（`cost`）生成密码哈希。

<br/>
- **`GenerateFromPassword` 方法源码：**

```go
func GenerateFromPassword(password []byte, cost int) ([]byte, error) {
    p, err := newFromPassword(password, cost)
    if err != nil {
        return nil, err
    }
    return p.Hash(), nil
}
```

- **🔹 代码解析**

	- **1️⃣ `newFromPassword(password, cost)`**

		- **`newFromPassword`** 是一个辅助函数，它将密码和工作因子（`cost`）作为输入，生成一个加密的哈希结构。
		- **`password []byte`**：待加密的密码。
		- **`cost int`**：工作因子，它决定了加密过程的复杂度（即计算成本）。较大的 `cost` 值会导致更高的计算负担，从而提高安全性。通常，`cost` 的推荐值是 `10`，但可以根据需要调整。
		  - **`cost`** 决定了 bcrypt 哈希计算的时间复杂度。较高的 `cost` 会让加密更加耗时，从而在暴力破解时增加困难。

	- **2️⃣ `if err != nil { return nil, err }`**
		- **错误检查**：如果 `newFromPassword` 在生成哈希时发生了错误（例如，传递了无效的 `cost` 值），它会返回一个错误。此时方法会 **返回 `nil` 和错误**。

- **3️⃣ `return p.Hash(), nil`**
	- **`p.Hash()`**：`p` 是通过 `newFromPassword` 创建的哈希结构，`Hash()` 方法会返回最终生成的密码哈希。这个哈希将作为返回值传递出去。
	- **返回值**：生成的哈希和 `nil` 错误表示成功。

<br/>

- **🔹 主要功能**
	- **密码哈希生成**：该方法用于 **生成加密后的密码哈希**。生成的哈希通常用于存储在数据库中，以便在用户登录时对比。
	- **`cost` 控制复杂度**：`cost` 是一个重要的参数，它控制了 bcrypt 加密算法的 **计算开销**，从而影响加密的速度和安全性。较高的 `cost` 值增加了攻击者在进行暴力破解时所需的时间。

<br/>
- **🔹 代码示例**

假设我们在 Go 中使用此函数来加密用户密码并存储它：

```go
package main

import (
    "fmt"
    "golang.org/x/crypto/bcrypt"
    "log"
)

func GenerateFromPassword(password []byte, cost int) ([]byte, error) {
    p, err := newFromPassword(password, cost)
    if err != nil {
        return nil, err
    }
    return p.Hash(), nil
}

func main() {
    password := []byte("mySecretPassword")
    cost := 10 // 工作因子
    
    // 生成哈希
    hashedPassword, err := GenerateFromPassword(password, cost)
    if err != nil {
        log.Fatal("Error generating hash:", err)
    }
    
    fmt.Printf("Generated Hash: %s\n", hashedPassword)
    
    // 使用 bcrypt 对比密码（假设用户输入的密码是 "mySecretPassword"）
    err = bcrypt.CompareHashAndPassword(hashedPassword, password)
    if err != nil {
        fmt.Println("Password mismatch")
    } else {
        fmt.Println("Password matched")
    }
}
```

- **执行结果**：
	- 当用户第一次注册时，我们会通过 `GenerateFromPassword` 生成哈希密码，并将其存储。
	- 用户登录时，我们使用 `bcrypt.CompareHashAndPassword` 来比较用户输入的密码与存储的哈希密码是否匹配。













