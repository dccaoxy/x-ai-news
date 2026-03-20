# x-ai-news Skill

从X.com获取AI大神最新资讯的技能。

## 触发条件

用户提到以下关键词时触发：
- "AI大神"、"AI资讯"
- "X AI"、"Twitter AI"
- "获取AI动态"、"AI更新"
- "跟踪AI"、"AI大V"

---

## 分批抓取策略

由于X.com页面加载需要时间，**每次只抓3个账号**，抓完后立即推送，然后再抓下一批。

### 分批安排

| 批次 | 账号 | 状态 |
|------|------|------|
| 第1批 | @ylecun, @karpathy, @sama | 已抓 ✅ |
| 第2批 | @AndrewYNg, @darioamodei, @elonmusk | 已抓 ✅ |
| 第3批 | @demishassabis, @geoffreyhinton, @fchollet | 已抓 ✅ |
| 第4批 | @jeffdean, @goodfellow_Ian, @hardmaru | 已抓 ✅ |
| 第5批 | @rasbt, @DrJimFan, @ch402 | 已抓 ✅ |
| 第6批 | @OpenAI, @AnthropicAI, @deepmind | ⏳ 待抓 |
| 第7批 | @GoogleAI, @MetaAI, @xai | ⏳ 待抓 |
| 第8批 | @nvidia, @markzuckerberg, @danielamodei | ⏳ 待抓 |

---

## 👷 工作流程（主/子Agent分工）

### 推荐分工流程

**主Agent**负责：
1. **创建文档**：首次运行时创建飞书文档，在文档**头部建立跟踪账号表格**
2. **表格格式**：使用 Markdown 表格，包含 5 列 → `批次 | X账号 | 姓名 | 公司 | 状态`
3. **整合写入**：子Agent返回抓取结果后，主Agent统一整理格式（去掉原文、只保留翻译），写入文档
4. **更新状态**：完成一批次后更新表格中状态标记（✅ 已抓取 / ⏳ 待抓取）
5. **掌控整体结构**：保持文档格式一致性

**子Agent**负责：
1. **并行抓取**：每个子Agent负责一个批次（3个账号）
2. **独立工作**：独立打开页面 → 等待加载 → 获取快照 → 提取内容
3. **返回结果**：将提取好的内容（原文+发布时间+互动）返回给主Agent

**示例：头部跟踪表格完整代码**

```markdown
## 跟踪账号列表

| 批次 | X账号 | 姓名 | 公司 | 状态 |
|------|-------|------|------|------|
| 1 | @ylecun | Yann LeCun | Meta | ✅ 已抓取 |
| 1 | @karpathy | Andrej Karpathy | Tesla/OpenAI | ✅ 已抓取 |
| 1 | @sama | Sam Altman | OpenAI | ✅ 已抓取 |
| 2 | @AndrewYNg | 吴恩达 | DeepLearning.AI | ✅ 已抓取 |
| 2 | @darioamodei | Dario Amodei | Anthropic | ✅ 已抓取 |
| 2 | @elonmusk | Elon Musk | xai/Tesla/SpaceX | ✅ 已抓取 |
| 3 | @demishassabis | Demis Hassabis | DeepMind | ✅ 已抓取 |
| 3 | @geoffreyhinton | Geoffrey Hinton | 多伦多大学 | ✅ 已抓取 |
| 3 | @fchollet | François Chollet | Google | ✅ 已抓取 |
| 4 | @jeffdean | Jeff Dean | Google | ✅ 已抓取 |
| 4 | @goodfellow_Ian | Ian Goodfellow | Apple | ✅ 已抓取 |
| 4 | @hardmaru | David Ha | Sakana AI | ✅ 已抓取 |
| 5 | @rasbt | Sebastian Raschka | Amazon | ✅ 已抓取 |
| 5 | @DrJimFan | Jim Fan | NVIDIA | ✅ 已抓取 |
| 5 | @ch402 | Chris Olah | Anthropic | ✅ 已抓取 |
| 6 | @OpenAI | OpenAI | OpenAI | ⏳ 待抓取 |
| 6 | @AnthropicAI | Anthropic | Anthropic | ⏳ 待抓取 |
| 6 | @deepmind | DeepMind | Google DeepMind | ⏳ 待抓取 |
| 7 | @GoogleAI | Google AI | Google | ⏳ 待抓取 |
| 7 | @MetaAI | Meta AI | Meta | ⏳ 待抓取 |
| 7 | @xai | xai | Elon Musk | ⏳ 待抓取 |
| 8 | @nvidia | NVIDIA | NVIDIA | ⏳ 待抓取 |
| 8 | @markzuckerberg | Mark Zuckerberg | Meta | ⏳ 待抓取 |
| 8 | @danielamodei | Daniela Amodei | Anthropic | ⏳ 待抓取 |

**总计**: 27 个账号 · 已抓取 15 个 · 待抓取 12 个
```

