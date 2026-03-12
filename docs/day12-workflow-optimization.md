# Day 12：工作流优化 —— 打造自主上线的全栈开发 Agent

> 🎯 **学习目标**：理解 OpenClaw 对日常工作的增益，掌握从 Awesome Usecases 寻找灵感的方法，并实战演示"需求文档驱动"的自动化代码开发与上线流程。

## 为什么需要工作流优化？

在之前几天的学习中，我们分别掌握了 OpenClaw 的单项技能（渠道接入、Skills、Cron、Webhooks）。但真正的效率飞跃，来自于将这些零散的**工具**组合成有价值的**协同工作流**。

你可以把 OpenClaw 想象成你的**初级且不知疲倦的数字打工人**：
- **只会聊天的时候**：它是个词典和草稿本。
- **加上了 Skills (`list_dir`, `write_to_file` 等)**：它成了会敲键盘的程序员。
- **加上了 Cron / Webhooks**：它成了主动跟进进度的项目经理。

**OpenClaw 对日常工作的增益包括：**
1. **打破"空白文档综合征"**：把写代码的第一行、写文章的草稿交给 Agent，你只负责 review 和微调。
2. **异步工作**：你在睡觉时，Cron 唤醒 Agent 抓取昨夜新闻、跑测试、写周报。
3. **隔离心智负担**：把"查阅大量 API 文档"、"在日志里找 bug" 等高内耗任务外包给专门的 Agent。

---

## 寻找灵感：Awesome OpenClaw Usecases

不知道该让 Agent 干什么？社区已经为你准备好了答案库。

