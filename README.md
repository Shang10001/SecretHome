# 笨笨家园

单文件 AI 聊天前端，通过 OpenRouter API 调用 Claude 等模型。零后端，纯前端，一个 `index.html` 搞定一切。

## 功能

### 对话

- 流式输出（SSE streaming）
- 多对话管理，按日期分组显示
- 对话重命名、删除
- 消息编辑（点击 `···` 菜单）
- 对话分支：从任意消息处往上或往下拆分出新对话
- 自动标题（取第一条消息前 30 字）

### 模型与参数

- 预置模型：Claude Opus 4.7 / 4.6、Sonnet 4.5、GPT-4o
- 支持自定义模型 ID（任何 OpenRouter 支持的模型）
- 可调参数：Max Tokens、Temperature、上下文消息数
- Extended Thinking（思考链）：支持开关和力度调节（low/medium/high/max），思考过程可折叠查看

### Prompt Caching

- 自动启用 OpenRouter 的 `cache_control: ephemeral`
- 缓存 TTL 可选 5 分钟或 1 小时
- 每条回复显示缓存状态：`cached` / `write` / `no cache`
- 顶部缓存指示器 + 倒计时
- 模型名旁绿色脉冲圆点 = 缓存在线，灰色 = 已过期

### 费用追踪

- 每条回复显示 input/output token 数、缓存命中数、单条费用
- 底部状态栏显示当前对话累计费用、消息数、总 token 数

### 数据存储

- IndexedDB 存对话内容，localStorage 存设置索引
- 自动从旧版 localStorage 迁移到 IndexedDB
- 存储用量显示（设置 → 备份页）

### 备份

- **WebDAV 云备份**：支持 NextCloud、Synology、InfiniCLOUD 等，一键备份/恢复，支持完全覆盖和智能合并两种恢复模式（坚果云因 CORS 限制无法使用）
- **本地文件备份**：导出/导入 JSON 文件
- **自动备份**：每 10 条新消息自动下载备份文件（可开关）
- **Kelivo 导入**：从 Kelivo 备份包迁移对话记录

### PWA

- 支持添加到主屏幕
- 内联 Service Worker 和 manifest，无需额外文件

## 部署

是一个单独的 HTML 文件，随便怎么部署：

- 直接浏览器打开 `index.html`
- 放到 GitHub Pages
- 扔到任何静态托管服务
- 本地 `python -m http.server` 起个服务也行

第一次打开去设置里填 OpenRouter API Key 就能用了。

## 技术栈

纯手写，无框架无构建：

- HTML / CSS / Vanilla JS
- 字体：Noto Sans SC + JetBrains Mono + Source Serif 4
- 存储：IndexedDB + localStorage
- API：OpenRouter（兼容 OpenAI 格式）

## 文件结构

就一个文件：

```
index.html    ← 所有 HTML + CSS + JS 都在这里
```

## 缓存省钱小贴士

- System Prompt 改动会触发一次 cache write，之后持续 cache hit
- 缓存有效期内连续对话，prompt 部分按缓存价计费（通常便宜 90%）
- 上下文消息数设置可以控制发送量，减少不必要的 token 消耗
- 底部状态栏和每条回复的 token 信息帮你随时掌握用量
