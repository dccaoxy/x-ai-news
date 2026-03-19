# X.com (Twitter) 新闻聚合

## 技能说明

从 X.com 上关注指定博主的推文，抓取最新内容，翻译成中文并整理到飞书文档，同时推送消息通知。支持去重，只抓取上次更新后新发布的内容。

## 触发条件

用户发送以下内容时激活：
- "x news"
- "twitter news"
- "更新X新闻"
- "抓取X推文"

---

## 📄 飞书文档创建方法

### 首次创建文档

**重要**：必须传入 `owner_open_id`（用户的飞书 ID），确保用户有完整访问权限。

```json
{
  "action": "create",
  "title": "🌐 X.com 新闻聚合",
  "owner_open_id": "YOUR_FEISHU_OPEN_ID"  // 替换为你的飞书 ID
}
```

**参数说明**：
- `action`: "create" - 创建文档
- `title`: 文档标题（建议包含 emoji 🌐 便于识别）
- `owner_open_id`: **必填** - 用户的飞书 open_id（从 inbound 元数据 `sender_id` 获取）

**如果不传 owner_open_id**：
- ❌ 只有 bot app 有访问权限
- ✅ 传入后用户自动获得 `full_access` 权限

### 文档 Token 管理

创建成功后保存 `doc_token`：
```json
{
  "document_id": "YOUR_DOC_TOKEN",
  "doc_token": "YOUR_DOC_TOKEN"  // 替换为实际文档 token
}
```

后续操作使用 `doc_token` 而非 `document_id`。

---

## 📋 飞书文档输出范式

### 文档结构（Markdown 格式）

```markdown
# 🌐 X.com 新闻聚合

## 更新日期：YYYY-MM-DD

---

## 📅 YYYY-MM-DD 更新

### 🇺🇸 美国政治

##### **1. [@username] - [推文标题/主题]**
**原文**: [英文原文（如果简短）]
**翻译**: [中文翻译]
**链接**: [原文链接](https://x.com/...)

##### **2. [@username] - [推文标题/主题]**
**原文**: [英文原文]
**翻译**: [中文翻译]
**链接**: [原文链接](https://x.com/...)

---

### 🤖 AI 科技

##### **1. [@username] - [推文标题/主题]**
**原文**: [英文原文]
**翻译**: [中文翻译]
**链接**: [原文链接](https://x.com/...)

---

### 💰 经济金融

##### **1. [@username] - [推文标题/主题]**
**原文**: [英文原文]
**翻译**: [中文翻译]
**链接**: [原文链接](https://x.com/...)

---

**📄 文档链接**: https://xxx.feishu.cn/docx/[doc_token]

---

*下次更新将在[配置时间]自动执行*
```

### 分类（固定顺序）

1. 🇺🇸 **美国政治**
2. 🏛️ **全球政治**
3. 🤖 **AI 科技**
4. 💼 **商业产业**
5. 💰 **经济金融**
6. 🇨🇳 **中国观察**

### 标题规范

- ✅ **格式**: `**[@username] - [推文主题]**`
- ✅ 使用 `#####`（五级标题）给每条推文分段
- ✅ 保持干净易读

### 内容规范

| 字段 | 格式 | 要求 |
|------|------|------|
| 原文 | 引用原文 | 保留英文原文，方便对照 |
| 翻译 | 完整中文翻译 | 保持原意，准确专业 |
| 链接 | 原文链接 | 必须可点击跳转 |

---

## 🔄 文档更新流程

### 1. 读取现有文档（去重用）

```json
{
  "action": "read",
  "doc_token": "N2jsd2HMpoCrEXx744CcbmeJnIb"
}
```

**解析内容**：
- 提取已有推文链接列表
- 提取已有日期，用于去重
- 只抓取上次更新后新发推文

### 2. 去重逻辑

```python
# 伪代码示例
existing_urls = [...]  # 从文档解析的链接

for tweet in new_tweets:
    if tweet.url in existing_urls:
        continue  # 跳过已发布
    # 新推文，添加到更新列表
    new_tweets_to_add.append(tweet)
```

**去重策略**：
- URL 完全匹配 → 跳过
- 如果同一推文多次转发，只保留最新

### 3. 浏览器操作规范

### 打开用户主页

```json
{
  "action": "open",
  "profile": "openclaw",
  "targetUrl": "https://x.com/username"
}
```

### 获取推文内容

使用 browser snapshot 获取页面内容，提取最新推文：
- 获取每条推文的文本内容
- 获取推文链接
- 获取发布时间
- 筛选出上次更新后新发推文

### 关闭页面（重要！）

**每次获取完一个用户页面后，必须关闭标签页**，避免浏览器资源占用：

```json
{
  "action": "close",
  "profile": "openclaw",
  "targetId": "target-page-id"
}
```

---

### 4. 写入文档（全量重写）

