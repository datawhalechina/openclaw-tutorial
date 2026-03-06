# Day 10：Webhook —— 让外部事件驱动你的 AI Agent

> 🎯 **学习目标**：理解 Webhook 的工作原理，并搭建一个 GitHub PR/Issue 分诊机器人，自动将代码评审建议推送到 Telegram 或飞书。

## 概念：门铃 vs 闹钟

在前面的 Day 5 和 Day 6，我们学过两种"主动出击"的方式：

- **Cron** 像手机闹钟，每天早 8 点响，不管有没有新事情发生。
- **Heartbeat** 像你自己定时去查邮件，每隔 N 分钟主动问一句"有什么新消息吗"。

这两种方式有一个共同缺点：**它们不知道事件什么时候发生**。假设你的 GitHub 仓库凌晨 3 点来了一个紧急 PR，Cron 要等到早上才会通知你，中间可能错过了 8 小时。

**Webhook 是门铃**。GitHub 一发生事件（PR 新开、Issue 被评论），就立刻按响你家的门铃，你的 Agent 即时醒来处理。主动权从你这边转移到了外部系统。

```
Cron/Heartbeat:  你 → 每隔一段时间 → 问有什么新消息？
Webhook:         外部系统 → 事件发生时 → 主动推送给你
```

OpenClaw 内置了一个轻量 HTTP 服务端，只需在配置里开启，就能接收来自任何第三方系统的 Webhook 请求。

---

## 实战案例：GitHub PR 分诊机器人

**场景**：你维护着一个开源项目，每天都可能有新的 PR 和 Issue。你不可能每次都亲自去 GitHub 看一眼再决定要不要处理。我们要搭建的是：

1. GitHub 有新 PR 或 Issue → 自动触发 Webhook
2. OpenClaw 收到后，调用一个专门的"代码评审 Agent"
3. Agent 分析：**要不要看？有什么风险点？建议怎么回复？**
4. 把分析报告**直接推送**到你的 Telegram 或飞书

整体数据流如下：

```mermaid
flowchart LR
    GitHub["GitHub<br> PR/Issue 事件"] -- POST /hooks/github --> OC["OpenClaw <br> Webhook 服务"]
    OC -- Mapping 解析<br>组装 message --> Agent["hooks Agent<br>代码评审专家"]
    Agent -- 分析报告 --> Deliver["deliver: true"]
    Deliver -- channel: telegram --> TG["Telegram / 飞书<br>收到分诊报告"]
```

---

## Step 1：启用 Webhook 服务

打开 `~/.openclaw/openclaw.json`，添加 `hooks` 配置块：

```json5
{
  "hooks": {
    "enabled": true,
    // 用于验证请求合法性的共享密钥，不能为空
    "token": "your-secret-token-here",
    "path": "/hooks",
    // 默认会话 key，所有 hook 触发的 Agent 共用这个会话隔离空间
    "defaultSessionKey": "hook:github",
    "allowRequestSessionKey": false,
    "allowedSessionKeyPrefixes": ["hook:"],
    // 只允许路由到 main 和 hooks 这两个 Agent
    "allowedAgentIds": ["main", "hooks"]
  }
}
```

然后在你的 Shell 环境中设置密钥（建议加入 `~/.zshrc` 或 `~/.bashrc`）：

```bash
export OPENCLAW_HOOKS_TOKEN="your-secret-token-here"
```

**密钥选取原则**：随机生成一个至少 32 位的字符串，不要用简单的单词。可以用以下命令生成：

```bash
openssl rand -hex 24
# 输出示例：a7f3c2d8e1b4096f5a23c7d9e0b1f4a2e3d6c8b5
```

重启 OpenClaw 使配置生效：

```bash
openclaw gateway restart
```

此时，OpenClaw 已经在监听 `POST http://127.0.0.1:18789/hooks/{mapping名称}` 端点了。例如，当你在后面的 Step 3 中添加名为 `github` 的 mapping 后，对应端点就是 `/hooks/github`。

---

## Step 2：创建代码评审专属 Agent

Webhook 请求需要一个专门的 Agent 来处理，而不是直接污染你日常对话的主 Agent。

