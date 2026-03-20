# x-ai-news Skill

从X.com获取AI大神最新资讯的技能。

## 触发条件

用户提到以下关键词时触发：
- "AI大神"、"AI资讯"
- "X AI"、"Twitter AI"
- "获取AI动态"、"AI更新"
- "跟踪AI"、"AI大V"

---

## 分批抓取策略（带间隔控制）

由于X.com有反爬机制，采用**分批抓取+间隔等待**策略：

### 抓取流程

```
批次1 (3个账号) → 推送到飞书 → 等待30秒 → 批次2 (3个账号) → 推送到飞书 → 等待30秒 → ...
```

### 间隔设置

| 间隔类型 | 时长 | 说明 |
|---------|------|------|
| **批次间隔** | 30-60秒 | 每批抓取完成后等待，避免触发反爬 |
| **页面停留** | 5-10秒 | 每个账号页面加载后停留时间 |
| **总时长估算** | 8-12分钟 | 抓取全部27个账号 |

### 分批安排

| 批次 | 账号 | 说明 |
|------|------|------|
| 第1批 | @ylecun, @karpathy, @sama | 核心AI研究员 |
| 第2批 | @AndrewYNg, @darioamodei, @elonmusk | AI领袖 |
| 第3批 | @demishassabis, @geoffreyhinton, @fchollet | 深度学习先驱 |
| 第4批 | @jeffdean, @goodfellow_Ian, @hardmaru | Google/Apple研究员 |
| 第5批 | @rasbt, @DrJimFan, @ch402 | 年轻AI研究者 |
| 第6批 | @OpenAI, @AnthropicAI, @GoogleDeepMind | AI公司官方 |
| 第7批 | @GoogleAI, @MetaAI, @xai | 大公司AI部门 |
| 第8批 | @nvidia, @markzuckerberg, @danielamodei | 科技巨头 |

**总计**: 27 个账号

---

## 👷 工作流程（增量式抓取+实时推送）

### 执行流程

```
开始
  ↓
创建/读取飞书文档
  ↓
批次1抓取 (3个账号)
  ↓
去重检查（与文档已有内容对比）
  ↓
追加新内容到飞书文档
  ↓
推送飞书消息（含文档链接）
  ↓
等待30秒
  ↓
批次2抓取 (3个账号)
  ↓
...重复直到所有批次完成...
  ↓
完成，发送总结消息
```

### 详细步骤

**Step 1: 准备阶段**
- 检查或创建飞书文档
- 读取文档中已有内容（用于去重）

**Step 2: 循环抓取（每批次）**
```python
for i, batch in enumerate(batches):
    # 1. 抓取当前批次（3个账号）
    fetch_batch(batch)
    
    # 2. 去重检查
    new_posts = deduplicate(fetched_posts, existing_posts)
    
    # 3. 追加到文档并推送通知
    if new_posts:
        append_to_document(new_posts)
        # 发送通知但不等待用户回复
        send_notification(batch_num=i+1, new_posts_count=len(new_posts), doc_link=doc_link)
    
    # 4. 自动间隔等待（最后一批除外）
    # ⚠️ 重要：使用 sleep 自动等待，不需要用户确认
    if i < len(batches) - 1:
        sleep(30)  # 自动等待30秒，继续下一批次
```

**关键说明**：
- ✅ 每批次完成后**自动**推送消息给用户
- ✅ 使用 `sleep 30` **自动等待**，不阻塞执行
- ✅ **不需要用户回复确认**，自动继续下一批次
- ✅ 用户会收到每批次的进度通知，但无需操作

**Step 3: 完成推送**
- 发送最终总结消息
- 报告总新增帖子数

---

## 🔄 去重逻辑

### 去重策略