```json
{
  "action": "write",
  "doc_token": "N2jsd2HMpoCrEXx744CcbmeJnIb",
  "content": "# 🌐 X.com 新闻聚合\n\n## 更新日期：2026-03-17\n\n---\n\n## 📅 2026-03-17 更新\n\n[完整 Markdown 内容...]"
}
```

**为什么用 write 而不是 append**：
- ✅ 可以控制整体结构
- ✅ 确保最新内容在文档最上端
- ✅ 便于维护文档格式一致性

### 5. 更新文档顶部日期

每次更新时修改顶部的"更新日期"为当前日期。

---

## 🌐 关注列表

**需要关注的博主**（你可以在这里修改添加/删除）：

| 分类 | 用户名 | 主题 | URL |
|------|----------|------|-----|
| 🇺🇸 美国政治 | @realDonaldTrump | 特朗普动态 | https://x.com/realDonaldTrump |
| 🇺🇸 美国政治 | @elonmusk | 马斯克 | https://x.com/elonmusk |
| 🤖 AI 科技 | @sama | Sam Altman | https://x.com/sama |
| 🤖 AI 科技 | @ylecun | Yann LeCun | https://x.com/ylecun |
| 🤖 AI 科技 | @demishassabis | Demis Hassabis | https://x.com/demishassabis |
| 💰 经济金融 | @elerianm | Mohamed El-Erian | https://x.com/elerianm |
| 💰 经济金融 | @jmblanchard | Joseph Blanchard | https://x.com/jmblanchard |
| 🇺🇸 政治分析 | @ggreenberg | Noah Greenberg | https://x.com/ggreenberg |
| 🌍 国际政治 | @bianagoled | Bianna Golodryga | https://x.com/bianagoled |

**添加新博主**：在表格中添加一行，指定分类、用户名、主题、URL 即可。

---

## 📤 飞书消息推送范式

### 推送格式

```markdown
🌐 **X.com 新闻聚合**

*更新日期: YYYY-MM-DD*

---

### 🇺🇸 美国政治

**[@elonmusk] - 关于 AI 监管**

**翻译**:
[中文翻译内容]

原文链接: https://x.com/...

---

✅ 本次更新 X 条，已整理到飞书文档
📄 文档链接: https://xxx.feishu.cn/docx/[doc_token]
```

### 推送要求

1. **分类推送**：按分类分开，每类不超过 5 条推文，超长拆分多条
2. **必须包含**：用户名 + 主题 + 翻译 + 原文链接
3. **文档通知**：最后说明本次更新条数 + 文档链接

---

## ⏰ 定时任务配置

### Cron 任务

```yaml
# 每天 8:00 自动执行
x-news-daily:
  schedule: "0 8 * * *"
  task: "获取 X.com 关注博主最新推文并更新文档"
```

### 执行流程

```
08:00 → 触发 x-news 技能
      → 逐个打开关注博主 X 主页
      → 获取最新推文
      → 去重检查
      → 翻译整理
      → 写入飞书文档（新内容在顶部）
      → 推送飞书消息通知
      → 关闭浏览器页面
```

---

## ⚠️ 注意事项

### 飞书文档

- ✅ 首次创建必须传入 `owner_open_id`
- ✅ 使用 `write` 动作全量重写（而非 append）
- ✅ 最新内容放在文档最上端
- ✅ 使用 emoji 分类，便于浏览

### 浏览器管理

- ✅ 每次获取完一个用户页面后**必须关闭页面**
- ✅ 使用 `profile: openclaw`
- ✅ 避免打开过多标签页导致资源泄漏

### 去重逻辑

- ✅ 基于 URL 去重
- ✅ 只抓取上次更新后新发推文

### 翻译要求

- ✅ 原文完整保留，方便对照
- ✅ 翻译准确通顺，专业术语正确
- ✅ 推文链接必须可点击跳转

---

## 🛠️ 技术备注

### 工具使用

| 操作 | 工具 | 参数 |
|------|------|------|
| 创建文档 | `feishu_doc` | `action: create` |
| 读取文档 | `feishu_doc` | `action: read` |
| 写入文档 | `feishu_doc` | `action: write` |
| 打开页面 | `browser` | `action: open` |
| 获取内容 | `browser` | `action: snapshot` |
| 关闭页面 | `browser` | `action: close` |
| 推送消息 | `message` | `action: send` |

### 关键配置

- browser profile: `openclaw`
- 获取页面快照后提取推文内容
- 翻译使用模型自带能力
- 飞书文档操作使用 `feishu_doc` 工具

---

## 📁 文件结构

```
skills/x-news/
└── SKILL.md          # 本文件
```

### 相关配置

- **Cron 任务**: `x-news-daily`（每天 8:00）
- **文档标题**: `🌐 X.com 新闻聚合`
- **推送目标**: 用户飞书 ID（从 inbound 元数据获取）