**这种分工的优点**：
- 🚀 **并发效率高**：多个子Agent可以同时抓取不同批次，节省时间
- 🎯 **职责清晰**：主Agent管结构，子Agent管抓取，分工明确
- 🎨 **格式统一**：主Agent保证输出格式一致性
- 🔄 **灵活可控**：主Agent可以随时暂停、继续、调整进度

---

## 跟踪账号完整名单

>（已抓取完成的账号从列表中移除）

### 一、核心AI研究员（1人）
| X账号 | 姓名 | 公司 | 岗位/头衔 |
|-------|------|------|----------|
| @ylecun | Yann LeCun | Meta | 图灵奖得主、首席AI科学家 |
| @karpathy | Andrej Karpathy | Tesla/OpenAI | AI总监（特斯拉）、创始人（OpenAI） |
| @AndrewYNg | 吴恩达 | DeepLearning.AI | 创始人、Coursera联合创始人 |

### 二、AI公司CEO/创始人（4人）
| X账号 | 姓名 | 公司 | 岗位/头衔 |
|-------|------|------|----------|
| @sama | Sam Altman | OpenAI | CEO |
| @elonmusk | Elon Musk | xAI / Tesla / SpaceX | CEO |
| @darioamodei | Dario Amodei | Anthropic | CEO & 创始人 |
| @danielamodei | Daniela Amodei | Anthropic | President & 创始人 |
| @markzuckerberg | Mark Zuckerberg | Meta | CEO & 创始人 |

### 三、其他AI研究者（1人）
| X账号 | 姓名 | 公司 | 岗位/头衔 |
|-------|------|------|----------|
| @geoffreyhinton | Geoffrey Hinton | 多伦多大学 | 图灵奖得主、"AI教父" |

---

## 浏览器操作流程

### 1. 检查登录状态
```bash
browser action=tabs profile=openclaw
```

### 2. 打开目标账号页面
```bash
browser action=open profile=openclaw targetUrl=https://x.com/{账号}
```

### 3. 等待页面加载
```bash
exec command="powershell -Command 'Start-Sleep -Seconds 5'"
```
**重要**：停留5秒，防止触发反爬机制

### 4. 获取页面快照
```bash
browser action=snapshot profile=openclaw targetId={targetId} compact=true
```

### 5. 提取帖子内容
从snapshot中提取：
- 帖子原文（英文）
- 发布时间
- 互动数据

---

## 翻译要求

### 翻译原则
- **完整翻译**：不 summarization，逐句翻译全文
- **去掉原文**：用户要求去掉原文，只保留翻译，减小文档体积

---

## 批次状态文件

位置：`C:\Users\caoxy-test\.openclaw\data-analysis-workspace\ai_batch_status.json`

内容示例：
```json
{
  "last_batch": 1,
  "last_updated": "2026-03-06T07:40:00Z"
}
```

---

## 📄 飞书文档存档方法

### 首次创建文档

**重要**：必须传入 `owner_open_id`（用户的飞书 ID），确保用户有完整访问权限。

```json
{
  "action": "create",
  "title": "🤖 X AI News - AI大神动态汇总",
  "owner_open_id": "YOUR_FEISHU_OPEN_ID"
}
```

**参数说明**：
- `action`: "create" - 创建文档
- `title`: 文档标题（建议包含 emoji 🤖 便于识别）
- `owner_open_id`: **必填** - 用户的飞书 open_id（从 inbound 元数据 `sender_id` 获取）

**如果不传 `owner_open_id`**：
- ❌ 只有 bot app 有访问权限
- ✅ 传入后用户自动获得 `full_access` 权限

