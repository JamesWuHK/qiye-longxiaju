# 短视频口播稿配图 Skill

## 目的

- 把“分析口播稿 -> 判断哪些镜头该搜图、哪些该生图 -> 下载事实图 -> 嵌入文档 -> 保留生成位”这套过程，固化成一个可重复执行的 skill
- 这个 skill 面向的是 `短视频口播稿`，不是长文，不是封面图
- 目标输出是：`带图文混排的 Obsidian 口播稿文档`

## 适用场景

- 用户已经有一篇短视频口播稿
- 希望给口播稿按镜头配图
- 有些镜头需要事实图，有些镜头需要概念图
- 最终要落在 Obsidian 里，支持 `![[...]]` 图文混排

## Skill 来源

这个 skill 结合了两类能力：

- `事实图配图`：来自这次我们对微信 ClawBot / OpenClaw 新闻的搜图、下载、本地化、插入过程
- `生图配图`：参考 `tentacle-pro/skills` 里的相关设计思路
  - `baoyu-article-illustrator`
  - `baoyu-image-gen`
  - `baoyu-cover-image`

但本 skill 面向的是 `短视频镜头`，不是文章段落或封面。

## Skill 名称

`skill.short-video-illustrator`

## 总流程图

```text
输入口播稿
  -> Step 1 镜头切分
  -> Step 2 镜头分类
  -> Step 3 事实图搜集
  -> Step 4 概念图生成规划
  -> Step 5 本地素材落盘
  -> Step 6 嵌入口播稿
  -> Step 7 检查与归档
```

## 输入

```json
{
  "script_path": "03-口播稿/某篇稿子.md",
  "topic": "微信接入龙虾",
  "mode": "hybrid",
  "asset_dir": "03-口播稿/Assets/topic-slug/",
  "need_real_images": true,
  "need_generated_images": true
}
```

## 输出

```json
{
  "updated_script_path": "03-口播稿/某篇稿子.md",
  "asset_dir": "03-口播稿/Assets/topic-slug/",
  "downloaded_images": ["..."],
  "embedded_images": ["..."],
  "generation_prompts": ["..."],
  "unresolved_shots": ["..."]
}
```

## Step 1 镜头切分

### 目标

- 把口播稿切成可配图的镜头单位

### 输入

- 口播稿正文

### 动作

1. 按口播自然停顿切成镜头段落
2. 每个镜头保留 5 个字段：
   - 时长
   - 口播内容
   - 画面意图
   - 屏幕字幕
   - 配图方式

### 输出格式

```text
镜头 7
- 时长：32-38 秒
- 口播：...
- 画面意图：微信里直接调 AI 干活
- 屏幕字幕：在微信里 直接调AI干活
- 配图方式：待判断
```

### 规则

- 真人出镜镜头不强制插图
- 每个镜头只允许一个主画面意图

## Step 2 镜头分类

### 目标

- 判断每个镜头是 `搜图`、`生图`、还是 `真人不配图`

### 分类规则

#### A. 必须搜图

满足任一条件即归为搜图：

- 涉及新闻事实
- 涉及平台界面
- 涉及产品功能展示
- 涉及“正式上线”“官方插件”“接入界面”等真实证据

#### B. 建议生图

满足任一条件即归为生图：

- 抽象概念解释
- 对比关系
- 老板经营状态
- 岗位变化
- 门槛变化
- “少招一个人”“数字员工”这类结果型概念

#### C. 真人镜头

- 你本人出镜表达判断
- 不必额外插图

### 输出格式

```json
{
  "shot_id": 12,
  "class": "generate",
  "reason": "这是大公司与小老板的概念对比，不适合搜事实图"
}
```

## Step 3 事实图搜集

### 目标

- 为 `搜图类镜头` 找到足够靠谱的事实图

### 动作

1. 先搜可信来源：
   - 官方页面
   - 主流媒体
   - 腾讯云/阿里云/百度云等文档
   - 高质量教程页
2. 过滤掉无关图和旧图
3. 优先下载这些图：
   - 新闻截图
   - 插件界面图
   - 聊天界面图
   - 官方接入流程图
4. 下载到本地素材目录

### 事实图来源优先级

1. 官方截图
2. 主流媒体正文配图
3. 云厂商教程图
4. 高质量实测文章图

### 禁止项

- 不要直接把 URL 留在正文里当最终结果
- 不要用明显无关的老截图充数
- 不要把搜索结果页截图当最终图，除非是为了展示“媒体热议”

