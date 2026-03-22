# 短视频口播稿配图子 Skill 设计

## 目的

- 把 `skill.short-video-illustrator` 再拆成更细的 4 个子 skill
- 让 agent 可以按模块执行，而不是每次整套跑完
- 方便后续逐步自动化

## 子 Skill 总览

```text
skill.short-video-shot-split
skill.short-video-fact-image-search
skill.short-video-prompt-generate
skill.short-video-image-generate
skill.short-video-embed-assets
```

---

## 1. skill.short-video-shot-split

### 作用

- 把短视频口播稿切成镜头级结构，并为后续配图做准备

### 适用场景

- 已经有一篇短视频口播稿
- 需要先切出镜头，再决定配图方式

### 输入

```json
{
  "script_path": "03-口播稿/某篇稿子.md",
  "script_text": "口播正文",
  "prefer_human_shots": true
}
```

### 输出

```json
{
  "shots": [
    {
      "shot_id": 1,
      "duration": "0-3秒",
      "voiceover": "...",
      "intent": "镜头想表达什么",
      "screen_text": "屏幕字幕",
      "is_human_shot": true,
      "image_mode": "none|fact|generate"
    }
  ]
}
```

### 执行步骤

1. 读取口播稿
2. 按自然停顿切镜头
3. 给每个镜头写一句 `intent`
4. 判断是否适合真人镜头
5. 预判 `image_mode`

### 执行规则

- 一个镜头只保留一个主意图
- 真人判断段优先标成 `is_human_shot = true`
- 不要在这一步写配图来源

### 分类规则

- 明显是你本人下判断、给观点的段落：`is_human_shot = true`
- 涉及新闻、产品界面、真实功能展示：`image_mode = fact`
- 涉及抽象概念、对比、趋势判断：`image_mode = generate`

### 失败条件

- 无法把一段口播压缩成一个单一画面意图

### 落库动作

- 不直接写文件，返回结构化镜头结果

### 完成标准

- 所有镜头都已获得 `intent`
- 所有镜头都有初步 `image_mode`

---

## 2. skill.short-video-fact-image-search

### 作用

- 为 `fact` 类型镜头找到真实可用的事实图，并下载到本地

### 适用场景

- 口播稿里出现新闻、界面、官方功能、产品接入这类事实镜头

### 输入

```json
{
  "topic": "微信接入龙虾",
  "asset_dir": "03-口播稿/Assets/wechat-clawbot/",
  "shots": [
    {
      "shot_id": 2,
      "intent": "微信正式上线ClawBot插件",
      "image_mode": "fact"
    }
  ]
}
```

### 输出

```json
{
  "fact_assets": [
    {
      "shot_id": 2,
      "local_paths": [
        "03-口播稿/Assets/wechat-clawbot/news-plugin-1.jpg"
      ],
      "status": "downloaded|not_found"
    }
  ]
}
```

### 执行步骤

1. 根据镜头 `intent` 生成搜索关键词
2. 优先搜：官方页面、主流媒体、云厂商教程、实测文章
3. 挑选最相关的 1-2 张图
4. 下载到本地 `asset_dir`
5. 命名为可读文件名

### 执行规则

- 每个镜头最多下载 2 张主图
- 下载后的文件名必须可读
- 最终结果必须是本地文件，不是网页链接

### 搜图关键词模板

- `品牌/产品 + 功能 + 截图`
- `事件 + 正式上线 + 插件`
- `平台 + 接入 + 界面`

### 可信源优先级

1. 官方页面
2. 主流媒体
3. 腾讯云 / 阿里云 / 百度云等文档
4. 高质量实测内容

### 禁止项

- 不把搜索结果页截图当主图
- 不保留裸 URL 作为最终交付
- 不用明显无关图凑数

### 落库动作

- 事实图必须本地化保存到 `asset_dir`

### 失败条件

- 搜不到可信来源图片
- 搜到的图片与镜头意图不匹配

### 完成标准

- 所有 `fact` 镜头都有 `downloaded` 或 `not_found` 结论

---

## 3. skill.short-video-prompt-generate

### 作用

- 为 `generate` 类型镜头生成可直接生图的 prompt

### 适用场景

- 镜头是抽象概念、老板状态、对比关系、信息图或趋势解释

### 输入

```json
{
  "topic": "微信接入龙虾",
  "shots": [
    {
      "shot_id": 12,
      "intent": "大公司和小老板的对比",
      "image_mode": "generate"
    }
  ],
  "aspect_ratio": "16:9",
  "style": "realistic-business"
}
```

### 输出

```json
{
  "generation_prompts": [
    {
      "shot_id": 12,
      "prompt": "...",
      "aspect_ratio": "16:9",
      "recommended_type": "scene|comparison|infographic"
    }
  ]
}
```

### 执行步骤

1. 读取镜头意图
2. 判断最适合的画面类型：
   - `scene`
   - `comparison`
   - `infographic`
3. 把抽象概念翻译成可见场景
4. 输出中文 prompt

### 执行规则

- prompt 必须能直接转成画面
- 默认比例优先 `16:9`
- 不能只写风格，不写主体和关系

### prompt 模板

```text
主体 + 场景 + 对比关系 + 视觉风格 + 构图方式 + 比例
```

### 示例

```text
左右分屏对比图，左侧是大型企业办公室团队协作，右侧是中国小老板一个人在办公室同时处理电脑、手机、表格和客户消息，商业纪实摄影风格，16:9
```

### 规则

- 不写空泛口号
- 不要只写“科技感”“高级感”这类模糊词
- 必须能让人想象出画面

### 失败条件

- prompt 仍然停留在抽象判断，无法对应具体画面

### 落库动作