```bash
# 创建名为 "hooks" 的专属 Agent
openclaw agents add hooks
```

这条命令会在 `~/.openclaw/workspace-hooks/` 下初始化一套独立的工作区。接下来为它配置专业人格，编辑 `~/.openclaw/workspace-hooks/SOUL.md`：

```markdown
你是一位资深代码评审助手，专门处理 GitHub PR 和 Issue 的快速分诊。

每当收到一条 PR 或 Issue 信息，你需要按照以下格式给出分诊报告：

## 📋 [PR/Issue 标题]

**🔴 优先级**：[紧急 / 普通 / 可忽略]
- 判断依据：（说明为什么这个优先级）

**⚠️ 风险点**：
- （列出 1-3 个潜在的技术风险或需要关注的点；如果是新功能 PR 则关注安全性/破坏性变更；如果是 Bug Issue 则关注影响范围）

**💬 建议回复**：
（给出一段可以直接复制粘贴的回复模板，语气友好专业）

---
回复要简洁，控制在 200 字以内。如果信息不足以判断风险，直接说明需要查看具体代码。
```

---

## Step 3：配置 GitHub Webhook Mapping

OpenClaw 的 `hooks.mappings` 功能允许你把任意 HTTP payload 转换成 Agent 可以理解的指令。这是整个流程的核心配置。

在 `openclaw.json` 中添加 `mappings` 字段：

```json5
{
  "hooks": {
    "enabled": true,
    "token": "your-secret-token-here",
    "path": "/hooks",
    "defaultSessionKey": "hook:github",
    "allowRequestSessionKey": false,
    "allowedSessionKeyPrefixes": ["hook:"],
    "allowedAgentIds": ["main", "hooks"],
    "mappings": [
      {
        "name": "github",
        "action": "agent",
        // 路由到专门的 hooks Agent
        "agentId": "hooks",
        // 分析完成后推送到 Telegram（改成 "feishu" 即可切换到飞书）
        "deliver": true,
        "channel": "telegram",
        // 用 {{}} 模板语法从 GitHub payload 中提取字段，组装成 messageTemplate
        "messageTemplate": "请对以下 GitHub 事件进行分诊：\n\n类型：{{action}} {{pull_request.title}}{{issue.title}}\n作者：{{pull_request.user.login}}{{issue.user.login}}\n仓库：{{repository.full_name}}\n链接：{{pull_request.html_url}}{{issue.html_url}}\n描述：{{pull_request.body}}{{issue.body}}"
      }
    ]
  }
}
```

> **模板语法说明**：字段名是 `messageTemplate`（不是 `message`）。`{{pull_request.title}}` 会从 GitHub 发来的 JSON payload 中提取对应字段。PR 事件和 Issue 事件的 payload 结构不同，`{{pull_request.title}}{{issue.title}}` 这种写法是把两个字段拼在一起——实际只有一个会有值，另一个为空，所以结果是对的。

重启 OpenClaw：

```bash
openclaw gateway restart
```

---

## Step 4：注册 GitHub Webhook

出于安全考虑，不建议使用本机来测试，请在服务器上测试。

OpenClaw 默认只监听 `127.0.0.1:18789`（本机回环），无法被外网直接访问。建议用 nginx 在 80 端口接收外部请求，再转发给本机的 18789。

```bash
# 安装 nginx（CentOS/OpenCloudOS）
yum install -y nginx

# 写 nginx 配置，只暴露 /hooks/ 路径
# 注意：需要在转发时注入 Authorization 头，因为 GitHub 原生 Webhook 不会自动携带它
cat > /etc/nginx/conf.d/openclaw-hooks.conf << 'EOF'
server {
    listen 80;
    server_name _;

    location /hooks/ {
        proxy_pass http://127.0.0.1:18789;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        # 把 OpenClaw token 在转发时注入（替换成你的实际 token）
        # 注意：这里的 token 是 OpenClaw 用于身份验证的令牌，和下面 GitHub 填写的 Secret 是两回事
        proxy_set_header Authorization "Bearer your-token-here";
    }
}
EOF

systemctl enable --now nginx
```

然后在云服务器控制台安全组中放行 **TCP 80 端口**的入站流量（18789 无需对外暴露）。

