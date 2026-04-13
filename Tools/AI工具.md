
- [AI集合模型ollama](#AI集合模型ollama)
- [OpenAI的CodeX](#OpenAI的CodeX)
- [CodeX-CLI使用](#CodeX-CLI使用)
	- [安装Cli](#安装Cli)
	- [CLI组合快捷键](#CLI组合快捷键)
	- [常用命令](#常用命令)
	- [提示词实例](#提示词实例) 	
	- [工程当前状态](#工程当前状态)
	- [进入工程目录并启动](#进入工程目录并启动)
	- [打开某个类文件修改某个工功能](#打开某个类文件修改某个工功能)
	- [终止当前提问](#终止当前提问)
		- [Ctrl+C停止交互，重启会话](#Ctrl+C停止交互，重启会话)
  			- [恢复会话](#恢复会话) 
		- [停止当前会话，继续重新提问](#停止当前会话，继续重新提问)
  	- [AGENTS.md记忆](#AGENTS.md记忆)
  	- [快捷命令](#快捷命令)
	- [网络搜索](#网络搜索)
 	- [配置MCP](#配置MCP) 
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
> <h2 id="CLI组合快捷键">CLI组合快捷键</h2>

| 组合快捷键      | 中文说明                       |
|------------|-------------------------------|
| Ctrl + J     | 在大多数终端里插入新行            |






***
<br/><br/><br/>
> <h2 id="常用命令">常用命令</h2>

```sh
# 非交互TUI模式，进入聊天模式
codex

# 带初始提示启动
codex "fix lint errors"


# 非交互式自动化模式, 直接通过 exec直接执行命令，当命令启动
codex exec "帮我写一个坦克大战的游戏开发需 求文档，要求500"

# 沙盒自动模式,不会对你本地的操作系统产生影响，使用单独的环境，比如跑到一个虚拟机或者容器中运行
codex --full-auto "crate the faaciest todo-list app"


# 非沙盒自动模式
codex --dangerously-bypass-approvals-and-sanbox "crate the faaciest todo-list app"


# 引用图片，codex -i [文件路径或者拷贝的图片] ["提问的内容用引号引起来，否则会cli认为是多个参数"]
codex -i /Users/harleyhuang/Desktop/CodeX_Test/2.png "分析这张图片"

```


***
<br/><br/><br/>
> <h2 id="提示词实例">提示词实例</h2>

| ✨ 提示词 | 功能 |
|-----------|------|
| codex "Refactor the Dashboard component to React Hooks" | Codex 重写了类组件，运行了 npm test，并显示了差异 |
| codex "Generate SQL migrations for adding a users table" | 推断你的 ORM，创建迁移文件，并在沙盒数据库中运行它们 |
| codex "Write unit tests for utils/date.ts" | 生成测试，执行测试，并反复迭代直到通过 |
| `codex "Bulk-rename *.jpeg -> *.jpg with git mv"` | 安全地重命名文件并更新导入 / 使用 |
| codex "Explain what this regex does: ^(?=.*[A-Z]).{8,}$" | 输入一步步的解释 |
| codex "Carefully review this repo, and propose 3 high impact well-scoped PRs" | 建议在当前代码库中提出有影响力的 PR |
| **备注** | 测试完之后做一个部署啊 |

当然后面的英文可以换成中文进行说明，其实就是交互式聊天




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


***
<br/><br/>
> <h3 id="恢复会话">恢复会话</h3>

恢复历史会话

* `codex resume`
  主控的人 `codex resume`，会弹出会话选择器，允许你选择要恢复的历史会话[1]。

* `codex resume --last`
  恢复最近的一次会话[1]。

* `codex resume <ID>`
  通过会话 ID 恢复某个特定会话。会话 ID 可以通过 `/status` 命令或在 `~/.codex/sessions/` 目录下查找[1]。

示例命令：

```sh
codex resume
codex resume --last
codex resume 719xxxxxx
```

你直接可以使用这个 Codex resume

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




***
<br/><br/><br/>
> <h2 id="AGENTS.md记忆">AGENTS.md记忆</h2>

你问的 **AGENTS** 可能有几个语境，我给你详细梳理下，并结合现代 AI / 软件开发环境来讲它的作用和用途。

---

## 1️⃣ 在 AI / ChatGPT 语境下的 Agents

在 OpenAI 或类似 AI 平台里，**Agent**（智能代理）指的是一种“可以自主执行任务的程序”，通常具备以下特点：

* **能自主决策**：根据输入和目标自己选择下一步操作，而不是单纯回答用户问题。
* **能执行多步骤任务**：不仅生成文本，还能调用 API、查找信息、处理文件等。
* **可集成工具**：Agent 可以使用浏览器、数据库、计算器、云存储等外部工具。

**用途示例**：

* 预定航班：用户输入“帮我订下周从上海到北京的机票”，Agent 会自己查航班、对比价格、填写信息。
* 数据分析：上传 CSV 文件，Agent 自动读取、分析、生成图表报告。
* 自动化办公：Agent 可以帮你整理邮件、生成周报、自动回复客户。

🔹 总结：在 AI 语境里，**Agents 是会“动手”的 AI，能完成连续任务，不只是单条问答**。

---

## 2️⃣ 在编程 / 系统设计中的 Agents

在软件工程中，Agent 常指“软件代理”或“后台服务”，特点是：

* **独立运行**：通常在后台运行，不依赖用户直接交互。
* **执行特定任务**：比如监控系统状态、同步数据、接收指令并处理。
* **可被调度或调用**：比如操作系统中的守护进程（daemon）、IoT 设备中的智能控制模块。

**用途示例**：

* **网络爬虫 Agent**：定时抓取网页数据。
* **IoT Agent**：监控传感器数据并触发动作。
* **自动化运维 Agent**：检查服务器健康状态、自动修复问题。

🔹 总结：在软件环境里，**Agent 是自动化执行任务的“智能后台程序”**。


***
<br/>

**AGENTS.md 记忆**

AGENTS.md 记忆功能类似 Claude Code 的 CLAUDE.md，让 AI 记住项目特定的指导：

```text
# 全局配置
~/.codex/AGENTS.md

# 项目配置
./AGENTS.md

# 子目录配置
./components/AGENTS.md
```

你可以在 codex 中输入 `/init` 提示，让 codex 帮你生成初始的 AGENTS.md 文件。

CodeX 中的 `AGENTS.md` 是一个简单又开放的格式，专门用来指导 CodeX 更好的干活。

可以把 `AGENTS.md` 想象成是给 Agents 准备的 README，它提供了一个专门的、可预测的地方，用来提供上下文和指令，帮助 AI 编码 Agents 更好地完成你的项目。

详细介绍: [https://agents.md/](https://agents.md/)
好然后是 agents.md 啊


***
<br/><br/><br/>
> <h2 id="快捷命令">快捷命令</h2>

**基础命令**

| 命令       | 中文说明                       | 备注/用法示例 |
|------------|-------------------------------|----------------|
| /model     | 切换模型和推理等级             | `/model gpt-5-mini` |
| /approvals | 设置授权模式【编辑的时候授权，读是不想要的】                   | `/approvals enable` |
| /new 或者 /clear      | new是开启新的会话， clear是清除并开启新的对话                   | `/new` |
| /init      | 初始化 AGENTS.md 指导文件       | `/init` |
| /compact   | 上下文压缩，避免触发上下文限制 | `/compact` |
| /diff      | 显示 git 差异                  | `/diff LocalPods/ArgusAppHome` |
| /mention   | 引用某个文件                   | `/mention BluetoothDetectUtil.swift` |
| /status    | 显示当前会话配置和 Token 用量  | `/status` |

---
<br/>

**高级命令示例**

- **初始化 AGENTS.md 并生成 Swift 模块模板**：
```bash
/init
````

<br/>

* **查看当前会话状态和 Token 使用量**：

```bash
/status
```

<br/>

* **对比代码与 Git 差异**：

```bash
/diff LocalPods/ArgusAppHome/Classes/Utils/BluetoothDetectUtil.swift
```

<br/>

* **引用特定文件内容辅助 Codex 生成代码**：

```bash
/mention BluetoothDetectUtil.swift
```

<br/>

* **压缩上下文以节约 Token 或避免上下文溢出**：

```bash
/compact
```

* **开启新会话，清空上下文**：

```bash
/new
```

---
<br/>

**使用建议**

1. 每次修改或新增 Agent 模块后，可以先执行 `/init` 或 `/mention` 来同步文档和代码。
2. 使用 `/status` 定期监控 Token 用量，避免消耗过快。
3. 在大型工程中，使用 `/diff` 辅助 Codex 查看变动与 Git 差异，保证安全提交。
4. `/compact` 用于复杂会话或大文件处理，避免超出上下文限制。


***
<br/><br/>
> <h3 id="网络搜索">网络搜索</h3>

有了 Web search 网络搜索，模型就能访问互联网的最新信息，让模型在生成回复之前，先上网搜搜最新的信息，并提供带有出处的答案。

想启用这个功能，传递 `--search` 参数即可：

```
codex --search

可以，这条命令会启动带联网搜索能力的 Codex 交互会话。
  进入后直接输入你的问题即可，不用再写 codex --search。


› 今天上海宝山天气，会不会下雨？哪些地方下雨了
```

这样可以仅使用网络搜索，而避免给 agent 完全不受限制的网络访问权限。

***
<br/><br/><br/>
> <h2 id="配置MCP">配置MCP</h2>

**什么是 “Codex 的 MCP”？**

**MCP = Model Context Protocol（模型上下文协议）**
它是一种开放的协议标准，用来让智能助理（像 Codex、Claude、Cursor 等 AI agent）**真正“操作外部工具和数据”**，而不仅仅是在内部回答文字问题。([FastAccess AI][1])

可以把它想象成：

📌 **AI 的 USB 接口**
就像电脑可以通过 USB 接鼠标、键盘、打印机一样，**MCP 是 AI 与外部系统/服务连接的标准桥梁**。通过 MCP，AI 能够：

* 自动操作浏览器、抓取网页
* 读取/写入数据库
* 调用第三方 API
* 操作 Git 仓库、执行代码、部署
* 访问文件系统或日志

而不是仅靠内置的“知识和推理”。([FastAccess AI][1])

> **核心总结：MCP 不是 Codex “内部功能”，而是扩展它能力的一个协议标准。**

---

## 🧠 二、MCP 对 Codex 的意义是什么？

### 🟩 没有 MCP 的时候

Codex 只能：

✔ 根据你的 prompt 生成代码
✔ 阅读静态文件/上下文
✔ 给出建议

但 **无法自动执行真实操作**（例如真正去调用 API、自动提交 Git PR、打开浏览器）。它只能告诉你要怎么做，但不能去做。([Vibe Sparking AI][2])

---

### 🟦 有了 MCP 之后

就像给 Codex 装上了“手”和“脚”，它可以：

✅ 自动执行现实操作（调用工具、API、数据库）
✅ 更智能地集成 CI/CD、代码仓库、日志、搜索工具
✅ 完成真实工作流，而不是只“讲出来”

这对于工程自动化、编排任务、甚至企业级自动化流程非常关键。([FastAccess AI][3])

---

## 📦 三、如何理解 Codex + MCP 的作用域？

| 场景                  | 有无 MCP | 能否完成 |
| ------------------- | ------ | ---- |
| 生成函数代码              | ❌      | ✔    |
| 访问数据库查询数据           | ❌      | ❌    |
| 调用第三方 API 并处理结果     | ❌      | ❌    |
| 自动提交 PR 到 GitHub    | ❌      | ❌    |
| 自动运行 UI 自动化测试       | ❌      | ❌    |
| 自动执行 PS 操作（浏览器/命令行） | ❌      | ❌    |
| 通过标准化协议调用外部工具       | ✔      | ✔    |

这就是说：**MCP 是为了让 AI “动起来” 而不是只会写代码。**

---

## 🧪 四、对于不同工程怎么写/配置 MCP

> **注意**：具体代码不是每个平台强制要求，但大体思路是一样的：
>
> 1. 添加 MCP 服务器配置
> 2. 在项目里启动 MCP 服务器
> 3. 让 Codex 通过协议去调用（类似 API 调用）

下面按常见工程类型来分步骤说明：

---

### 📱 1) iOS Swift 工程

Swift 工程本身不能直接处理 MCP 协议，但一般做法是：

#### ✅ 方法 1 — 使用 **Codex CLI + MCP Server**

**步骤概览（能让 AI 自动帮你执行任务）**：

1. 安装 Codex CLI：

```bash
npm install -g @openai/codex
```

2. 编辑 `~/.codex/config.toml`
   添加 MCP Server 配置。例如：

```toml
[mcp_servers.my_ios_tools]
command = "swift"
args = ["run", "MySwiftMCPServer"]
startup_timeout_ms = 20000
```

这里 `MySwiftMCPServer` 是一个你自己写的 Swift 服务器，它实现了 MCP 协议，可以处理来自 AI 的请求。([Vibe Sparking AI][2])

3. 启动服务器：

```bash
swift run MySwiftMCPServer
```

4. Codex 运行就能调用你的 Swift 服务了。

---

#### ✅ 方法 2 — 把 MCP Server 做成 REST API

如果你不想写协议服务，你可以：

1. 写一个 **Swift 后端 Web 服务（Vapor 等）**
2. 实现一些 API（比如生成代码、保存文件、运行测试）
3. 使用现成的 MCP 转接器（让 REST API 变成 MCP）

形式上类似：

```
AI(client)  ⇄  MCP Server  ⇄  REST Swift API ⇄  iOS 项目后端逻辑
```

---

### 💻 2) React (JavaScript / TypeScript) 工程

React 本身是前端，因此常见用法是：

#### 🟢 内部自动化（如自动测试、浏览器操作）

你可以写一个 Node.js MCP Server：

```js
const { startMCPServer } = require("@modelcontextprotocol/sdk");

startMCPServer({
  tools: {
    // AI 可以调用的动作
    openBrowser: async ({ url }) => { /* 用 Puppeteer 打开 */ },
    runTest: async () => { /* 用 Jest/Playwright 运行测试 */ },
  }
});
```

然后在 `~/.codex/config.toml`：

```toml
[mcp_servers.react_dev]
command = "node"
args = ["./mcp-server.js"]
```

这样 Codex 能自动：

✔ 运行测试
✔ 操作界面
✔ 修改画面结构
✔ 自动创建 issue/PR

---

### 🐹 3) Go 工程

Go 工程非常适合写自己的 MCP Server，因为 Go 天然有 HTTP 和并发支持。

#### 🟡 典型流程：

1. 写一个 Go MCP Server（使用 JSON-RPC / stdio / socket）

示例（伪代码）：

```go
server := mcp.NewServer()

server.Handle("runGoTest", func(params map[string]interface{}) {
    // 调用 go test 或者 build
})

server.ListenAndServe()
```

2. 配置 Codex：

```toml
[mcp_servers.go_tools]
command = "/path/to/your/gotool"
args = []
```

3. 启动后，Codex 能调用 Go 工具链，例如：

🔹 `runGoTest` → 执行 `go test ./...`
🔹 `grepCode` → 在工程里搜索

---

## 🧩 五、最基础的 MCP 配置模板

这个是你可以复制到 `~/.codex/config.toml` 的例子（适用于各种语言工程）：

```toml
[mcp_servers.serverA]
command = "node"
args = ["./server-a-mcp.js"]
startup_timeout_ms = 15000

[mcp_servers.serverB]
command = "go"
args = ["run", "./mcpserver.go"]
```

当你启动这些 server 并写好逻辑之后，**Codex 在生成代码时就能自动调用这些工具完成一系列操作**。([Vibe Sparking AI][2])

---

## 🧠 六、简要总结（尽量理解）

📍 **MCP 不是代码，而是一种协议**
📍 它让 AI 不只是“会说出一段代码”，而是真正能去调工具、对工程执行真实操作
📍 对于不同项目（Swift / React / Go）来说：
✔ Swift 更多会结合服务器或后端服务接 MCP
✔ React 常用 Node.js MCP Server + 浏览器自动化
✔ Go 最容易写自定义 MCP 服务并且性能高

---

## 🧠 进一步理解的小白比喻

想象：

* **AI Codex = 一个大脑**
* **MCP = 大脑与身体部位之间的标准化神经线**
* **MCP Server = 不同身体器官（浏览器、测试框架、CI/CD、数据库）**

没有 MCP 大脑只能想象、说话
有 MCP 大脑可以自己做事情了 🎯

---

如你要，我也可以给你：

📌 一个 **从零到一的 Swift MCP Server 教程**
📌 一个 **React 具体 MCP Server 代码模板**
📌 一个 **Go MCP Server REST + JSON-RPC 示例**

随时继续问我！👍

[1]: https://fastgptplus.com/zh/posts/codex-cli-mcp-guide?utm_source=chatgpt.com "Codex CLI MCP配置教程 2025完整指南：快速上手与10+服务器示例 | FastGPTPlus"
[2]: https://www.vibesparking.com/zh-cn/blog/ai/openai/codex/mcp/2025-09-10-codex-mcp-setup-10-servers/?utm_source=chatgpt.com "Codex 接入推荐的 10 个 MCP | Vibe Sparking AI"
[3]: https://fastgptplus.com/zh/posts/codex-cli-mcp-integration?utm_source=chatgpt.com "Codex CLI MCP集成完全指南：从配置到实战的深度教程 | FastGPTPlus"

<br/><br/>

Codex CLI 可以通过在 `~/.codex/config.toml` 中定义一个 mcp_servers 部分来配置 MCP，和 Claude 和 Cursor 在各自的 JSON 配置文件中定义 mcpServers 一样，但是 Codex 的格式略有不同，它使用 TOML，而不是 JSON。

比如我添加了以下几个 MCP：

```json
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
env = { "test" = "123456" }

[mcp_servers.puppeteer]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-puppeteer"]
env = { "test" = "123456" }
```

你可以手动的在这个 configure

验证 MCP

<br/><br/><br/>

***
<br/>

> <h1 id= "OpenCode">OpenCode</h1>
[下载和配置：](https://opencode.ai/zh/download)



