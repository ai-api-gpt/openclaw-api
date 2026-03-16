# 【2026最新】OpenClaw配置教程：一键接入AI大模型API中转站 | 简易API

👋 大家好呀！随着 2026 年各类 AI 模型的爆发，OpenClaw 已经成为了许多开发者和 AI 爱好者的得力助手。但是，直接使用官方的 API 往往面临着**价格昂贵**、**网络连通性差**甚至**封号风险**等痛点。

今天给大家带来一篇小白保姆级的 **OpenClaw配置教程**！教你如何通过接入 **API中转站**，优雅、低成本地调用诸如 Claude 4.5、GPT-4o 等顶级的 **AI大模型API**。

> 📌 **适用版本**：2026.2.6 及之后的 OpenClaw（参考仓库 `openclaw/openclaw`）
> ⏱️ **难度**：⭐（只需修改一个配置文件，5分钟搞定！）

---

## 📚 一、这篇教程会帮你做到什么？

简单来说，我们要完成一次“偷天换日”。让 OpenClaw 默认使用你配置的第三方中转服务，而不是官方昂贵的 API。

**🧐 原理图解：**
从“直连官方网络受限”，变成“走极速中转平台”：

```text
你（用户） ──> OpenClaw 主程序
     ├─（❌ 原路：官方 OpenAI/Anthropic，太贵/连不上）
     └─（✅ 新路：稳定高性价比的 API中转站）
                    ↓
          Claude 4.5 / GPT-4o / 其他顶级模型
```

全流程我们只需要修改一个文件：`~/.openclaw/openclaw.json`。

---

## 🛠️ 二、你要提前准备的三样东西

在开始动手之前，你需要一个靠谱的 **API中转站** 提供商。

