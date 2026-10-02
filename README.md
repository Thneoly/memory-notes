# memory-notes · 跨应用共享记忆系统的写端 demo

> **OctoSense 黑客松参赛项目 · 商店应用赛道**
> 演示"每条便签都可被你的助手记住"的交互模型。

---

## 一句话

`Memory Notes` 是一个便签应用，**每条便签**都有一个独立的 **Remember** 按钮。点一下 → 走 `octos.turn.start` 让助手记住这条；失败 → 便签**行记下来**——本地永远可用，AI 是渐进增强层。

> 📘 完整方案与三仓联合演示见 [`docs/JOINT-DEMO.md`](docs/JOINT-DEMO.md)（路径：../os-memory/docs/JOINT-DEMO.md）

---

## 角色

在三仓架构里，本仓是**写端**——演示"用 App 自己的 Agent 把内容写入自己 peer"：

```
   ┌────────────┐
   │memory-notes│  本仓
   │  （写端）   │   ↑ 每条便签独立按钮
   │   ┌──┐    │   ↓ host.request("octos.turn.start", ...)
   │   │记│  → │     │ 失败 → "kept locally: <text>"
   │   │住│  → │     │ 成功 → "Remembered ✅ <text>"
   │   └──┘     │
        ↓
       peer ─────→ os.memory (主作品) 汇聚
```

**与 `digest` 的对比**：

| | notes（本仓） | digest |
|---|---|---|
| 颗粒度 | **逐条**（每行一个） | 聚合（一次 = 一次完整更新） |
| 失败语义 | 单条独立降级 | 整段聚合降级 |
| 状态行 | "Asking → Remembered / kept locally" | "Asking → From your assistant / from your local interests" |

---

## 本仓特异

| 字段 | 值 |
|---|---|
| 应用 id | `memory-notes` |
| 版本 | `0.1.0` |
| 命名空间 | 商店应用（自有 id） |
| 提交路径 | `octo check` + `hub check --publisher-key` |
| 资源上限 | 16 MiB storage · 16,777,216 bytes |
| Agent profile | `read-only` |
| Capabilities | `storage` + `octos.turn.start` |
| Platforms | `windows`（其他平台未验证，**不假装**） |

### Gate 实测

```bash
$ python ../OctoScript-App-Design-Flow/tools/octo check bundle
memory-notes 0.1.0 — PASSED
```

警告：
- `publisher-signature: unsigned`（首次可不上签）
- `listing.json still holds template placeholders`（评审者注意！）

### 关键源码（`bundle/main.splash` · 摘要）

- 状态变量 `remember_status`（非 const，遵循"模板先例"——不发明 `const` 语法）
- `remember(text)` 函数：
  1. 立即 `remember_status = "Asking the assistant to remember: " + text`
  2. 异步 `host.request("octos.turn.start", {text: "Remember: " + text}, fn(r){...})`
  3. 回调成功：`remember_status = "Remembered ✅ " + text`
  4. 回调失败：`remember_status = "No assistant on this device — kept locally: " + text`
- 持久化：`notes.json` 数组，`add`/`remove` 即时 `save`
- UI：顶部 TextInput + 中部 ScrollYView 列表（每行 Remember + 🗑）+ 底部 hint

---

## 关键决策（锚定）

完整决策清单见 [`docs/JOINT-DEMO.md` § 3](../os-memory/docs/JOINT-DEMO.md)。本仓最相关：

- **A3.1 诚实降级是核心 UX**——AI 失败必须有本地降级路径，便签绝不能因 AI 不可用而坏 UI
- **A1.2 不发明 API**——`const` 没有模板先例就换掉；不猜未文档化字段
- **A1.4 可见窗口演示**——用户屏幕直接看，3 仓并排

---

## 演示与复现

### 截图

- `bundle/screenshots/01-main.png` — 主屏
- `build/demo-notes.png` — 本地演示快照

### 复现命令

```bash
git clone https://github.com/Thneoly/memory-notes.git
cd memory-notes

# 自检（已 PASSED）
python ../OctoScript-App-Design-Flow/tools/octo check bundle

# 启动 card-host 跑起来（独立 16 MiB）
card-host bundle --port 8141

# 截图
python ../OctoScript-App-Design-Flow/tools/octo shot 8141 bundle/screenshots/01-main.png
```

### 现场试加一条 → Remember → 看诚实降级

1. 输入 "Wash the car on Saturday" → Enter
2. 点新便签的 **Remember**
3. 状态行变化：`Asking → ✅ Remembered`（成功）或 `Asking → kept locally`（降级）
5. 关掉 card-host → 再点 → 确认"kept locally"路径生效

---

## 已知边界

- **Windows only** — `platforms: ["windows"]`
- **未签名** — `publisher-signature: unsigned`（首次可）
- **未转 public** — 赛前必转 public
- **bundle/manifest.json 未提交** — `octo check` 内部 stamp 重算，内容已一致，赛前需要 `git add bundle/manifest.json`

---

## 项目结构

```
memory-notes/
├── README.md            ← 你在这
├── BRIEF.md             ← 简报
├── AGENTS.md            ← 跨 agent 开发说明
├── CLAUDE.md / GEMINI.md ← @AGENTS.md 转发
├── bundle/
│   ├── manifest.json    ← stamp manifest（**未提交版本**待赛前 add）
│   ├── listing.json     ← store 元数据（publisher 占位待替换）
│   ├── main.splash      ← 主程序
│   ├── screenshots/01-main.png
│   └── assets/icon.svg
├── build/
│   └── demo-notes.png
└── .local-state/memory-notes/notes.json  ← 运行时便签数据
```

---

## 提交前 TODO

见 [`docs/JOINT-DEMO.md` § 6](../os-memory/docs/JOINT-DEMO.md)。本仓特异：
- [ ] git add `bundle/manifest.json`（stamp 已重算）
- [ ] 生成 Packet（review.json + 7 问 REVIEW-ANSWERS.md）
- [ ] publisher 占位替换

---

## 参考

- 联合演示：[`docs/JOINT-DEMO.md`](../os-memory/docs/JOINT-DEMO.md)
- 比赛规则：[App Hub docs/PUBLISHING.md](https://github.com/OctoSense-org/OctoSense-App-Hub/blob/main/docs/PUBLISHING.md)

---

*生成于 2026-10-02 · 锚定大会话 `6d0c2850...`（2026-09-29 → 2026-10-02）· mcp memory #30*