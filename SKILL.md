---
name: gbro-series-vocab
description: 追剧学英语 / Learn English vocabulary from TV series. 用户给出剧名+集数（如「追剧学英语 Breaking Bad S02E03」），自动找该集字幕/剧本，提取 50 个词汇短语，生成 Anki 导入 CSV + Markdown 词汇笔记。触发词：「追剧学英语」「剧学英语」「series vocab」「learn english from」+ 剧名集数。仅处理英美剧单集，不处理 YouTube 视频或普通翻译请求。
---

# gbro-series-vocab · 追剧学英语

帮用户从一集英美剧中高效提取词汇，输出 Anki 闪卡 + Markdown 词汇笔记。

---

## 首次使用（配置）

skill 目录下没有 `config.md` 时，先问用户两个问题，答案写入 `config.md`，之后不再重复问：

1. **词汇笔记保存目录**：默认 `~/Documents/series-vocab/`；Obsidian/Logseq 用户可以填 vault 内的路径
2. **释义语言**：默认中文；其他母语用户可改（下文所有「中文释义/中文译句」按此替换为用户母语）

`config.md` 格式：

```markdown
notes_dir: ~/Documents/series-vocab/
native_language: 中文
```

---

## 输入解析

用户会说类似：`追剧学英语 Breaking Bad S02E03`

提取：
- **剧名**：英文名（如 Breaking Bad）
- **集数**：SxxExx 格式 → 解析为 season=2, episode=3

---

## 第一步：找字幕/剧本

按顺序尝试，找到即停止。

### 方法 A：Springfield Springfield（老剧首选）

slug 规则：剧名转小写，空格换连字符，**去掉所有非字母数字和连字符的字符**（撇号、冒号、逗号、句点等），连续连字符合并为一个。例如 `Grey's Anatomy` → `greys-anatomy`，`It's Always Sunny in Philadelphia` → `its-always-sunny-in-philadelphia`。

```bash
SHOW_SLUG=$(echo "Grey's Anatomy" | tr '[:upper:]' '[:lower:]' | tr ' ' '-' | sed -E 's/[^a-z0-9-]//g; s/-+/-/g')
curl -sL "https://r.jina.ai/https://www.springfieldspringfield.co.uk/view_episode_scripts.php?tv-show=${SHOW_SLUG}&episode=s02e03"
```

**判断成功（两条必须同时满足）**：
1. 页面标题含剧名（找不到该剧时网站返回的错误页标题只有集数没有剧名，且正文只剩导航链接，长度也可能上万字符——不要只看长度）
2. 正文含大量对话行（连续的口语句子），而非以 `[链接](url)` 为主的导航内容

**注意**：Springfield 已基本停更，2023 年后的新剧（如 The Pitt、Severance、White Lotus S3）都没有收录。方法 A 失败一次就果断转方法 B，不要反复试。

### 方法 B：subslikescript 站内搜索（新剧首选）

subslikescript 持续收录新剧，但直接 curl 会被 JS 验证码挡住，**每一步都必须经 r.jina.ai 抓取**。三步走：

```bash
# 1. 站内搜索，拿剧集页 URL（形如 /series/{Name}-{id}）
curl -sL "https://r.jina.ai/https://subslikescript.com/search?q={关键词}"

# 2. 抓剧集页，找目标集链接（形如 /series/{Name}-{id}/season-{s}/episode-{e}-{集标题}）
#    单集 URL 末尾带集标题，无法直接拼出，必须从剧集页拿
curl -sL "https://r.jina.ai/https://subslikescript.com/series/{Name}-{id}"

# 3. 抓单集正文
curl -sL "https://r.jina.ai/{单集URL}"
```

坑（实测）：
- 搜索是按词匹配的，全名可能搜不到：`The Pitt` 无结果，搜 `Pitt` 才命中。失败时去掉 The/标点，用剧名里最独特的一个词重搜
- 从 markdown 链接里抽出的 URL 可能带尾部空格，拼给 r.jina.ai 前先修剪，否则返回空
- 同名剧按年份区分（如 Severance 1988 vs 2022–…），选年份对的那个

### 方法 C：通用 WebSearch