💡 **私人推荐**：如果你还没有稳定好用的接口，我目前自用且强烈推荐的是 [**简易API (Jeniya Chat)**](https://jeniya.chat/)。它全面兼容 OpenAI 协议，支持全网最新模型，不仅速度极快，而且性价比极高，非常适合搭配 OpenClaw 使用！

请在你的中转平台（如 [简易API](https://jeniya.chat/)）后台，获取并准备好以下 3 个信息：

1. **API Base URL（接口地址）**
   * 示例：`https://jeniya.chat/v1` （如果是其他平台请替换）
   * ⚠️ **重要提示**：链接最后必须带 `/v1`！
2. **API Key（密钥）**
   * 示例：`sk-abc123456...`
   * 这是你的余额凭证，**千万不要泄露给别人**。
3. **Model ID（模型 ID）**
   * 示例：`claude-opus-4-5-20251101-thinking`
   * 必须和平台提供的模型列表里一字不差。

---

## 💾 三、先备份配置（出错能秒回滚！）

**这一步绝对不能跳过！** 修改 JSON 文件最怕少个逗号，备份好后，一行命令就能恢复如初。

根据你的操作系统，复制对应命令执行：

**🍎 macOS / 🐧 Linux (bash / zsh)：**
```bash
cd ~/.openclaw
# 备份当前配置
cp openclaw.json openclaw.json.bak
# 如果改坏了，用这行恢复：cp openclaw.json.bak openclaw.json
```

**🪟 Windows PowerShell：**
```powershell
cd "$env:USERPROFILE\.openclaw"
# 备份当前配置
Copy-Item openclaw.json openclaw.json.bak -Force
# 如果改坏了，用这行恢复：Copy-Item openclaw.json.bak openclaw.json -Force
```

只要 `.bak` 备份在，心里就不慌！😎

---

## ✍️ 四、修改核心配置（只需改 3 处）

请用 VS Code、Sublime Text 或 Cursor 等代码编辑器打开 `openclaw.json`。文件路径通常是：
* **macOS / Linux**：`~/.openclaw/openclaw.json`
* **Windows**：`C:\Users\你的用户名\.openclaw\openclaw.json`

### 📍 第一处：配置 API Key（安全第一）
在文件最上方，找到 `"env": { ... }` 这一块，如果没有就自己加上。我们将把简易API的密钥存在这里：

```json
  // ... 前面是 meta, wizard 等 ...

  "env": {
    // 👇【新增这一行】名字自己起，这里我用 JENIYA_API_KEY，后面填你的 sk-密钥
    "JENIYA_API_KEY": "sk-xxxxxxxxxxxxxxxxxxxxxxxx"
  },

  // ...
```
*(注意：下面第二处我们会通过 `${JENIYA_API_KEY}` 来引用它，这样更安全！)*

### 📍 第二处：注册第三方服务商（models.providers）
向下滑动，找到 `"models": { "providers": { ... } }`，在里面新增我们中转站的配置：

```json
  "models": {
    "mode": "merge",
    "providers": {

      // 👇【从这里开始新增】===
      "jeniya_api": {
        // 1. 【必须修改】中转地址，记得带 /v1
        "baseUrl": "https://jeniya.chat/v1",

        // 2. 引用上面 env 里设置的 Key！⚠️注意是 $ + {变量名}
        "apiKey": "${JENIYA_API_KEY}",

        // 3. 固定写法，表示兼容 OpenAI 协议
        "api": "openai-completions",

        // 4. 这里列出你想用的 AI大模型API
        "models": [
          {
            // 【必须修改】模型 ID：和中转平台显示的一模一样
            "id": "claude-opus-4-5-20251101-thinking",
            "name": "Claude 4.5 Thinking (简易API)",
            "input": ["text"],
            "contextWindow": 200000,
            "maxTokens": 8192
          },
          {
            // 【可选】你可以把常用的 GPT-4o 也加进来
            "id": "gpt-4o",
            "name": "GPT-4o (简易API)",
            "input": ["text", "image"], // GPT-4o 支持识图，所以加个 image
            "contextWindow": 128000,
            "maxTokens": 4096
          }
        ]
      }
      // 👆【新增结束】===

    }
  },
```
📝 **记忆口诀**：以后在 OpenClaw 里，这个模型的完整名字就是 `provider名字/模型ID`，即：`jeniya_api/claude-opus-4-5-20251101-thinking`。

### 📍 第三处：把主路由切过去（agents.defaults）
最后，告诉 OpenClaw 默认使用我们刚配置的模型。找到 `"agents": { "defaults": { ... } }`：

```json
  "agents": {
    "defaults": {
      "model": {
        // 👇【修改这一行】格式是： "provider名字/模型ID"
        "primary": "jeniya_api/claude-opus-4-5-20251101-thinking"
      },

      "models": {
        // 👇【可选】设置别名，方便在日志里看
        "jeniya_api/claude-opus-4-5-20251101-thinking": {
          "alias": "Claude 4.5 Pro"
        }
      },

      // ... 其他原有配置不要动 ...
      "workspace": "/Users/你的用户名/.openclaw/workspace"
    }
  }
```

---

## 🔄 五、重启 OpenClaw，让配置生效

修改完保存后，我们需要让 OpenClaw 重新加载配置。

**🍎 macOS（官方推荐方式）：**
打开终端，执行下面这行命令强制重启网关服务：
```bash
launchctl kickstart -k gui/$(id -u)/ai.openclaw.gateway
```

**🐧 Linux / 🪟 Windows：**
因为安装方式（Docker、systemd、手动运行等）各异，最稳妥的办法是**怎么启动的，就怎么重启**。先停掉旧进程，再重新启动即可（可参考 [OpenClaw官方文档](https://github.com/openclaw/openclaw)）。

重启后，用这条通用命令检查状态：
```bash
openclaw status --deep
```
如果输出里看到了 `default claude-opus-4-5-20251101-thinking` 字样，🎉 恭喜你，配置完美成功！

---

## ✅ 六、验证测试：真的连上了吗？

1. **Dashboard 测试**：浏览器访问 `http://127.0.0.1:18789/`。
2. **发起对话**：新建对话，发一句：*“你是哪个模型？请复述你的具体版本号。”*
3. **后台查岗**：同时打开 [简易API](https://jeniya.chat/) 的后台，查看“日志”或“用量”统计。

如果 Dashboard 回复流畅，且你的中转站后台出现了扣费调用记录，说明 OpenClaw 已经完美跑在你的专属 **API中转站** 上啦！🚀

---

## ❓ 七、常见问题（Q&A）

**Q1：保存时编辑器报错（JSON Error）怎么办？**
> 💡 **99% 是逗号问题！** 列表 `[]` 或对象 `{}` 的最后一项后面**不能**有逗号，而两项之间**必须**有逗号。建议把代码复制到在线 JSON 校验工具里检查一下。

**Q2：我想换回原来的模型怎么办？**
> 💡 找到第三处的 `"primary": "jeniya_api/claude-opus-4-5..."`，把它改回原来的（例如 `openai/gpt-4o`），然后重启服务即可。

**Q3：为什么配置了没反应，一直报错？**
> 💡 重点检查两处：
> 1. `baseUrl` 后面是不是漏了 `/v1`？
> 2. `apiKey` 里的环境变量名是否和 `env` 里写的一模一样？(上面是 `"JENIYA_API_KEY"`，下面就必须是 `"${JENIYA_API_KEY}"`)。

---

## 🧠 八、一页记完的极简心智模型

最后送给大家一个思维导图，以后不管接什么平台，只要改这 3 个地方：

```text
【简易API中转站】
  ├─ Base URL: https://jeniya.chat/v1
  ├─ API Key: sk-xxxxxx
  └─ 模型 ID: claude-opus-4-5-20251101-thinking

          ↓ 写进 openclaw.json

【openclaw.json】
  ├─ 1. env.JENIYA_API_KEY                 → 存你的 Key
  ├─ 2. models.providers.jeniya_api        → 描述这个中转平台与支持的模型
  └─ 3. agents.defaults.model.primary      → 选一个当“默认主路由模型”
```

学会了这个 **OpenClaw配置教程**，你就可以彻底告别高昂的官方费用，借助 [**简易API**](https://jeniya.chat/) 这样的优质平台，低成本玩转所有前沿的 **AI大模型API** 了！

觉得有用的话，别忘了把这篇教程分享给你的极客朋友们哦！✨ Happy Coding！