```python
# 从文档中解析已有帖子
existing_posts = parse_existing_posts(document_content)

for post in fetched_posts:
    # 策略1: 推文URL完全匹配
    if post.url in existing_urls:
        continue
    
    # 策略2: 作者+发布时间组合匹配
    post_key = f"{post.author}_{post.timestamp}"
    if post_key in existing_keys:
        continue
    
    # 策略3: 内容相似度>90%（避免同一推文不同格式）
    if similarity(post.content, existing_contents) > 0.9:
        continue
    
    # 新帖子，添加到列表
    new_posts.append(post)
```

### 去重标识

| 字段 | 说明 |
|------|------|
| `tweet_url` | 推文永久链接（最可靠）|
| `author_timestamp` | 作者+发布时间组合 |
| `content_hash` | 内容哈希（检测微小变化）|

---

## 🌐 浏览器操作规范

### 抓取参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `page_load_wait` | 5秒 | 页面加载等待时间 |
| `page_stay_time` | 3-5秒 | 页面停留时间（模拟真实浏览）|
| `batch_interval` | 30-60秒 | 批次间间隔 |
| `profile` | openclaw | 浏览器配置 |

### 操作流程（⚠️ 必须严格遵循）

```bash
# 1. 打开账号页面
browser action=open profile=openclaw url=https://x.com/{账号}
# 返回 targetId，必须保存用于后续关闭

# 2. 等待加载（5秒）
exec sleep 5

# 3. 获取快照
browser action=snapshot profile=openclaw targetId={targetId}

# 4. 停留3-5秒（模拟阅读）
exec sleep 3

# 5. 【强制】关闭页面（释放资源）
# ⚠️ 必须执行，否则会导致浏览器标签页堆积
browser action=close profile=openclaw targetId={targetId}
```

### 🚨 资源释放规范

**必须关闭页面的情况**：
- ✅ 每个账号抓取完成后立即关闭
- ✅ 批次内所有账号抓取完成后检查并关闭残留
- ✅ 任务中断/异常时清理所有已打开页面
- ✅ 子代理执行时必须传递 targetId 并确保关闭

**检查命令**：
```bash
# 检查当前所有标签页
browser action=tabs profile=openclaw

# 批量关闭所有 X 相关页面（清理用）
# 遍历 tabs 结果，关闭所有 url 包含 x.com 的页面
```

**子代理执行要求**：
- 子代理必须在 `finally` 块或确保执行的代码段中关闭页面
- 父代理应在子代理完成后检查并清理残留标签页
- 记录每个打开的 targetId，确保可追溯关闭

---

## 📄 飞书文档存档方法

### 首次创建文档

```json
{
  "action": "create",
  "title": "🤖 X AI News - AI大神动态汇总",
  "owner_open_id": "YOUR_FEISHU_OPEN_ID"
}
```

### 文档结构

```markdown
# 🤖 X AI News - AI大神动态汇总

## 更新日期：YYYY-MM-DD

## 跟踪账号列表

| 批次 | X账号 | 姓名 | 公司 | 状态 |
|------|-------|------|------|------|
| 1 | @ylecun | Yann LeCun | Meta | ✅ |
| 1 | @karpathy | Andrej Karpathy | Tesla/OpenAI | ✅ |
... (完整27个账号) ...

---

## 📅 YYYY-MM-DD 更新

### 批次 X: @账号1, @账号2, @账号3

#### 👔 [@账号] 帖子标题

**发布时间**: YYYY-MM-DD

**翻译**:
[中文翻译内容]

**互动**: N 回复 · N 转发 · N 点赞 · N 浏览

---
```

### 更新方式

**追加模式**（不是全量重写）：
```json
{
  "action": "append",
  "doc_token": "YOUR_DOC_TOKEN",
  "content": "\n\n### 批次 X: ...\n\n[新内容]"
}
```

**更新日期**：每次执行时更新顶部的"更新日期"

---

## 📤 飞书消息推送范式

### 每批次推送（实时）

