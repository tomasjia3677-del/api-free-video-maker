---
name: api-free-video-maker
description: 零API调用的低成本竖版短视频自动生成流水线。不依赖Runway/Pika/任何多模态API，通过edge-tts+FFmpeg本地合成，单条成本几分钱。适用于知识科普视频、资讯汇总、产品卖点卡等批量出片场景。触发词包括"生成视频"、"不用API做视频"、"零成本视频"、"跑流水线"、"出片"、"本地合成视频"
---

# API-Free Video Maker

## 概览

不调用任何多模态视频/图片生成 API，全本地运算：

- **单条成本**：几分钱（电费 + TTS 一次）
- **生成速度**：30 秒 ~ 2 分钟出片
- **规模复制**：无边际成本，跑 1 条和跑 1000 条单价一样

## 核心架构

```
news JSON → TTS (edge-tts) → 字幕帧 (Pillow) → 视频合成 (FFmpeg) → MP4
                ↓                ↓                    ↓
          免费本地TTS       Pillow叠加文字       素材 + 字幕 +
                                           背景音乐混流
          全程不走任何 API
```

## 快速上手

```bash
python3 scripts/daily_news_video.py
```

前提：`/tmp/daily_news_items.json` 已就绪（格式见 `news_template.json`）

## 脚本说明

### `scripts/daily_news_video.py`

主 Pipeline。参数：

| 参数 | 默认值 | 说明 |
|---|---|---|
| `--input` | `/tmp/daily_news_items.json` | 输入脚本 JSON 路径 |
| `--project` | `ai_news_project` | 项目目录（相对 workspace） |

路径自动解析 `OPENCLAW_WORKSPACE` 环境变量，或从脚本位置推断。

JSON 字段说明：

| 字段 | 说明 |
|---|---|
| `date` | 日期标签（显示在视频顶部） |
| `intro` | 开场白 TTS 文本 |
| `outro` | 片尾语 TTS 文本 |
| `project` | 项目目录（可选，可用 --project 覆盖） |
| `news[].title` | 标题，12 字以内 |
| `news[].tts_text` | 该条的口播文本 |
| `news[].lines` | 字幕 3 行，每行 15 字以内 |
| `news[].clip` | 背景素材文件名（留空则自动匹配） |
| `news[].offset` | 素材剪辑起始秒数（留空则自动匹配） |

### `scripts/footage_matcher.py`

新闻文字内容 → 素材分类智能匹配器。

根据标题+要点提取关键词，按 tech / ai / abstract / nature / business 五类打分，自动选最佳分类。

默认回退：素材库随机选 → 全部无素材则用 qwen_01.mp4。

```python
from scripts import footage_matcher as fm

cat = fm.match_category("OpenAI GPT-5.5", lines)        # → "ai"
clip, offset = fm.pick_clip(cat, "/path/to/project")   # → ("ai/language_model.mp4", 5)
```

### `scripts/news_with_tiktok.py`

素材自动采集 + 匹配 Pipeline。

按新闻标题提取关键词 → 搜索 Pexels 免费竖版视频 → 下载到对应分类目录 → 自动填入 JSON。

多源路由：Pexels（免费可商用）→ 已有素材库回退。

```bash
# 在写好的 JSON 基础上跑：
python3 scripts/news_with_tiktok.py
# 更新 JSON 中的 clip/offset 字段
```

支持 `--input` 参数指定 JSON 路径。

### `scripts/news_template.json`

模板 + 示例，包含完整 JSON 结构和填充示例。

### `scripts/task_progress.py`

进度上报工具。Pipeline 每步自动调用，外部可通过 `/tmp/openclaw_task_status.json` 查询。

## 技术参数

| 项 | 值 |
|---|---|
| 分辨率 | 720 × 1280 竖版 9:16 |
| 封面 | `{project}/user_cover_ready.png` |
| 片尾 | `{project}/user_outro_ready.png` |
| 素材 | `{project}/raw_materials/*.mp4` |
| 字体 | 文泉驿微米黑 28/34px |
| TTS | edge-tts, zh-CN-XiaoxiaoNeural, 语速 +30% |
| 编码 | H.264 (libx264, CRF 22) + AAC 128k |
| 输出 | `{project}/ai_news_latest.mp4` |

## 依赖

- Python 3.8+
- edge-tts + Pillow
- FFmpeg + ffprobe
- 字体：wqy-microhei