- 默认写回当前稿件的对应镜头下方
- 如果后续接生图工具，再落到本地图片目录

### 完成标准

- 所有 `generate` 镜头都至少有 1 条可执行 prompt

---

## 4. skill.short-video-image-generate

### 作用

- 调用具体的图像生成能力，把 `generate` 类型镜头的 prompt 变成本地图片文件

### 适用场景

- 已经有可执行 prompt
- 需要真正生成概念图，而不是只保留 prompt 文本

### 输入

```json
{
  "asset_dir": "03-口播稿/Assets/wechat-clawbot/",
  "generation_prompts": [
    {
      "shot_id": 12,
      "prompt": "左右分屏对比图...",
      "aspect_ratio": "16:9",
      "recommended_type": "comparison",
      "output_name": "boss-vs-enterprise.png"
    }
  ],
  "provider": "dashscope|openai|google|replicate",
  "model": "optional"
}
```

### 输出

```json
{
  "generated_assets": [
    {
      "shot_id": 12,
      "local_paths": [
        "03-口播稿/Assets/wechat-clawbot/boss-vs-enterprise.png"
      ],
      "provider": "dashscope",
      "model": "z-image-turbo",
      "status": "generated|failed"
    }
  ]
}
```

### 执行步骤

1. 读取 `generation_prompts`
2. 确定 provider：
   - 优先显式传入
   - 否则按环境变量可用性选择
3. 确定 model：
   - CLI 参数优先
   - 其次环境变量
   - 最后默认模型
4. 将 prompt 保存为 prompt file
5. 调用生成脚本执行
6. 输出图片到 `asset_dir`

### 执行规则

- 生成前必须明确显示：`Using [provider] / [model]`
- prompt 不允许只存在内存里，必须先落成文件
- 输出文件必须用可读文件名
- 默认使用 `16:9`

### 建议调用方式

参考 `baoyu-image-gen` 的模式，统一按这种执行：

```bash
bun .agents/skills/baoyu-image-gen/scripts/main.ts --promptfiles <prompt-file> --image <output-image> --provider <provider> --ar <aspect>
```

### 已验证的 DashScope 执行方式

- 接口：`/api/v1/services/aigc/text2image/image-synthesis`
- 模型：`qwen-image-plus`
- 模式：异步任务
- 流程：
  1. 提交生成任务
  2. 获取 `task_id`
  3. 轮询 `/api/v1/tasks/{task_id}`
  4. 状态 `SUCCEEDED` 后下载 `results[0].url`

### 已验证说明

- 该流程已在当前项目的概念镜头生成中实际跑通
- 已成功生成并落盘 5 张图片

### 环境要求

- `OPENAI_API_KEY`
- `GOOGLE_API_KEY`
- `DASHSCOPE_API_KEY`
- `REPLICATE_API_TOKEN`

优先级：

- CLI 参数
- 环境变量
- `.agents/skills/.env`

### 失败条件

- 对应 provider 的 key 不存在
- prompt file 未落盘
- 生成失败且重试后仍失败

### 落库动作

- 图片输出到 `asset_dir`
- prompt file 同时保存到 `asset_dir/prompts/`

### 完成标准

- 每个 `generate` 镜头都返回 `generated` 或 `failed` 结果
- 成功图片已在本地目录落盘

---

## 5. skill.short-video-embed-assets

### 作用

- 把事实图和生成图统一嵌入到 Obsidian 文档对应镜头位置

### 适用场景

- 已经拿到本地事实图或生成图
- 需要把图片正式变成 Obsidian 图文混排稿件

### 输入

```json
{
  "script_path": "03-口播稿/某篇稿子.md",
  "fact_assets": [
    {
      "shot_id": 2,
      "local_paths": ["03-口播稿/Assets/topic/news-plugin-1.jpg"]
    }
  ],
  "generated_assets": [
    {
      "shot_id": 12,
      "local_paths": ["03-口播稿/Assets/topic/boss-vs-enterprise.png"]
    }
  ]
}
```

### 输出

```json
{
  "updated_script_path": "03-口播稿/某篇稿子.md",
  "embedded_shots": [2, 12],
  "status": "done|partial"
}
```

### 执行步骤

1. 找到对应镜头块
2. 在镜头块内插入 `- 插图：`
3. 使用 Obsidian 语法嵌入：

```text
![[03-口播稿/Assets/topic/image.png]]
```

4. 如果没有实际生成图，则保留 `生成提示词`
5. 清理旧的 URL 说明，保留最终可用结果

### 执行规则

- 优先保留本地图片嵌入，不保留 URL 清单
- 每个镜头最多插 2 张主图
- 真人镜头默认不插图
- 文档最终应以本地图片嵌入为准，不以外链为准

### 完成标准

- 在 Obsidian 中打开文档即可看到图文混排
- 不需要再人工自己补链接

### 失败条件

- 图片未本地化
- 插图位置错位
- 最终文档仍以 URL 列表为主

---

## 推荐编排方式

### 最小可用链路

```text
skill.short-video-shot-split
-> skill.short-video-fact-image-search
-> skill.short-video-embed-assets
```

### 完整混合链路

```text
skill.short-video-shot-split
-> skill.short-video-fact-image-search
-> skill.short-video-prompt-generate
-> skill.short-video-image-generate
-> skill.short-video-embed-assets
```

## 当前优先实现建议

最值得优先稳定的顺序：

1. `skill.short-video-shot-split`
2. `skill.short-video-fact-image-search`
3. `skill.short-video-embed-assets`
4. `skill.short-video-prompt-generate`
5. `skill.short-video-image-generate`

原因：

- 先把事实图配图链跑通，内容马上就能用
- 生图是闭环增强项，但依赖 provider key