```markdown
🤖 **X AI News - 批次 X 完成**

✅ 已抓取: @账号1, @账号2, @账号3
📝 新增帖子: Y 条
⏱️ 下一批次: 30秒后继续

📄 文档链接: https://xxx.feishu.cn/docx/ABC123
```

### 最终总结推送

```markdown
🤖 **X AI News - 全部抓取完成**

✅ 总批次: 8/8
📊 总新增: XX 条帖子
⏱️ 总耗时: X 分钟

📄 查看完整文档: https://xxx.feishu.cn/docx/ABC123
```

---

## ⚠️ 注意事项

### 反爬规避

- ✅ **批次间隔**: 30-60秒（必须）
- ✅ **页面停留**: 每个页面停留3-5秒
- ✅ **随机延迟**: 可适当添加随机1-3秒延迟
- ✅ **关闭页面**: 每抓取完一个账号立即关闭页面

### 文档管理

- ✅ **首次创建**: 必须传入 `owner_open_id`
- ✅ **去重检查**: 与已有内容对比，避免重复
- ✅ **追加模式**: 使用 append 而非 write（保留历史）
- ✅ **去掉原文**: 只保留中文翻译

### 异常处理

- ❌ **抓取失败**: 记录错误，继续下一批次
- ❌ **页面加载超时**: 重试1次，仍失败则跳过
- ❌ **登录失效**: 停止执行，通知用户重新登录

---

## 🛠️ 技术备注

### 工具使用

| 操作 | 工具 | 参数 |
|------|------|------|
| 创建文档 | `feishu_doc` | `action: create` |
| 读取文档 | `feishu_doc` | `action: read` |
| 追加内容 | `feishu_doc` | `action: append` |
| 打开页面 | `browser` | `action: open` |
| 获取内容 | `browser` | `action: snapshot` |
| 关闭页面 | `browser` | `action: close` |
| 推送消息 | `message` | `action: send` |
| 等待间隔 | `exec` | `sleep` |

### 批次配置

```python
batches = [
    ["ylecun", "karpathy", "sama"],
    ["AndrewYNg", "darioamodei", "elonmusk"],
    ["demishassabis", "geoffreyhinton", "fchollet"],
    ["jeffdean", "goodfellow_Ian", "hardmaru"],
    ["rasbt", "DrJimFan", "ch402"],
    ["OpenAI", "AnthropicAI", "GoogleDeepMind"],
    ["GoogleAI", "MetaAI", "xai"],
    ["nvidia", "markzuckerberg", "danielamodei"]
]
```

---

## 💓 Heartbeat 自动续传机制

### 机制说明

为防止长时间任务中断，Skill 执行时自动设置 Heartbeat 任务：

**执行流程：**
```
1. 开始执行 Skill
   ↓
2. 创建 job_list.json 记录进度
   ↓
3. 设置 Heartbeat 检查（每5分钟）
   ↓
4. 执行批次抓取
   ↓
5. 如果中断：
   - Heartbeat 检测到未完成任务
   - 自动继续执行
   ↓
6. 全部完成后：
   - 更新 job_list.json 为完成状态
   - 终止 Heartbeat 检查
```

### Job List 文件

**位置**: `job_list.json`

**内容示例**:
```json
{
  "task": "x-ai-news",
  "total_batches": 8,
  "completed_batches": [1, 2],
  "current_batch": 3,
  "status": "running",
  "doc_token": "YOUR_DOC_TOKEN"
}
```

---

## 📝 执行记录机制（防止重复创建文档）

### 执行记录文件

**位置**: `/Users/caoxy/.openclaw/agents/news-officer/x_ai_news_config.json`

**作用**: 记录首次创建的飞书文档信息，后续执行都更新同一文档

