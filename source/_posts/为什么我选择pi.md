---
title: 为什么我选择pi
tags:
  - pi
  - agent
  - AI辅助写作
categories:
  - tools
date: 2026-09-28 12:04:56
---

关于我在众多`agent`工具中，为什么选择了[`pi`](https://pi.dev/)

试过的工具不算少，最后让我固定下来的原因其实很朴素：**它足够小，改动它的成本足够低，而且不用我改变已有的习惯**。下面按当初做决定时的三条标准来说。

## 极简

没有复杂的功能，一切由自己定义。可以配置几个简单的`skills`即可，不占用过多的上下文

![image-20260928120905681](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/image-20260928120905681.png)

“极简”不是功能少，而是**默认不加载**。`pi` 的资源一共就四类，从轻到重排是：

- `skills`：一个目录加 `SKILL.md`，可以顺带带脚本和参考资料
- `prompt templates`：纯 `Markdown`，文件名就是命令名
- `extensions`：`TypeScript` 模块，能注册工具、命令、快捷键和事件钩子
- `themes`：终端配色

关键在于 `skills` 的加载方式：启动时进系统提示词的只有每个 `skills` 的**名字和描述**，完整说明要等模型判断这个任务需要它时才去读 `SKILL.md`。也就是说，装了十个 `skills` 不等于每次对话都背着十份说明书——这一点直接决定了我能不能长期往里堆东西而不把上下文撑爆。

`prompt templates` 更轻，本质就是给一段常用提示词起个名字：

```markdown
---
description: Review staged git changes
argument-hint: "[focus]"
---
Review the staged changes. Focus on ${1:-correctness, security, and error handling}.
```

放在 `~/.pi/agent/prompts/review.md`，它就变成 `/review`；`${1:-默认值}` 接参数，`/review concurrency` 这样传。不用写一行代码。

只有确实需要**可执行**的东西时才轮到 `extensions`，那时才需要写 `TypeScript`。这个梯度很重要：大多数需求我用 `Markdown` 就能解决，不必为了加个小功能去维护一个插件工程。

## 响应

启动速度很快，可以适配各种自己的模型，用`cc-switch`接入就好

![image-20260928121041483](https://euclid-picgo.oss-cn-shenzhen.aliyuncs.com/image/image-20260928121041483.png)

模型这块是我最看重的，因为它决定了我能不能用上自己手头的额度。`pi` 的做法是：

- 内置目录里的服务商直接 `/login`，走订阅或 `API key`，凭证存在 `auth.json`
- 也支持环境变量，例如 `OPENAI_API_KEY`、`DEEPSEEK_API_KEY`、`MISTRAL_API_KEY` 这类，适合不想落盘的场景
- 切换模型用 `/model`，`Ctrl+P` 在候选之间循环；在选择器里按 `Ctrl+S` 可以把当前模型存成新会话的默认
- `/thinking` 调思考等级，`/scoped-models` 控制 `Ctrl+P` 循环的范围

自己接的服务也留了口子：只要对方讲的是 `OpenAI` / `Anthropic` / `Google` 兼容的协议（本地的 `Ollama`、`LM Studio`、`vLLM`、`SGLang`，或者各种中转），写进 `models.json` 就是一个可选项：

```json
{
  "providers": {
    "ollama": {
      "baseUrl": "http://localhost:11434/v1",
      "api": "openai-completions",
      "apiKey": "ollama",
      "models": [{ "id": "qwen2.5-coder:7b" }]
    }
  }
}
```

本地 `GGUF` 模型有专门的 `llama.cpp` 路由，用 `/llama` 管理、`/model` 选择。凭证的优先级是 `--api-key` > `auth.json` > `models.json` 里的 `apiKey` > 环境变量，冲突时按这个顺序生效。

顺带记一下版本：我是从 `0.87.1` 开始用的，`Node.js` 需要 `22.19` 以上。命令类的东西版本一变就可能对不上，写下来免得以后自己回头看不懂。

## 类claude code

命令模式基本和 `claude code`一致，迁移无感

这不是“抄得像”，而是**肌肉记忆不用重建**：`/` 打开命令菜单，`@` 搜文件加进上下文，`Tab` 补全路径，`Shift+Enter` 换行，`Ctrl+O` 折叠或展开工具输出，`Ctrl+T` 显示思考块。

边跑边插话的交互也一致，这点在实际干活时比快捷键更重要：

| 想做的事 | 操作 |
| --- | --- |
| 修正当前任务方向 | 输入后 `Enter` |
| 在当前任务之后追加工作 | 输入后 `Alt+Enter` |
| 把排队中的消息收回编辑器 | `Alt+Up` |
| 中断当前任务 | `Escape` |

**`Windows` 用户注意**：`Windows Terminal` 占用了部分 `Alt` 快捷键，上面几个组合键未必都能直接用，需要按官方文档里给的替代键位配一下。我在 `Windows` 上就先踩过这个。

## 上下文和会话

用久了的工具，最后拼的都是“上下文怎么管”。`pi` 把会话存成 `JSONL`，消息、工具调用、模型切换、压缩都作为条目记录，而且**按树存**：

- `/tree` 在当前会话里换分支，回到早先的提问改一版，原来的分支不会被抹掉
- `/fork` 从某条历史用户消息开一个新会话，把岔路做成独立的工作
- `/clone` 把当前状态复制成一个新会话

上下文逼近上限时会自动压缩：插入一条摘要条目，后续请求用摘要替代更早的消息，**原始条目仍然留在会话文件里**，不是删掉。想控制压缩后保留什么，可以手动 `/compact 保留关于 xxx 的结论`。页脚实时显示上下文占用，`/session` 能看到会话文件、消息数、token 和花费。

对我这种经常“干到一半想回去换个方向”的人，这套东西比单线的聊天记录实用得多。

## 定制和分发

四类资源都能单独用，也能打包成一个 `Pi package` 通过 `npm` 或 `git` 分发：

```bash
pi install npm:@example/pi-tools@1.0.0
pi install git:github.com/example/pi-tools@v1
pi install ./local-package
pi list
```

`pi update --extensions` 负责把包里的资源刷新到最新，`pi -e npm:@example/pi-tools` 可以在不写进配置的情况下先试一次。对我这种喜欢自己改工具的人，等于“**个人配置和项目配置是分开的**”：项目里的 `.pi/settings.json` 只在信任该项目之后才加载。

## 能被脚本调用

这一点容易被忽略，但决定了它能不能长在别的工作流里。同一套 agent 和会话机制，有几种外壳：

| 模式 | 用途 |
| --- | --- |
| 交互模式 | 人在终端里干活 |
| `print` 模式 | 跑一个提示词，只要最终结果 |
| `JSON` 模式 | 把事件按 `JSONL` 输出 |
| `RPC` 模式 | 作为子进程，`stdin` / `stdout` 收发 `JSONL`，与语言无关 |
| `TypeScript SDK` | 在 `Node.js` 进程内直接创建和控制会话 |

所以同一个工具，既能在终端里手敲，也能被脚本或别的程序驱动，不必再为“自动化场景”换一个产品。

## 代价

说好处也得说代价，免得看着像无脑推荐：

- **它不逐条确认工具调用**。方便，但如果你习惯“每一步都点同意”，会不适应；不确定的目录该用沙箱或自己控制权限。
- **`extensions` 在 `pi` 进程内执行**，拿到的是当前用户的系统权限。第三方包安装前值得读一眼源码，项目级配置更要先信任项目再加载。
- **极简的另一面是你得自己补**。缺少的功能要么用 `skills`、`prompt templates` 补，要么自己写 `extensions`——这对愿意折腾的人是优点，对只想开箱即用的人是成本。

## 小结

按我的三条标准对照下来：

- **极简**：核心很小，资源默认不加载，`skills` 只在需要时读正文
- **响应**：启动快，模型能换成自己的，兼容端点写进 `models.json` 就完事
- **迁移**：命令和交互跟 `claude code` 基本一致，`Windows` 上注意 `Alt` 键被占用的问题

它的取舍很明确：**把“定义权”交给你，代价是你要自己定义**。对我来说这笔交易是划算的。
