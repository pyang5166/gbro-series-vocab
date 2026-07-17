# gbro-series-vocab

追剧学英语：报一个剧名 + 集数，agent 自动找到该集字幕，提取 50 个值得学的词汇短语（短语动词、习语、俚语、B2-C1 实用词），生成可以直接导入 Anki 的闪卡文件 + 一份 Markdown 词汇笔记。先花 10 分钟过一遍词，再看这集，生词全在语境里。

Learn English from TV series: give it a show name + episode, get 50 Anki-ready vocabulary cards extracted from that episode's actual script.

支持 Claude Code、Codex，以及任何支持自定义 skill 的 AI agent。不需要 API key。

---

## 怎么用

```
追剧学英语 Breaking Bad S02E03
```

或者英文触发：

```
series vocab The Pitt S01E02
```

Agent 会：

1. **自动找这一集的剧本/字幕**——老剧走 Springfield Springfield，新剧走 subslikescript，再兜不住就全网搜（含 Fandom Wiki），全都失败才会请你手动贴剧本
2. **提取 50 个词条**——只挑值得学的：短语动词 > 习语固定搭配 > 俚语口语 > B2-C1 实用词汇，跳过基础词；例句全部取自剧本原文，不造句
3. **生成两个文件**：
   - `anki-{剧名}-{集数}.csv` —— 自带 Anki 导入指令头，双击导入即可，牌组（`追剧英语::{剧名}`）和正反面映射自动设好
   - 一份 Markdown 词汇笔记（词/释义/IPA/原句/译句六列表格），存到你配置的笔记目录

卡片方向：**正面英文原句，背面母语译句 + 释义 + 音标**——考察的是"看英文能不能懂"，符合看剧场景。

---

## 首次使用

第一次触发时 skill 会问你两个问题并记住（写入 skill 目录的 `config.md`）：

1. **词汇笔记存哪**：默认 `~/Documents/series-vocab/`，Obsidian 用户可以直接填 vault 内路径
2. **释义用什么语言**：默认中文，其他语言也可以

---

## 安装

```bash
git clone https://github.com/pyang5166/gbro-series-vocab.git ~/.claude/skills/gbro-series-vocab
```

其他 agent 平台：把本仓库放进对应的 skills 目录即可。

依赖：只需要 `curl`（macOS/Linux 自带）。字幕抓取经 [r.jina.ai](https://jina.ai/reader/) 免费通道，无需注册。

---

## 已验证剧集

| 剧 | 来源 |
|---|---|
| Friends、Breaking Bad、Grey's Anatomy、It's Always Sunny in Philadelphia | Springfield Springfield |
| Severance、The Pitt、The White Lotus、9-1-1 | subslikescript |

剧名带撇号（Grey's Anatomy）、带数字（9-1-1）、2025 新剧（The Pitt）均可正常处理。

---

## 版权说明

字幕/剧本内容来自公开网页，仅供个人学习使用。skill 产出的文件只包含选出的例句，不保存整集剧本。

---

## License

MIT © 狗哥笔记
