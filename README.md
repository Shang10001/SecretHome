# 笨笨家园 BenBenHome

一个单 HTML 文件的 AI 聊天前端，通过 OpenRouter 调用 Claude / GPT 等模型。
不需要构建、不需要服务器，传到 GitHub Pages 打开就能用，也可以添加到手机主屏幕当 PWA 用。

---

## 功能一览

**聊天核心**
- 流式输出（SSE streaming）
- 发送图片（自动压缩到 800px / JPEG 70%，嵌入对话数据，备份时一起走）
- 多对话管理（新建 / 切换 / 删除）
- 消息编辑（点 `⋮` → 编辑，直接在气泡里改）
- 对话分支（从任意消息往上或往下拆成新对话）

**模型 & 参数**
- 支持 Claude Opus 4.7 / 4.6、Sonnet 4.5、GPT-4o，也可以手动输入任意 OpenRouter 模型 ID
- Prompt Caching（5 分钟 / 1 小时 TTL），缓存状态实时显示
- Extended Thinking / 思考链（Low / Medium / High / Max 四档力度）
- System Prompt、Temperature、Max Tokens、上下文消息数 均可调

**数据 & 备份**
- 自动保存到浏览器 IndexedDB，刷新不丢
- WebDAV 备份 / 恢复（支持"完全覆盖"和"智能合并"两种模式）
- 从 Kelivo 导入对话记录
- 每条消息记录 token 用量、缓存命中、花费

**外观**
- 两套主题：奶油暖白（参考了Claude.ai配色） / 夜间护眼
- 移动端适配，键盘弹出不跳页（VisualViewport 钉位）

---

## 文件结构

```
index.html   ← 全部代码都在这一个文件里
icon.png     ← 主屏幕图标（正方形 PNG，换图标只需替换这个文件）
README.md
```

---

## 使用方法

1. 在 GitHub 上开启 Pages（Settings → Pages → Source 选 `main` 分支）
2. 用手机 Safari 打开 `https://你的用户名.github.io/BenBenHome/`
3. 点"添加到主屏幕"就变成全屏 App
4. 进 App 后打开设置，填入 OpenRouter API Key，选模型，开聊

---

## 换图标

1. 准备一张**正方形 PNG**（建议 1024×1024，铺实色底，别留透明，别自己磨圆角）
2. 命名为 `icon.png`，在 GitHub 上覆盖掉旧的
3. 手机上删掉旧的主屏幕图标 → 重新"添加到主屏幕"（iOS 会缓存旧图标）

---

## WebDAV 备份

在设置 → 备份页填入 WebDAV 地址、用户名、密码，点测试确认连通，然后：
- **备份**：把所有对话 + 设置 + 记忆打包成 `benben_backup.json` 上传到 WebDAV
- **恢复 — 智能合并**：只补充本地没有的对话和记忆，不碰已有数据
- **恢复 — 完全覆盖**：用备份内容替换本地所有数据（WebDAV 配置本身不受影响）

---

## 备忘

- **图片 token 估算**：`宽 × 高 / 750`，一张 800×800 ≈ 853 tok
- **缓存命中的图片**只花原价 10% 的输入费用
- **上下文消息数**设为 0 表示发送全部历史；设一个偶数（比如 20）可以控制每轮发送的历史长度，省 token
- iOS 上如果键盘弹出时页面跳动，确认 viewport meta 里有 `interactive-widget=resizes-content`
