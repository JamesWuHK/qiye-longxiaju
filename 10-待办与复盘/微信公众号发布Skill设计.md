# 微信公众号发布 Skill 设计

## 目的

- 把 `企业龙虾局` 知识库里的文章或稿件，自动发布到微信公众号草稿箱
- 目标不是群发，而是 `draft/add` 草稿保存
- 设计参考：`tentacle-pro/skills` 的 `baoyu-post-to-wechat` 与 `tentacle-post2wechat`

## Skill 名称

`skill.wechat-draft-publish`

## 适用场景

- 用户已经有一篇公众号文章或 Markdown 稿件
- 需要把文章上传到微信公众号草稿箱
- 文章包含封面图和正文图片
- 希望统一从 Obsidian 内容资产库直接进入公众号发布链路

## 总流程图

```text
输入文章路径
  -> Step 1 读取文章
  -> Step 2 清洗为公众号兼容稿
  -> Step 3 上传正文图片
  -> Step 4 上传封面图
  -> Step 5 调用 draft/add
  -> Step 6 返回草稿结果
```

## 输入

```json
{
  "source_path": "04-长文稿/某篇文章.md",
  "title": "文章标题",
  "author": "作者名",
  "summary": "摘要，不超过120字",
  "cover_image": "Assets/Cover-Images/topic/cover.png",
  "need_open_comment": 1,
  "only_fans_can_comment": 0
}
```

## 输出

```json
{
  "draft_id": "optional",
  "status": "success|failed",
  "title": "文章标题",
  "uploaded_cover_media_id": "thumb_media_id",
  "uploaded_inline_images": ["..."],
  "message": "结果说明"
}
```

## 凭证设计

### 推荐方式

- 使用本地环境变量，不写入仓库

```dotenv
WECHAT_APP_ID=...
WECHAT_APP_SECRET=...
```

### 读取顺序

1. 进程环境变量
2. 本地私有 `.env`
3. 不允许从知识库 markdown 或 git 跟踪文件读取密钥

## Step 1 读取文章

### 目标

- 读取 Markdown 或 HTML 原稿

### 输入

- `source_path`

### 动作

1. 判断输入是 `.md` 还是 `.html`
2. 读取正文
3. 如果是 Markdown，转到 Step 2 清洗链

### 规则

- 文章来源必须是知识库内文件
- 不直接发布未整理的临时聊天记录

## Step 2 清洗为公众号兼容稿

### 目标

- 把 Obsidian 稿件转换成公众号兼容 HTML

### 动作

1. 清理 Obsidian 专用语法
2. 处理 `![[...]]` 图片嵌入
3. 把内部 wikilink 去掉或转普通文本
4. 转成公众号适合的 HTML

### 规则

- 不保留 `![[...]]`
- 不保留 `[[...]]`
- 标题层级不宜过深
- 段落长度适中，适合公众号阅读

### 输出

- 干净 HTML 正文

## Step 3 上传正文图片

### 目标

- 将正文中的本地图片上传到公众号，并替换 HTML 中的图片链接

### 动作

1. 从 HTML 中提取图片路径
2. 上传每张图到 `media/uploadimg`
3. 用返回的 URL 替换 `<img src>`

### 规则

- 正文图和封面图分开处理
- 所有正文图片必须先上传再写入内容

### 输出

- 替换后的 HTML
- 已上传图片 URL 列表

## Step 4 上传封面图

### 目标

- 获取 `thumb_media_id`

### 动作

1. 读取 `cover_image`
2. 调 `material/add_material?type=image`
3. 获取返回的 `media_id`

### 规则

- 封面图必须是本地文件
- 如果未显式提供封面图，可后续定义 fallback 规则，但首版建议强制提供

## Step 5 调用 draft/add

### 目标

- 把整理好的文章保存进公众号草稿箱

### 动作

1. 组装 `articles` payload
2. 填入：
   - `title`
   - `author`
   - `digest`
   - `content`
   - `thumb_media_id`
   - `need_open_comment`
   - `only_fans_can_comment`
3. 调用 `draft/add`

### 规则

- 当前 skill 只到草稿箱，不做群发
- 默认开启评论，默认不限粉丝评论

## Step 6 返回草稿结果

### 输出

- 草稿创建成功/失败结果
- 返回草稿相关信息
- 返回上传过的封面和正文图片结果

## API 映射

- 正文图片：`media/uploadimg`
- 封面图：`material/add_material?type=image`
- 草稿保存：`draft/add`

## 推荐与现有知识库的衔接方式

- 源稿建议来自：`04-长文稿/`
- 封面建议来自：`Assets/Cover-Images/`
- 发布结果可记录到：`10-待办与复盘/`

## 完成标准

- 本地 Markdown/HTML 可被成功转换并发布到公众号草稿箱
- 封面和正文图片全部正常上传
- 不把公众号密钥写入仓库或 markdown 文档

## 后续可拆分子 Skill

- `skill.wechat-article-wash`
- `skill.wechat-inline-image-upload`
- `skill.wechat-cover-upload`
- `skill.wechat-draft-create`