### 文档 Token 管理

创建成功后保存 `doc_token`：
```json
{
  "document_id": "YOUR_DOC_TOKEN",
  "doc_token": "YOUR_DOC_TOKEN"
}
```

后续操作使用 `doc_token` 而非 `document_id`。

---

## 📋 飞书文档输出范式

### 文档结构（只保留翻译版本）

> （用户要求去掉原文，只保留翻译，减小文档体积）

```markdown
# 🤖 X AI News - AI大神动态汇总

## 更新日期：YYYY-MM-DD

## 跟踪账号列表

| 批次 | X账号 | 姓名 | 公司 | 状态 |
|------|-------|------|------|------|
... (完整表格如上) ...

---

## 📅 YYYY-MM-DD 更新

### 批次 X: @账号1, @账号2, @账号3

---

#### 👔 [@账号] 帖子标题

**发布时间**: YYYY-MM-DD

**翻译**:
[中文翻译内容]

**互动**: N 回复 · N 转发 · N 点赞 · N 浏览

---

#### ...... (下一篇帖子)
```

### 标题规范

- ✅ **每个批次单独分段**，标明批次号和账号
- ✅ 每个帖子使用 `####` 四级标题
- ✅ **去掉原文，只保留翻译**（用户要求），减小文档体积
- ✅ 保留发布时间和互动数据

### 模块顺序（固定）

按批次倒序存档，每次抓取的新内容放在**跟踪表格下方**（文档最上端），最新内容最先看。

---

## 🔄 文档更新流程

### 1. 读取现有文档（去重用）

```json
{
  "action": "read",
  "doc_token": "YOUR_DOC_TOKEN"
}
```

**解析内容**：
- 提取已有帖子 URL 列表
- 提取已有帖子发布时间 + 作者组合
- 用于后续去重比对

### 2. 去重逻辑

```python
# 伪代码示例
existing_urls = [...]    # 从文档解析的 URL

for post in new_posts:
    if post.url in existing_urls:
        continue  # 跳过已存档帖子
    # 新帖子，添加到更新列表
    new_posts_to_add.append(post)
```

**去重策略**：
- URL 完全匹配 → 跳过
- 同一作者 + 同一发布时间 → 跳过

### 3. 写入文档（前置追加 + 全量重写）

每次抓取新批次后，将新内容插入到表格下方，然后全量重写整个文档：

```json
{
  "action": "write",
  "doc_token": "YOUR_DOC_TOKEN",
  "content": "# 🤖 X AI News - AI大神动态汇总\n\n## 更新日期: YYYY-MM-DD\n\n## 跟踪账号列表\n\n| 批次 | X账号 | 姓名 | 公司 | 状态 |\n... (完整表格如上) ...\n\n---\n\n[新批次内容]\n\n---\n\n[原有内容]\n"
}
```

**为什么这样做**：
- ✅ 最新内容在顶部，方便阅读
- ✅ 保持文档结构一致性
- ✅ 便于维护完整存档

### 4. 更新文档顶部日期

每次更新时修改顶部的"更新日期"为当前日期。

---

## ⚠️ 飞书文档注意事项

### 飞书文档

- ✅ 首次创建必须传入 `owner_open_id`
- ✅ 使用 `write` 动作全量重写（而非 append）
- ✅ 最新内容放在表格下方（文档最上端）
- ✅ **去掉原文，只保留翻译**（用户要求），减小文档体积
- ✅ 记录互动数据（点赞/转发/评论/浏览）
- ✅ 头部必须包含完整跟踪账号表格

### 推送通知

每次更新文档后，在飞书消息中说明：
- 本次抓取完成第 X 批次
- 新增 Y 条帖子
- 附上文档链接供查阅

---

## 文件路径

| 文件 | 路径 |
|------|------|
| 账号名单 | `C:\Users\caoxy-test\.openclaw\data-analysis-workspace\ai_experts.md` |
| 批次状态 | `C:\Users\caoxy-test\.openclaw\data-analysis-workspace\ai_batch_status.json` |
| 文档配置 | `C:\Users\caoxy-test\.openclaw\data-analysis-workspace\ai_doc_config.json` |
| Skill 目录 | `C:\Users\caoxy-test\.openclaw\data-analysis-workspace\skills\x-ai-news` |

---
