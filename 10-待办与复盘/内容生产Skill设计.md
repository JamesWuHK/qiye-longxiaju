# 内容生产 Skill 设计

## 目的

- 把 `内容生产SOP` 拆成可单独执行的 skill 模块
- 每个模块都能被 agent 直接调用
- 每个模块必须清楚定义：输入、输出、规则、失败条件、落库动作

## Skill 总览

```text
skill.topic-capture
skill.topic-score
skill.angle-select
skill.title-generate
skill.script-build
skill.visual-plan
skill.publish-pack
skill.post-review
skill.asset-maintain
```

## 1. skill.topic-capture

### 作用

- 把新闻、灵感、评论区问题转成可评估的候选选题

### 输入

```json
{
  "source_type": "news|competitor|comment|observation",
  "raw_input": "原始信息",
  "date": "YYYY-MM-DD",
  "priority": "high|medium|low"
}
```

### 输出

```json
{
  "event": "发生了什么",
  "boss_relevance": "老板为什么要关心",
  "result_angle": "可能的结果感",
  "suggested_topic": "候选选题一句话"
}
```

### 执行规则

- 不做长篇分析
- 强制压缩成 4 个字段
- `boss_relevance` 必须是老板视角，不是技术视角

### 失败条件

- 无法用一句话说清“老板为什么要关心”

### 落库动作

- 写入 `09-素材摘录/`

## 2. skill.topic-score

### 作用

- 判断选题值不值得进入生产流程

### 输入

```json
{
  "event": "...",
  "boss_relevance": "...",
  "result_angle": "...",
  "suggested_topic": "..."
}
```

### 输出

```json
{
  "score": 0,
  "reason": ["..."],
  "decision": "produce|observe|drop"
}
```

### 评分规则

- 和小企业老板直接相关：1 分
- 有明确结果感：1 分
- 能 3 秒讲清冲突：1 分
- 能自然接下一条：1 分
- 符合账号定位：1 分

### 决策规则

- `>=4`：`produce`
- `=3`：`observe`
- `<=2`：`drop`

### 落库动作

- `produce` 或 `observe` 写入 `02-选题库/`

## 3. skill.angle-select

### 作用

- 给选题确定唯一主角度

### 输入

```json
{
  "topic": "...",
  "score": 0,
  "boss_relevance": "...",
  "result_angle": "..."
}
```

### 输出

```json
{
  "angle": "hot-translation|boss-result|risk-warning|opportunity|action-scene",
  "core_judgement": "一句核心判断",
  "avoid": ["不该讲的内容"]
}
```

### 执行规则

- 一条内容只能有一个主角度
- `avoid` 至少写 2 条

## 4. skill.title-generate

### 作用

- 根据选题和角度生成标题

### 输入

```json
{
  "topic": "...",
  "angle": "...",
  "core_judgement": "..."
}
```

### 输出

```json
{
  "main_title": "...",
  "alt_titles": ["...", "...", "..."],
  "cover_text": "..."
}
```

### 执行规则

- 主标题优先不超过 20 字
- 标题必须有老板视角或结果感
- 禁止使用咨询黑话

### 失败条件

- 标题读起来像报告标题

## 5. skill.script-build

### 作用

- 生成可直接录制的口播成稿

### 输入

```json
{
  "main_title": "...",
  "angle": "...",
  "core_judgement": "...",
  "facts": ["..."]
}
```

### 输出

```json
{
  "full_script": "60秒版口播稿",
  "short_script": "20秒版口播稿",
  "first_sentence": "文案区首句",
  "comment_hook": "评论区引导语"
}
```

### 固定结构

1. 先抛新闻或现象
2. 翻译成老板该关心什么
3. 解释为什么是变化
4. 讲 2-3 个现实结果
5. 一句强结论收尾

### 执行规则

- 不能先定义技术概念
- 每句话尽量只表达一个意思
- 全文必须回到小老板结果

### 落库动作

- 写入 `03-口播稿/`

## 6. skill.visual-plan

### 作用

- 把口播稿拆成适合平台分发的画面脚本

### 输入

```json
{
  "full_script": "...",
  "topic_type": "hot|series|generic"
}
```

### 输出

```json
{
  "shots": [
    {
      "time_range": "0-3s",
      "visual_type": "talking-head|news-shot|ui-shot|broll",
      "visual_desc": "画面内容",
      "screen_text": "屏幕字幕"
    }
  ]
}
```

### 执行规则

- 每 3-5 秒必须切镜
- 不能全程纯口播黑底字
- 必须包含至少两类画面：新闻截图 / 界面图 / 现实场景 / 真人出镜

## 7. skill.publish-pack

### 作用

- 生成发布时所需的全部组件

### 输入

```json
{
  "main_title": "...",
  "cover_text": "...",
  "full_script": "...",
  "first_sentence": "...",
  "comment_hook": "..."
}
```

### 输出

```json
{
  "copy_block": {
    "title": "...",
    "cover": "...",
    "script": "...",
    "caption_open": "...",
    "comment_hook": "..."
  }
}
```

### 执行规则

- 输出必须可直接复制
- 必须包含一键复制块

## 8. skill.post-review

### 作用

- 对单条内容进行结构化复盘

### 输入

```json
{
  "title": "...",
  "comments_summary": "...",
  "performance_notes": "..."
}
```

### 输出

```json
{
  "title_result": "strong|medium|weak",
  "best_line": "...",
  "best_visual": "...",
  "user_focus": "opportunity|risk|how-to",
  "next_topic": "..."
}
```

### 落库动作

- 写入 `10-待办与复盘/`

## 9. skill.asset-maintain

### 作用

- 保持知识库结构干净，不堆中间稿

### 输入

```json
{
  "topic": "...",
  "file_type": "draft|active|archive",
  "path": "...",
  "action": "overwrite|archive|create|rename"
}
```

### 输出

```json
{
  "final_path": "...",
  "navigation_update": true,
  "archive_update": true
}
```

### 执行规则

- 同一主题优先覆盖
- 旧稿移到 `03-口播稿/归档/`
- 新增活跃稿后，更新 `03-口播稿/目录导航.md`
- 结构变化后，更新 `知识库导航.md`

## 推荐调用链

### 热点内容生产链

```text
skill.topic-capture
-> skill.topic-score
-> skill.angle-select
-> skill.title-generate
-> skill.script-build
-> skill.visual-plan
-> skill.publish-pack
```

### 发布后维护链

```text
skill.post-review
-> skill.asset-maintain
```

## 当前最适合先实现的 3 个 skill

- `skill.title-generate`
- `skill.script-build`
- `skill.visual-plan`

因为这 3 个模块最直接决定能不能稳定产出可发布内容。