**内容示例**:
```json
{
  "first_execution": "2026-03-20T12:00:00+08:00",
  "execution_count": 5,
  "doc_token": "D2THdz61yoR3qzxDFFMcCQgYnHd",
  "doc_url": "https://feishu.cn/docx/D2THdz61yoR3qzxDFFMcCQgYnHd",
  "doc_title": "🤖 X AI News - AI大神动态汇总",
  "history": [
    {
      "execution": 1,
      "time": "2026-03-20T12:00:00+08:00",
      "batches": [1, 2, 3],
      "tweets": 25
    },
    {
      "execution": 2,
      "time": "2026-03-20T14:30:00+08:00",
      "batches": [4, 5],
      "tweets": 18
    }
  ]
}
```

### 执行流程（带执行记录和资源清理）

```python
def execute_skill():
    opened_tabs = []  # 记录所有打开的页面 targetId
    
    try:
        # 1. 检查执行记录
        config = read_config()
        
        if config.exists():
            doc_token = config.doc_token
            execution_count = config.execution_count + 1
        else:
            doc = create_feishu_doc()
            doc_token = doc.token
            config = {
                "first_execution": now(),
                "execution_count": 1,
                "doc_token": doc_token,
                "doc_url": doc.url,
                "history": []
            }
            save_config(config)
        
        # 2. 执行抓取（记录每个打开的页面）
        for account in accounts:
            target_id = browser_open(f"https://x.com/{account}")
            opened_tabs.append(target_id)  # 记录 targetId
            
            sleep(5)
            content = browser_snapshot(target_id)
            
            # 立即关闭页面
            browser_close(target_id)
            opened_tabs.remove(target_id)  # 从记录中移除
        
        # 3. 更新文档
        update_feishu_doc(doc_token, results)
        
        # 4. 记录执行历史
        config.history.append({
            "execution": config.execution_count,
            "time": now(),
            "batches": results.batches,
            "tweets": results.tweet_count
        })
        save_config(config)
        
    finally:
        # 5. 【强制】清理所有残留页面
        cleanup_all_tabs(opened_tabs)

def cleanup_all_tabs(opened_tabs):
    """清理所有已打开的标签页"""
    # 关闭记录的页面
    for target_id in opened_tabs:
        try:
            browser_close(target_id)
        except:
            pass  # 忽略已关闭的页面错误
    
    # 额外检查：关闭所有 X 相关页面
    all_tabs = browser_list_tabs()
    for tab in all_tabs:
        if "x.com" in tab.url and tab.type == "page":
            try:
                browser_close(tab.targetId)
            except:
                pass
```

### 关键规则

1. **首次执行**: 创建飞书文档，记录 doc_token
2. **后续执行**: 读取 config，使用已有 doc_token 更新同一文档
3. **执行计数**: 每次执行 +1，记录在历史中
4. **断点续传**: 结合 job_list.json，从上次中断处继续

### 文件位置

| 文件 | 位置 | 作用 |
|------|------|------|
| `x_ai_news_config.json` | `~/.openclaw/agents/news-officer/` | 执行记录、文档链接 |
| `job_list.json` | `~/.openclaw/agents/news-officer/` | 批次进度、当前状态 |
| `x_ai_news.log` | `~/.openclaw/agents/news-officer/` | 执行日志（可选） |

### Heartbeat 检查逻辑

```python
# Heartbeat 每5分钟执行
def heartbeat_check():
    job = read_job_list()
    if job.status == "running" and job.current_batch <= job.total_batches:
        # 有未完成任务，自动继续
        continue_fetching(job.current_batch)
    elif job.status == "completed":
        # 任务完成，跳过
        pass
```

---

## 📁 文件结构

```
skills/x-ai-news/
├── SKILL.md          # 本文件
└── job_list.json     # 任务状态文件（自动创建）
```

---

## 📝 更新日志

### 2026-03-20
- 新增：分批抓取+间隔控制机制
- 新增：每批次实时推送功能
- 新增：智能去重逻辑
- 新增：Heartbeat 自动续传机制
- 优化：反爬策略（页面停留、批次间隔）
