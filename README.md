# API-Free Video Maker

> 零 API 调用的低成本竖版短视频自动生成流水线。不依赖 Runway / Pika / 任何多模态生成 API，单条成本几分钱。

## 为什么不用 API？

传统方案：调用多模态 API 生成画面 → 每次 $1~5 → 1000 条视频就是几千美金。

本方案：TTS + FFmpeg 本地合成 → 每条几分钱（电费）。

让做视频从"烧 API 成本"变成"几乎零成本跑流水线"。

## 效果对比

| 维度 | 传统 API 方案 | 本方案 |
|------|-------------|--------|
| 单条成本 | $1 ~ $5 | 几分钱 |
| 生成速度 | 5~30 分钟 | 30 秒 ~ 2 分钟 |
| 可控性 | 黑盒生成，结果随机 | 逐帧可控，排版统一 |
| 可规模复制 | API 费用线性增长 | 几乎无边际成本 |

## 适用场景

- 资讯/知识科普每日视频
- 产品卖点卡 / 营销短视频
- 任何需要固定模板批量出片的场景

## 快速开始

### 依赖

```bash
pip install edge-tts Pillow

# 字体
# Ubuntu/Debian: apt install fonts-wqy-microhei
# macOS: brew install --cask font-wqy-microhei

# FFmpeg
# Ubuntu/Debian: apt install ffmpeg
# macOS: brew install ffmpeg
```

### 准备素材

```
your_project/
├── raw_materials/          # 背景视频素材（竖版）
│   ├── clip_01.mp4
│   └── clip_02.mp4
├── user_cover_ready.png    # 封面图 720×1280
└── user_outro_ready.png    # 片尾图 720×1280
```

### 准备素材（自动采集）

如果素材库不足，可以自动搜索下载匹配的免费竖版视频：

```bash
# 1. 写新闻脚本（clip/offset 可留空）
python3 -c "..." > /tmp/daily_news_items.json

# 2. 自动采集素材 + 匹配
python3 scripts/news_with_tiktok.py
# → 从 Pexels / Coverr 搜索并下载匹配的竖版视频
# → 自动填入 clip/offset 字段
# → 下载失败则从已有素材库随机选

# 3. 出片
python3 scripts/daily_news_video.py
```

### 手动指定素材

参考 `news_template.json`，写入 `/tmp/daily_news_items.json`。核心字段：

- `intro` / `outro` — 口播开头结尾
- `news[].tts_text` — 每条的口播文本
- `news[].lines` — 字幕三行
- `news[].clip` — 背景素材文件名（留空则自动匹配）
- `news[].offset` — 素材起始时间（留空则自动匹配）

### 跑起来

```bash
python3 scripts/daily_news_video.py
# 输出: your_project/ai_news_latest.mp4
```

自定义路径：

```bash
python3 scripts/daily_news_video.py \
  --input /tmp/my_script.json \
  --project my_project
```

### 完整自动化 cron

可用于每日定时出片（配合 GitHub Actions 或 cron job）：

```bash
# cron: 0 9 * * *
# 1. 搜新闻 → 写入 JSON
# 2. 自动采素材 → python3 scripts/news_with_tiktok.py
# 3. 出片 → python3 scripts/daily_news_video.py
# 4. 发送到微信/Telegram/消息通道
```

## 输出规格

- 分辨率：720 × 1280（竖版 9:16）
- 时长：封面 2.5s + N 条内容（每条 ~6.5s）+ 片尾 3s
- 编码：H.264 + AAC
- TTS：edge-tts zh-CN-XiaoxiaoNeural，语速 +30%

## 技术栈

- **TTS**：edge-tts（免费，本地运行）
- **渲染**：FFmpeg + Pillow
- **管道**：Python 3
- **无 API 依赖**：全程本地运算，不调用任何付费接口

## License

MIT
