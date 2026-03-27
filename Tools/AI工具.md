
- [AI集合模型ollama](#AI集合模型ollama)
- [OpenAI的CodeX](#OpenAI的CodeX)
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

> <h1 id= "OpenCode">OpenCode</h1>
[下载和配置：](https://opencode.ai/zh/download)



