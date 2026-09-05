# skill-open-source-publish

**把任务成果 / 事故复盘做成可开源的 agent skill 并发布 GitHub + Gitee 双平台——脱敏、双语、建仓、推送、验证全流程。**
**Turn a task outcome or incident postmortem into an open-source agent skill and publish to GitHub + Gitee — desensitization, bilingual docs, repo creation, token-safe pushing, and cross-platform verification.**

[中文](#中文) | [English](#english)

---

## 中文

### 这是什么

一套"经验资产化"的完整流水线，来自 14+ 个双平台开源仓库的实战沉淀：

- **脱敏六类**：密码 / token / 站点名 / 邮箱手机号 / 实名 / IP 全量 grep + **阳性对照**（搜一个必然存在的词）防假阴性
- **双语规范**：README 中文在前 + 锚点导航；frontmatter 纯英文（触发器不被混语言稀释）
- **双平台发布**：建仓布尔走 JSON body、PATCH 带 name、令牌一次性 URL 不落盘、公开性金标准 = 匿名 HTTP 200
- **Release 附件管理**：附件 id 走 attach_files API（详情接口不含）、multipart 上传、大文件后台轮询防网关 504
- **活文档维护**：双副本分叉合流、三层导航索引（症状 / 场景 / 主题）、多库路由
- **元模式**：跨平台做同一件事，先假设两边语义不一致

### 安装

```bash
git clone https://github.com/mowenQWQ/skill-open-source-publish.git
cp -r skill-open-source-publish /path/to/your/agent/skills/
```

---

## English

### What is this

A complete "experience-to-asset" pipeline, battle-tested across 14+ dual-platform open-source repos:

- **Six-class desensitization**: passwords / tokens / site names / emails & phones / real names / IPs — full grep plus a **positive control** (search a term that must exist) to prevent false negatives
- **Bilingual standards**: README with Chinese first + anchor navigation; English-only frontmatter (so triggers aren't diluted by mixed languages)
- **Dual-platform publishing**: booleans via JSON body, PATCH requires name, one-shot token URLs never persisted to disk, and the gold standard of public visibility = anonymous HTTP 200
- **Release asset management**: attachment IDs via the attach_files API (detail endpoints omit them), multipart uploads, backgrounded transfers with flag polling to survive gateway timeouts
- **Live-document maintenance**: dual-copy fork merging, three-layer navigation indexes (symptom / scenario / topic), multi-repo routing
- **Meta-pattern**: when doing the same thing across platforms, assume the two sides disagree on semantics first

### Install

```bash
git clone https://github.com/mowenQWQ/skill-open-source-publish.git
cp -r skill-open-source-publish /path/to/your/agent/skills/
```

---

## License

MIT-0（MIT No Attribution）— 详见 [LICENSE](LICENSE)。任何人可自由使用、修改、再分发，无需署名。

MIT No Attribution — see [LICENSE](LICENSE). Free to use, modify, and redistribute with no attribution required.

---

## 🤖 AI 使用声明 / AI Usage Disclosure

本项目在开发与维护过程中使用了 AI 编程助手（Claude / Anthropic）辅助代码编写、文档整理与问题排查；核心决策、内容审核与最终发布由维护者完成。

This project was developed and maintained with the assistance of an AI coding assistant (Claude / Anthropic) for coding, documentation, and troubleshooting. Core decisions, content review, and final releases are made by the maintainer.