
- [AI集合模型ollama](#AI集合模型ollama)
- [OpenAI的CodeX](#OpenAI的CodeX)
- [CodeX-CLI使用](#CodeX-CLI使用)
	- [安装Cli](#安装Cli)
	- [了解工程当前状态](#工程当前状态)
	- [进入工程目录并启动](#进入工程目录并启动)
	- [打开某个类文件修改某个工功能](#打开某个类文件修改某个工功能)
	- [终止当前提问](#终止当前提问)
		- [Ctrl+C停止交互，重启会话](#Ctrl+C停止交互，重启会话) 
		- [停止当前会话，继续重新提问](#停止当前会话，继续重新提问)
- [OpenCode](#OpenCode)
- [快马-inscode](https://inscode.net)


<br/><br/><br/>

***
<br/>

> <h1 id= "AI集合模型ollama">AI集合模型ollama</h1>


[**安装页面**](https://ollama.com/search) 


<br/><br/><br/>

***
<br/>

> <h1 id= "OpenAI的CodeX">OpenAI的CodeX</h1>
- [下载URL](https://chatgpt.com/codex)
- [使用指南【法典】](https://developers.openai.com/codex/learn/best-practices)

**在大型或复杂的仓库工作，最大的解锁是给 Codex 提供正确的任务背景和清晰的结构。**

- 一个好的默认做法是在提示中包含四点：
 - Goal【目标】： 你想改变或建设什么？
 - Context【上下文】： 哪些文件、文件夹、文档、示例或错误对这项任务很重要？你可以@提及某些文件作为上下文。
 - Constraints【限制条件】：Codex 应遵循哪些标准、架构、安全要求或惯例？
 - Done when【完成时间】： 任务完成前应实现哪些情况，比如测试通过、行为改变或某个漏洞不再复制？

<br/>


- **推理层级**，测试最适合你工作流程的方法。不同用户和任务配合不同设置效果最佳。
 - 低，适合更快、范围明确的任务
 - 中等或高，用于更复杂的更改或调试
 - Extra High 适合长时间、行动性强、推理繁重的任务

<br/>

- **先为困难任务做规划**
 - 如果任务复杂、模糊或难以描述，请 Codex 在开始编码前先做计划。
 - 有几种方法效果不错：
  - 使用计划模式： 对大多数用户来说，这是最简单且最有效的选择。计划模式允许 Codex 收集背景信息，提出澄清问题，并在实施前制定更强有力的计划。用 `/plan`或 `Shift + Tab` 键切换。
  - 请 Codex 采访你： 如果你对自己想要什么有个大致想法，但不确定如何好好描述，先让 Codex 问你问题。告诉它挑战你的假设，把模糊的想法变成具体的东西，然后再写代码。
  - 使用 PLANS.md 模板： 对于更高级的工作流程，你可以配置 `Codex 遵循 PLANS.md` 或执行计划模板，用于较长时间或多步骤的工作。更多细节请参见执行 [计划指南](https://developers.openai.com/cookbook/articles/codex_exec_plans)。



<br/><br/><br/>

***
<br/>

> <h1 id= "CodeX-CLI使用">CodeX-CLI使用</h1>

<br/><br/><br/>
> <h2 id= "安装Cli">安装Cli</h2>

官方给的安装方式包括：

```sh
npm i -g @openai/codex
```

或者

```sh
brew install --cask codex
```


***
<br/><br/><br/>
> <h2 id="工程当前状态">工程当前状态</h2>

- `Explored → Codex` 扫描工程，了解目录和文件。
- `Ran git status --short -- <path>` → 查看指定路径文件的 Git 状态。
- `Ran git diff -- <file>` → 查看指定文件的具体修改内容。

***
<br/><br/><br/>
> <h2 id="进入工程目录并启动">进入工程目录并启动</h2>

```sh
cd /Users/huanggang/HGFiles/Code/GitLab/xxxx工程名 && codex

// 或者
cd /Users/huanggang/HGFiles/Code/GitLab/xxxx工程名 
codex
```

进去后，你会进入全屏交互模式（终端 UI），Codex 会读取整个工程，理解结构和类之间关系。

<br/>

- **第一次进入后，可以直接让它做什么**

进入后你可以直接输入类似这些话：

```sh
先帮我理解这个 iOS 工程的整体结构
```

```sh
分析一下这个项目的登录流程在哪几个文件里
```

```sh
帮我找到局域网配网相关代码入口
```

```sh
解释一下 argus-app-ios 里网络层是怎么封装的
```

```sh
帮我搜索 BLE 配网相关类，并总结调用链
```

Codex CLI 的交互模式就是让你在当前仓库里边聊边改。


<br/>

- **带着问题一起打开**

可以直接把问题跟在命令后面：

```sh
cd /Users/huanggang/HGFiles/Code/GitLab/argus-app-ios
codex "先帮我理解这个 iOS 项目的目录结构和关键模块"
```


***
<br/><br/><br/>
> <h2 id="打开某个类文件修改某个工功能">打开某个类文件修改某个工功能</h2>

在 Codex CLI 里，目前没有像 **Codex App** 那样可以直接“拖文件进去”的功能。CLI 的操作仍然是基于当前工程目录和命令行交互的方式。官方文档明确说明：

* CLI 会读取当前目录的所有文件，交互模式可以直接修改、添加、分析代码。
* 你可以一次性打开整个工程，然后用自然语言让 Codex 定位和修改任意文件。
* CLI 没有图形化拖拽功能，快捷方式是用命令或者在启动时带初始提示。([developers.openai.com](https://developers.openai.com/codex/cli/features/?utm_source=chatgpt.com))

<br/>

- **快捷方法替代“拖文件”**

- **1.直接指定文件路径启动（类似一次性打开某个文件）：**

```bash
codex "请打开 BluetoothDetectUtil.swift 文件"
```

<br/>

**2.在 CLI 里记住上一次操作的文件**
   交互模式中，Codex 会记住你上一次编辑的文件，再次提到方法或属性时无需重复指定文件名。你可以说：

```text
修改 scanNearbyDevices 方法，加日志和过滤逻辑
```

CLI 会自动在上一次操作的文件里查找该方法。

<br/>

**3.全工程打开 + 自然语言定位**

   * `cd` 到工程根目录
   * `codex` 进入交互模式
   * 一次性说：“帮我修改 BluetoothDetectUtil.swift 的 scanNearbyDevices 方法”，它会定位文件并修改。

> ⚠️ 总结：CLI 没有拖拽文件功能，但可以通过初始提示或记忆上次操作来模拟快捷打开文件。


***
<br/><br/><br/>
> <h2 id="终止当前提问">终止当前提问</h2>

- **Codex CLI，终止当前运行**
	- 如果是命令行交互中，按 `Ctrl + C` 可以立即停止当前请求。

- 重新提问
	- 等待提示符恢复后，直接输入新的问题即可。


<br/><br/>
> <h3 id="Ctrl+C停止交互，重启会话">Ctrl+C停止交互，重启会话</h3>

在 **Codex CLI** 中使用 `Ctrl+C` 停止了当前请求，会话没有完全关闭，而是被标记为“已停止”。所以 CLI 给出了提示：

```
To continue this session, run codex resume 019d65d5-027b-7f61-b7e5-9b6010f37bbb
```

这意味着你的会话 ID 是：

```
019d65d5-027b-7f61-b7e5-9b6010f37bbb
```

---
<br/>

继续提问

**1.恢复会话**

在终端里输入：

```bash
codex resume 019d65d5-027b-7f61-b7e5-9b6010f37bbb
```

运行后，你会回到原来的交互状态，之前的上下文依然保留。

<br/>

**2. 重新提问**

恢复会话后，就像平常一样在命令行输入新的问题即可。
Codex 会把新问题理解为 **同一会话的延续**，你之前的上下文仍然可用。

---
<br/>

**提示**

* 如果你想完全 **新开一个会话**（清除之前的上下文），直接执行：

```bash
codex new
```

然后输入你的新问题即可。

<br/>

* 如果你不确定会话 ID，可以用：

```bash
codex sessions
```

列出所有会话及状态（active、stopped 等）。


<br/><br/>
> <h3 id="停止当前会话，继续重新提问">停止当前会话，继续重新提问</h3>

不必一定等回答完成，也不必只用 `Ctrl+C`。在 **Codex CLI** 中，你可以立刻停止当前会话并立即重新提问，有几种方式：

---

1️⃣ **使用快捷终止命令**

* **Ctrl+C** 是最常用的，但也可以用 Codex 提供的命令：

```bash
codex stop
```

> 会立即停止当前正在处理的请求（不管是否完成输出）。

* 停止后，会话仍存在，你可以选择：

  * **继续会话**：

    ```bash
    codex resume <session_id>
    ```
  * **新开会话**（清除上下文）：

    ```bash
    codex new
    ```

---
<br/>

**2️⃣ 直接新开会话提问**

如果你想完全忽略错误提问，不关心原上下文：

```bash
codex new
```

* 会创建一个全新的会话
* 输入你的新问题，Codex 会立即处理
* 不用等待前一个回答完成，也不用 `Ctrl+C`，只是前一个请求会继续在后台执行，或者你可以先用 `codex stop` 确保终止

---
<br/> 

**3️⃣ 总结流程**

| 场景       | 命令/操作                       | 说明         |
| -------- | --------------------------- | ---------- |
| 立刻停止当前回答 | `Ctrl+C` 或 `codex stop`     | 当前请求立即终止   |
| 继续旧会话    | `codex resume <session_id>` | 上下文保留，接着提问 |
| 开新会话     | `codex new`                 | 清空上下文，重新提问 |









<br/><br/><br/>

***
<br/>

> <h1 id= "OpenCode">OpenCode</h1>
[下载和配置：](https://opencode.ai/zh/download)



