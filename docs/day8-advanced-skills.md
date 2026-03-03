# Day 8：深入 Skills 定制 —— 高阶 API 接入 (以 Notion 为例)

> 💡 **学习目标**：掌握 OpenClaw 与第三方重量级 API 的深度集成，构建个人知识库自动化工作流。

## 为什么要接入 Notion？

在日常使用中，AI 助手往往会产生大量有价值的对话、灵感和代码片段。仅仅保留在聊天工具的对话流中非常容易丢失。通过集成 Notion，我们可以实现：
- 📝 **灵感捕获**：随口说出的想法自动分类归档为 Notion 笔记。
- 📊 **数据同步**：将长篇总结、网页抓取内容无缝导入数据库。
- 🔄 **知识沉淀**：让 AI 成为你的“第二大脑”管理员，实现资料从收集到整理的自动化。

相比手动复制粘贴，API 的打通能将保存资料的时间极大缩短，你的效率将得到指数级提升。

## 配置指南

### Step 1: 获取 Notion API Key
1. 访问 [Notion Developers 平台](https://www.notion.so/profile/integrations/internal)
2. 创建一个新的内部集成，获取 `Internal Integration Secret`
3. 在你的 Notion 笔记本中，点击右上角的集成，将刚才创建的机器人的权限挂载到指定 Database。

### Step 2: 创建 Skill 结构与配置文件

要真正发挥 Notion API 的威力，我们需要将它与 OpenClaw 的 **Skill 机制** 结合起来。为了安全和解耦，我们**不推荐**将 Notion 的私有 API Key 写在全局的 `openclaw.json` 里，而是保存在专属 Skill 的配置文件中。

创建一个专属的 Notion Clipper Skill 文件夹，并在里面放入配置。
*(注意：如果你将配置推送到云端，记得将 `config.json` 加入到 `.gitignore` 中以防止敏感信息泄露！)*

```bash
mkdir -p ~/.openclaw/workspace/skills/custom-notion-clipper
cd ~/.openclaw/workspace/skills/custom-notion-clipper
```

创建 `config.json`：
```json
{
  "NOTION_API_KEY": "secret_你的_Notion_Token",
  "DATABASE_ID": "在此处填入你的_Database_ID"
}
```
*提示：`DATABASE_ID` 是指当你网页端打开该 Database 时 URL 里的那串 32 位字符。*

## 实战案例：打造“超级信息捕手” Skill

要真正发挥 Notion API 的威力，我们需要将它与 OpenClaw 的 **Skill 机制** 结合起来。仅仅简单发送文本到 Notion 是不够的，我们可以编写一个高级 Skill，让大模型在保存前为你做大量的预处理工作。

### 痛点场景
你在 Telegram 里看到一篇很长的英文技术文章，或者一个开源项目的 GitHub 链接。你想把它存到 Notion 里，但你不想只存一个干巴巴的链接或者长篇大论的生肉。

### 解决方案：赋予 Agent “阅读图纸”的能力

Agent 默认并不知道 Notion API 的请求格式，也不知道密钥在哪里。我们需要在 `SKILL.md` 里明确告诉它去哪找“钥匙”（读取 `config.json`），并且给它附上一张简单的“说明书”（API 请求示例）。

创建 `~/.openclaw/workspace/skills/custom-notion-clipper/SKILL.md`：

```markdown
---
name: custom-notion-clipper
description: 提取网页或长文本的核心要点，并规范化保存到 Notion 知识库
---

# 任务目标
当用户发来链接或者大段需要收藏的材料时，你必须严格按照以下步骤执行：

1. **信息提炼**：
   - 提取核心论点（不超过 3 句话）
   - 总结 3 个可执行的 Action Items
   - 翻译：如果是外文，提供准确的中文摘要
2. **结构化组装**：将上述信息组装为带有清晰 Markdown 层级的标准文档块。
3. **获取凭证**：请使用文件读取工具，读取当前目录下的 `config.json` 文件，获取 `NOTION_API_KEY` 和 `DATABASE_ID`。
4. **调用 API 归档**：使用 `web_fetch` 或终端的 `curl` 工具，参考文末底部的 API 说明，发起真实的 HTTP 请求将组装好的内容写入 Notion。

# 执行范例
用户输入：保存这篇文章 https://example.com/long-article
你的回复：
1. 正在阅读并提炼本文核心观点...
2. 提取要点完成，正在读取 config.json 准备调用 API 归档...
3. ✅ 归档成功！文章关键内容已提炼，Notion 链接：[点击查看](...)

---

# 附录：Notion API 请求参考

你可以根据不同的需要选择合适的 API 进行交互，请始终记得在 Headers 中带上你的 \`NOTION_API_KEY\`。

## 场景 1：在 Database 中创建新 Page（常用）
当需要将收集的资料作为新笔记存入数据库时，使用此接口：

\`\`\`bash
curl -X POST 'https://api.notion.com/v1/pages' \
  -H 'Authorization: Bearer <NOTION_API_KEY>' \
  -H 'Content-Type: application/json' \
  -H 'Notion-Version: 2022-06-28' \
  -d '{
  "parent": { "type": "database_id", "database_id": "<DATABASE_ID>" },
  "properties": {
    "Name": { "title": [ { "text": { "content": "<提取的标题>" } } ] },
    "Tags": { "multi_select": [ { "name": "自动收集" }, { "name": "<你的分类提取>" } ] }
  },
  "children": [
    {
      "object": "block",
      "type": "paragraph",
      "paragraph": { "rich_text": [ { "type": "text", "text": { "content": "<填入总结的正文内容>" } } ] }
    }
  ]
}'
\`\`\`

## 场景 2：向已有 Page 追加内容块（追加模式）
如果笔记页面已经存在（例如你想把今天的多个闪念追加到同一个“今日收件箱” Page 里），你需要知道目标 \`PAGE_ID\` 或 \`BLOCK_ID\`（这里互通），然后使用 Append Block Children 接口：

\`\`\`bash
curl -X PATCH 'https://api.notion.com/v1/blocks/<PAGE_ID_OR_BLOCK_ID>/children' \
  -H 'Authorization: Bearer <NOTION_API_KEY>' \
  -H 'Content-Type: application/json' \
  -H 'Notion-Version: 2022-06-28' \
  -d '{
  "children": [
    {
      "object": "block",
      "type": "heading_2",
      "heading_2": { "rich_text": [ { "text": { "content": "<追加的段落标题>" } } ] }
    },
    {
      "object": "block",
      "type": "paragraph",
      "paragraph": { "rich_text": [ { "text": { "content": "<追加的正文内容>" } } ] }
    }
  ]
}'
\`\`\`
```

### 效果展示
当你在这个 Skill 的加持下向 Agent 发送那篇两万字的英文长文时，Agent 会自动阅读全文内容，提取出几百字的中文精要总结，然后直接将排版精美的结构化卡片推送到你的 Notion Database 里，并打上对应的分类标签。

通过将 **API 的原子能力**（这里指往 Notion 写入数据）和 **Skill 的业务编排能力**（告诉大模型在写入前必须经过翻译、提炼、贴标签等复杂工序）结合，你才真正创造了一个拥有高级自动化工作流的专属 AI 助理。

## 延展与作业
1. **探索更多 API 组合**：Notion API 不仅能存数据，还能**读**数据。试着编写一个 `todo-reminder` Skill，让 Agent 每天早上根据 Notion 里的任务库给你发早报。
2. **今日作业**：完成你的第一个包含自动化提炼逻辑的 Notion Clipper，并在测试群组里提交效果截图交付！