### 输出格式

```json
{
  "shot_id": 7,
  "downloads": [
    "03-口播稿/Assets/topic-slug/wechat-service-chat.jpg"
  ]
}
```

## Step 4 概念图生成规划

### 目标

- 为 `生图类镜头` 生成可执行的 prompt

### 动作

1. 明确画面关系：人物 / 对比 / 信息图 / 场景
2. 把抽象概念翻译成视觉场景
3. 生成一条可执行 prompt
4. 标注建议比例：通常 `16:9`

### prompt 结构

```text
主体 + 场景 + 关系 + 风格 + 构图 + 比例
```

### 示例

```text
左右分屏对比图，左侧是大型企业办公室团队协作，右侧是中国小老板一个人在办公室同时处理电脑、手机、表格和客户消息，商业纪实摄影风格，16:9
```

### 规则

- 不要写抽象口号型 prompt
- 必须让画面能被直接想象出来
- 中文内容优先用中文 prompt

## Step 5 本地素材落盘

### 目标

- 所有最终使用的图片都必须落到本地目录

### 动作

1. 创建目录：`03-口播稿/Assets/{topic-slug}/`
2. 所有事实图下载到这里
3. 所有信息卡、SVG、自制图也存这里
4. 若后续生图成功，也统一存这里

### 命名规则

- `news-plugin-1.jpg`
- `wechat-service-chat.jpg`
- `constraint-card.svg`
- `small-boss-vs-enterprise.png`

### 规则

- 文件名必须可读
- 不允许随机 hash 名作为最终文件名

## Step 6 嵌入口播稿

### 目标

- 把本地图像直接插入到 Obsidian 文档的对应镜头位置

### 动作

1. 找到对应镜头块
2. 在 `- 插图：` 下插入本地图片
3. 使用 Obsidian 嵌入语法：

```text
![[03-口播稿/Assets/topic-slug/image-name.png]]
```

4. 每个镜头插 1-2 张即可，避免过多

### 规则

- 真人镜头不强制插图
- 一个镜头最多插 2 张图，除非明确是快切镜头
- 不要在最终稿里保留图片 URL 清单

## Step 7 检查与归档

### 目标

- 确保文档是可直接在 Obsidian 中查看和继续编辑的

### 检查项

1. 图片是否已经本地化
2. 文档里是否使用 `![[...]]` 而不是裸 URL
3. 事实图镜头是否已尽量补齐
4. 生图镜头是否至少保留 prompt
5. 是否还有“配图建议”但没有实际落地的占位内容

### 输出结论

```json
{
  "status": "ready|partial|needs-generation",
  "facts_done": true,
  "generation_pending": [12, 13, 14, 16]
}
```

## 推荐调用关系

### 如果只是补事实图

```text
Step 1 -> Step 2 -> Step 3 -> Step 5 -> Step 6 -> Step 7
```

### 如果是完整混合配图

```text
Step 1 -> Step 2 -> Step 3 -> Step 4 -> Step 5 -> Step 6 -> Step 7
```

## 与 tentacle-pro skills 的关系

### 借鉴自 `baoyu-article-illustrator`

- 先分析内容结构，再决定哪里需要图
- 不是见段落就配图，而是按信息价值配图

### 借鉴自 `baoyu-image-gen`

- 对于概念镜头，使用 prompt 驱动生成
- 统一使用本地落盘后的图片结果

### 不直接照搬的地方

- 本 skill 不是文章段落配图，而是短视频镜头配图
- 优先级不是“美观”，而是“事实可信 + 镜头可发”

## 当前适合企业龙虾局的配图原则

- 新闻事实镜头：优先搜图
- 产品界面镜头：优先搜图
- 老板经营状态：优先生图或自己拍 B-roll
- 对比/趋势/门槛：优先生成信息图或概念图

## 最终产物标准

完成后，一篇合格的短视频口播稿文档应包含：

- 标题
- 一键复制口播稿
- 文案区首句
- 评论区引导
- 逐镜头配图执行版
- 已嵌入的本地事实图
- 待生成镜头的 prompt

## 当前可继续拆分的子 skill

- `skill.short-video-shot-split`
- `skill.short-video-fact-image-search`
- `skill.short-video-prompt-generate`
- `skill.short-video-embed-assets`

其中最值得先实现的是：

- `skill.short-video-fact-image-search`
- `skill.short-video-embed-assets`

因为这两步最直接决定文档能不能从“方案”变成“可发布资产”。
