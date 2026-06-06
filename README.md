# 笨笨家园 BenBenHome

个人用 AI 聊天前端，单文件 PWA，通过 OpenRouter API 连接 Claude 等模型。主要用于创意写作和角色扮演。部署在 GitHub Pages 上，日常在 iOS Safari（添加到主屏幕）和桌面浏览器两端使用。

所有代码由 Claude 编写，我负责提需求、设计和测试。

---

## 架构概览

整个应用就是一个 `index.html`（~6000 行），加一张 `icon.PNG` 做主屏图标。没有构建工具，没有依赖，推到 main 分支就是部署。

```
index.html    ← 全部代码：HTML + CSS + JS
icon.PNG      ← iOS 主屏图标
README.md     ← 你正在看的这个
```

### 存储层

- **IndexedDB**：主存储，保存对话内容和设置。分 store 设计（settings / conversations）
- **localStorage**：轻量元数据，主题偏好等
- **`navigator.storage.persist()`**：防止 iOS Safari 静默清除 IndexedDB
- **页面生命周期钩子**：`visibilitychange` / `pagehide` / `beforeunload` 三重保险自动保存

### API 层

- 通过 **OpenRouter** 调用模型，主力 Claude，偶尔 GPT-4o / Gemini
- `provider: { order: ['Anthropic'] }` 锁定 Anthropic 直连，防止路由到 Bedrock/Vertex 导致 cache 失效
- `cache_control` 请求级别设置，1 小时 TTL，Opus 模型需要 ~4096 原生 token 才触发缓存

---

## 功能清单

### 多助手系统
- 每个助手有独立的名称、主题色、模型选择、系统提示词
- 对话按助手分组，侧边栏按助手归类
- 从 v1 单助手格式自动迁移

### 记忆库（世界书）
- 手动管理的记忆条目，可逐条启用/禁用
- 按关键词自动匹配并注入到请求上下文中
- 消息气泡上显示匹配了哪些记忆条目

### 对话管理
- 按日期分组的对话列表（Kelivo 风格）
- 对话重命名、删除
- 消息编辑、重新生成
- **对话分支**：从任意消息处分叉出新对话，继承 assistantId

### 全局搜索
- 带防抖的全文搜索，跨所有对话
- 关键词高亮，点击结果直接跳转到对应消息

### 缓存状态指示
- 模型名旁边的状态点：脉动绿 = 缓存命中，灰 = 已过期
- 缓存倒计时显示

### 图片支持
- 拍照 / 相册上传，自动压缩
- 图片以 base64 内联发送

### 费用统计面板
- 按助手分色的堆叠柱状图
- 日历热力图显示每日使用量
- 分时段统计（7天 / 30天 / 全部）
- token 用量、对话数、估算费用

### 主题切换
- **奶油暖白**（默认）：Claude.ai 风格暖色调
- **夜间护眼**：低蓝光暖色暗主题
- 切换前预写入 localStorage，避免页面闪烁

### 备份系统

**云端 WebDAV 备份：**
- 通过 CORS 代理（Express.js / Render 免费层）连接 WebDAV
- 所有 WebDAV 方法改写为 POST + `X-Method` 头，绕过 Safari 跨域限制
- 滚动时间戳备份，自动保留最近 5 份
- PROPFIND XML 解析，自动找到最新备份进行恢复

**本地备份：**
- Header 下拉菜单中的「保存到本地」/「从本地导入」
- JSON 格式，包含全部数据

---

## 踩过的坑

这些是开发过程中积累的经验，以后改代码前值得回顾：

1. **iOS Safari 存储蒸发**：不调 `navigator.storage.persist()`，IndexedDB 随时可能被系统回收，用户毫无感知
2. **OpenRouter 缓存**：必须锁 `provider.order` 到 Anthropic，路由到 Bedrock/Vertex 会静默破坏 `cache_control`
3. **Safari CORS + WebDAV**：Safari 连带正确 CORS 头也会拦截 PROPFIND/DELETE/MKCOL，必须全部走 POST 代理
4. **缓存一致性**：注入变化的时间戳、用同模型做自动标题，都会破坏提示词缓存命中——时间信息应在发送时按消息记录，标题生成换用其他模型
5. **分支对话**：必须继承父对话的 `assistantId`，否则侧边栏归类出错
6. **文件版本**：上传的文件可能是旧版，要和对话历史交叉对比确认
7. **GitHub Pages 缓存**：测试时用 `?v=N` 查询参数强制刷新 CDN 缓存

---

## 部署

推到 `main` 分支就行。GitHub Pages 设置为从 main 根目录部署。

测试时在 URL 后加 `?v=N` 绕过缓存：
```
https://<username>.github.io/benben/?v=42
```

---

## CORS 代理

独立的 Express.js 服务，部署在 Render 免费层。用于将 Safari 不支持的 WebDAV 方法（PROPFIND、DELETE、MKCOL）转写为 POST 请求。

核心逻辑：客户端发 POST，通过 `X-Method` 头指定实际的 WebDAV 方法，代理服务器转发为对应的原生请求。

---

## 待办

- [ ] 坚果云 WebDAV 适配（CORS 问题待解决，现有代理架构理论上可支持）