`"{剧名}" "S{xx}E{xx}" transcript script site:transcripts.foreverdreaming.org OR site:subslikescript.com OR site:springfieldspringfield.co.uk`

剧集的 Fandom Wiki transcript 页也是可靠来源。

### 方法 D：直连兜底（r.jina.ai 不可用时，仅限 Springfield）

如果 r.jina.ai 超时或限流，直接 curl Springfield 原始页面并本地清洗：

```bash
# 注意起点必须匹配 class="scrolling-script-container"（带 class=）——
# 裸的 scrolling-script-container 会先命中页面内联 CSS，抓到样式表
curl -sL "https://www.springfieldspringfield.co.uk/view_episode_scripts.php?tv-show=${SHOW_SLUG}&episode=s02e03" \
  | sed -n '/class="scrolling-script-container"/,/<\/div>/p' \
  | sed -E 's/<br>/\n/g; s/<[^>]+>//g' \
  | sed '/^[[:space:]]*$/d'
```

### 找不到时

告诉用户：「未找到《{剧名}》第{season}季第{episode}集的字幕，请从网上复制剧本文字后发给我，我继续处理。」

---

## 第二步：提取词汇 — 50 个

提取标准（按优先级）：
1. **Phrasal verbs**：get away with、pull off、cut it out 这类动词短语
2. **习语和固定搭配**：字面意思和实际意思不同的表达
3. **俚语/口语**：词典查不到准确语境的词
4. **B2-C1 级实用词汇**：跳过 hello、sorry 等基础词，但真实场景高频出现

每个词条格式：
```
序号. 词/短语 | 释义（用户母语） | IPA音标 | 原句 | 原句译文（用户母语）
```

直接从剧本中找原句，不要自己造句。相同英文原句只保留一条。

---

## 第三步：生成 Anki CSV

路径：`~/Downloads/anki-{剧名slug}-{SxxExx}.csv`

文件开头必须写 Anki 导入指令头（以 `#` 开头的行），让 Anki 自动完成分隔符、字段映射和牌组，用户无需手动确认映射：

```
#separator:Comma
#html:false
#notetype:Basic
#deck:追剧英语::{剧名}
#columns:Front,Back
"英文原句","译句（释义 /IPA/）"
"英文原句","译句（释义 /IPA/）"
```

- **Front = 英文原句**（看英文考自己），**Back = 母语译句 +（释义 /IPA/）**，不要反
- 数据行用英文双引号包裹每个字段，内部引号用两个双引号转义
- 指令头之后不再加表头行

---

## 第四步：保存词汇笔记

路径：`{notes_dir}/{剧名}/YYYY-MM-DD-{剧名}-{SxxExx}.md`（`notes_dir` 读 `config.md`）

目录不存在时自动 `mkdir -p`。

```markdown
---
title: "{剧名} {SxxExx} 词汇"
date: YYYY-MM-DD
tags: [english, series-vocab]
source: {Springfield Springfield / subslikescript / 其他来源}
---

# {剧名} {SxxExx} 词汇表

共 {N} 个词条。Anki 文件：`~/Downloads/{csv文件名}`

---

## 词汇列表

| # | 词/短语 | 释义 | IPA | 英文原句 | 译句 |
|---|---------|------|-----|---------|------|
| 1 | ... | ... | ... | ... | ... |

---

## 如何使用

1. 复习这些词（10-15 分钟）
2. 看这集时注意这些词出现的场景
3. 把 CSV 导入 Anki：File → Import → 选文件 → Import（牌组和字段映射已由文件头自动设置）
```

---

## 完成回复格式

```
✅ {剧名} {SxxExx} 词汇提取完成

📚 共提取 {N} 个词条
📄 Anki CSV：~/Downloads/{csv文件名}
📝 词汇笔记：{notes_dir}/{剧名}/{filename}.md

导入 Anki：File → Import → 选 CSV → Import 即可
（牌组「追剧英语::{剧名}」和正反面映射已在文件头设好，无需手动调整）

建议：先花 10 分钟过一遍词汇，再看这集。
```

---

## 版权边界

字幕/剧本内容来自公开网页，仅供用户个人学习使用；不要把整集剧本原文写入产出文件（笔记和 CSV 里只保留选出的例句）。