> **安全性说明**：这个方案的安全边界依赖 `hooks.token`。每个请求必须携带正确的 `Authorization: Bearer <token>` 才会被处理，错误 token 会触发 429 限速。只要 token 足够随机（`openssl rand -hex 24` 生成），暴力破解的成本极高。`/hooks/` 以外的路径不会被 nginx 转发，其余端口也保持关闭。

得到公网地址后，前往 GitHub：

1. 打开你的仓库 → **Settings** → **Webhooks** → **Add webhook**
2. **Payload URL**：`http://你的服务器IP/hooks/github`
3. **Content type**：选 `application/json`
4. **Secret**：可以留空，或填写任意字符串（此处 GitHub 的 Secret 是用于计算 HMAC 签名的，OpenClaw 当前版本不做签名校验，访问控制由 nginx 注入的 `Authorization` 头承担）
5. **Which events**：选 `Let me select individual events`，勾选：
  - ✅ Pull requests
  - ✅ Issues
6. 确保 **Active** 已勾选，点击 **Add webhook**

GitHub 会立即发一个 `ping` 事件测试连通性，你可以在 Webhooks 页面看到绿色勾表示成功。

---

## Step 5：本地测试验证

不需要真的去 GitHub 提一个 PR，可以用 `curl` 模拟 GitHub 发来的 payload：

```bash
# 模拟一个 PR 事件
curl -X POST http://127.0.0.1:18789/hooks/github \
  -H 'Authorization: Bearer your-secret-token-here' \
  -H 'Content-Type: application/json' \
  -d '{
    "action": "opened",
    "pull_request": {
      "title": "feat: 新增用户认证模块，使用 JWT",
      "user": { "login": "contributor-alice" },
      "html_url": "https://github.com/your-org/your-repo/pull/42",
      "body": "本 PR 引入了 JWT 认证。修改了 auth/middleware.ts，移除了旧的 session 机制，并新增了 /api/token 端点。影响所有需要登录的接口。"
    },
    "repository": { "full_name": "your-org/your-repo" }
  }'
```

预期：

- 命令返回 `200 OK`
- 几秒后，你的 Telegram 或飞书收到一条类似这样的消息：

```
📋 feat: 新增用户认证模块，使用 JWT

🔴 优先级：紧急
- 判断依据：涉及认证机制的破坏性变更，移除了旧 session 系统

⚠️ 风险点：
- JWT 密钥管理：需确认 secret 是否通过环境变量注入，不能硬编码
- 向后兼容性：移除 session 机制会导致已登录用户被强制登出
- /api/token 端点安全：需检查是否有速率限制防止暴力破解

💬 建议回复：
感谢贡献！这个 PR 引入了认证机制的重要变更。在合并前，能否确认：1）JWT secret 的存储方式；2）是否有迁移方案避免现有用户被登出；3）新端点是否已有限流保护？
```

下面是实际测试结果截图：

![webhook_success](../docs/imgs/webhook_success.png)


---

## 常见问题排查

**收到 `401 Unauthorized`**

- 检查 `Authorization: Bearer <token>` 的 token 值是否与配置一致

**收到 `400 Bad Request`**

- 检查 `Content-Type: application/json` 是否已设置
- 检查 JSON 格式是否合法（可用 `jq` 校验）

**消息没有推送到 Telegram/飞书**

- 确认 `openclaw.json` 中 `deliver: true` 和 `channel` 已正确设置
- 确认 Telegram Bot 或飞书渠道已在 `channels` 中配置并正常工作
- 用 `openclaw gateway logs` 查看实时日志

---

## ✅ 今日练习

- 在 `openclaw.json` 中启用 hooks，重启后用 Step 5 的 curl 命令测试 `/hooks/github` 端点能正常返回 `200`
- 配置 GitHub Mapping，用模拟 payload 验证分诊报告能推送到 Telegram 或飞书
- （进阶）修改 Mapping 的 `channel` 字段，尝试同时推送到两个渠道

> 完整配置参考：[`examples/webhooks/github-pr-triage.json`](../examples/webhooks/github-pr-triage.json)

---

[上一天：多 Agent 团队配置 ←](day9-multi-agent.md)