👉 **[Awesome OpenClaw Usecases](https://github.com/hesamsheikh/awesome-openclaw-usecases)**

这是一个由社区维护的真实应用场景合集，只收录真实能用的案例。分类包括：

- **Social Media**：比如 [Tech News Digest]，结合 RSS 和抓取，自动生成有质量过滤的每日科技新闻简报。
- **Creative & Building**：比如 [Goal-Driven Autonomous Tasks]，你只需扔给它你的大致目标，它会在半夜自主切分任务甚至写好轻应用的雏形代码。
- **Infrastructure & DevOps**：比如 [Self-Healing Home Server]，一个带有 SSH 权限且一直运行的家域网管理员，遇到服务宕机会尝试自动重启。
- **Productivity & Finance 等**：比如接管所有邮件与日历作为个人 CRM。

---

## 实战案例：从 0 到上线开发"贪吃蛇"游戏

本节实战借鉴了社区经典案例：[Autonomous Educational Game Development Pipeline](https://github.com/hesamsheikh/awesome-openclaw-usecases/blob/main/usecases/autonomous-game-dev-pipeline.md)（一位奶爸用 Agent 每天流水线式开发和修 bug，最终为女儿搭建了无广告的 40+ 款页游门户）。

我们将用相似的思路，配置一个专属的**Web开发 Agent**，仅仅通过提供一份**需求规格说明书 (Spec)**，让它自主完成一个贪吃蛇游戏的开发，甚至借助 `git` 命令推送到 GitHub Pages 上线！

### 核心思路：约束即自由

要想让 LLM 写出可用的项目，**绝不是直接对它喊一句"给我写个游戏"**，而是用 `SOUL.md` 和 `game-spec.md` 给出极度具体的规则约束。

### Step 1: 创建专属的 Web Dev Agent

日常对话的 `main` Agent 不适合做繁重的开发任务。我们需要一个专属的"游戏开发工匠"。

```bash
openclaw agents add game-dev
```

进入它的工作区，覆盖 `~/.openclaw/workspace-game-dev/SOUL.md`：

```markdown
你是一个顶尖的前端游戏开发工程师。
你的任务是根据用户的需求文档，严格用纯 HTML/CSS/JS (无框架) 开发可玩的 Web 游戏。

**工作流准则（CRITICAL）：**
1. 收到需求后，先使用 `list_dir` 和 `view_file` 检查当前目录及需求文档内容。
2. 使用 `write_to_file` 一次性将完整的、无缺漏的代码写入对应的文件（比如 index.html, style.css, script.js）。不要省略任何逻辑。
3. 严格遵循代码规范：自适应移动端，CSS 美观现代，JS 逻辑必须包含 Game Over 和重置机制。
4. 如果用户报告 Bug，优先处理 Bug，不要新开需求。
```

> 此处的灵魂在于，你赋予了 Agent 使用本地文件的权限（内置工具）。

### Step 2: 准备需求规格书（Spec）

新建一个工作目录并准备需求文档（不要让 Agent 在默认目录乱写）：

```bash
mkdir -p ~/.openclaw/workspace-game-dev/my-snake-game
cd ~/.openclaw/workspace-game-dev/my-snake-game
```

在目录下创建 `game-spec.md`：

```markdown
# 贪吃蛇游戏需求规格书 (Snake Game Spec)

## 技术栈
- 纯单个文件 `index.html`，内联 CSS 和 JS，方便部署。

## 视觉与交互
- 经典像素风或现代极简风（请发挥你的设计水平设定 CSS）。
- 居中的 400x400 游戏画布 (Canvas)。
- 页面必须有：标题 "Snake Claw"、当前分数展示、最高分展示（localStorage 存储）。

## 游戏机制
- 初始长度为 3，速度适中（例如 100ms 刷新率）。
- 蛇撞到墙壁或撞到自身即触发 Game Over。
- 吃到苹果（用红色方块表示）长度+1，分数+10。
- 按键盘上下左右控制方向，防止蛇直接 180 度掉头（如正在向右，不能直接向左）。

## 交付
阅读完毕后，请直接在当前目录生成 `index.html`。
```

### Step 3: 开始召唤 Agent 写代码

我们要把这个完整的项目通过一次对话交给 Agent 开发。

1. 打开 OpenClaw Control UI 控制台：
   ```bash
   openclaw dashboard
   ```
2. 在左侧选择刚刚创建的 `game-dev` Agent。
3. 输入以下完整指令（注意替换为你自己的完整路径）：

```text
请查看我电脑上的这个目录： /Users/你的用户名/.openclaw/workspace-game-dev/my-snake-game
请严格遵照该目录下的 game-spec.md 需求规范，在这个目录下为我一步到位开发出这个游戏代码文件 index.html
```

> **预期表现**：你会看到 Agent 的思考过程，它调用了 `list_dir` 和 `view_file` 看了 spec，接着调用 `write_to_file` 在你的指定目录下创建了 `index.html`。打开浏览器访问 `file:///Users/你的用户名/.openclaw/workspace-game-dev/my-snake-game/index.html` 试玩一下。

### Step 4:"Bugs First" —— 测试与修 Bug

自动生成的代码极少 100% 完美。实战中的智慧是**让 Agent 自己修自己。**

如果没遇到 Bug，我们可以提点优化意见：

```text
现在的游戏速度有点慢，并且我希望蛇头有特别的颜色区分，游戏结束时屏幕变暗并显示一个居中的重新开始按钮。请更新 index.html
```

Agent 会用 `replace_file_content` 或直接重写来修复它。这个循环（Review -> 提 Issue -> Agent Fix）就是未来程序员最核心的日常。

### Step 5: 初始化 Git 并部署到 GitHub Pages

游戏测试没问题后，我们要把它挂载到公网。既然我们的 Agent 可以帮你写代码，**我们完全可以把整个 `git` 工作流也交给它！**只要你赋予它调用终端命令的权利就行。

在目前的对话流下，直接跟 Agent 说：

```text
这套代码我觉得非常完美了。
1. 请先在 GitHub 上帮我创建一个 public 仓库，命名为 snake-claw。
2. 然后帮我在当前这个项目目录下面初始化 git，把所有文件加进去，完成第一次 commit，并 push 到这个新建的远程仓库主分支。
```

> **注意：** 使用这个进阶流派，你的 Agent 环境需要有一些前置权限。如果是本地运行的 OpenClaw，Agent 通常有权限替你执行终端命令建立仓库；如果受到环境限制，你需要自己先去 GitHub 网页端建立一个空仓库，然后把仓库地址发给 Agent 告诉它 "这是远程地址，请帮我 push"。 

**部署 GitHub Pages**（这步网页依然是最简单的）：
1. 打开 GitHub 刚刚建好的仓库页面 -> Settings -> Pages。
2. Source 选择 `Deploy from a branch`。
3. Branch 选择 `main`，文件夹默认 `/root`，点击 Save。
4. 等待 1-2 分钟，你就可以在 `https://你的用户名.github.io/snake-claw/` 访问你的贪吃蛇游戏了！

---

## 更多

欢迎大家在这里补充更多工作流优化案例。

---

## ✅ 今日练习

- [ ] 浏览 Awesome Usecases，找到一个最能解决你当前痛点的案例记录研读。
- [ ] 体会"约束 LLM（写好 Prompt 和限制环境）"比"放任 LLM 发散"更容易得到稳定结果。

---
[下一天：最终项目实战探讨 →](day13-final-project.md) (待更新)
