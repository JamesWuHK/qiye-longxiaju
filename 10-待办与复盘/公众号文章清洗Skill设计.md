# 公众号文章清洗 Skill 设计

## 目的

- 把 Obsidian 里的内容资产，清洗成适合微信公众号发布的稿件
- 这不是发布 skill，而是发布前的内容清洗层
- 设计灵感来自 `tentacle-skills-washing`，但对象从“skill”改成“文章资产”

## Skill 名称

`skill.wechat-article-wash`

## 适用场景

- 原稿来自 Obsidian
- 文档中含有 wiki 链接、Obsidian 图片语法、内部导航内容
- 需要转成公众号可发布的纯内容版本

## 总流程图

```text
读取 Obsidian 原稿
  -> Step 1 内容盘点
  -> Step 2 删除知识库噪音
  -> Step 3 清洗语法
  -> Step 4 调整公众号阅读结构
  -> Step 5 导出发布版
```

## 输入

```json
{
  "source_path": "04-长文稿/某篇文章.md",
  "mode": "wechat",
  "keep_images": true,
  "remove_internal_links": true
}
```

## 输出

```json
{
  "clean_markdown": "...",
  "embedded_images": ["..."],
  "removed_blocks": ["..."],
  "status": "cleaned"
}
```

## Step 1 内容盘点

### 目标

- 识别哪些内容属于“正文”，哪些内容属于“知识库噪音”

### 需要识别的噪音

- `README` 风格入口说明
- `导航` 内容
- `归档说明`
- `内部链接` 说明
- `Skill 设计` 里的结构化元信息

## Step 2 删除知识库噪音

### 动作

删除或跳过这些内容：

- 路径说明
- 目录规则
- “这份文档怎么用”类型的操作说明
- 只服务于知识库管理、不服务于读者的块

## Step 3 清洗语法

### 动作

1. 处理 `[[...]]`
2. 处理 `![[...]]`
3. 处理仅知识库内部使用的代码块
4. 去掉 markdown 中不适合公众号的技术性标记

### 规则

- 图片保留，但要转成可上传资源
- 内部链接改成纯文本或直接删除

## Step 4 调整公众号阅读结构

### 动作

1. 压缩过长小标题
2. 调整段落长度
3. 必要时补一个引子
4. 必要时补一个结尾引导

### 规则

- 公众号阅读优先级：易读 > 完整
- 不保留知识库操作语言
- 不保留“你可以点击这个文档”之类 Obsidian 内语句

## Step 5 导出发布版

### 输出形式

- 清洗后的 Markdown
- 或进一步转成 HTML 的中间稿

### 落库位置建议

- `04-长文稿/发布版/`

## 完成标准

- 读者看不出这是从 Obsidian 导出来的
- 文档中没有 Obsidian 特有语法残留
- 内容结构适合公众号阅读

## 与发布 skill 的关系

- `skill.wechat-article-wash` 先执行
- `skill.wechat-draft-publish` 后执行

## 推荐调用链

```text
skill.wechat-article-wash
-> skill.wechat-draft-publish
```
