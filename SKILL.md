---
name: skill-open-source-publish
description: "Turn a task outcome or incident postmortem into an open-source agent skill and publish to GitHub + Gitee — desensitization, bilingual README (Chinese first), repo creation, token-safe pushing, cross-platform verification, release asset management, live-document merging and multi-repo routing. Use when the user says "make this a skill and open-source it", "publish to GitHub", "sync to Gitee", or asks to check cross-platform repo differences. 关键词：开源发布、双平台同步、脱敏、技能打包。Keywords: open-source publishing, GitHub Gitee sync, desensitization, skill packaging, release assets"
version: "1.2.0"
---

# Skill 开源发布流程

## 适用场景
- 把一次任务经验/事故复盘/工作流固化为 skill 并开源
- 用户已有 GitHub 仓库，需要制作/更新/检查 skill 文件
- ClawHub 等平台的 skill 活文档更新、双副本分叉合流、改名/多库运营

## 执行步骤

1. **确认脱敏边界**（最优先）：
   - 用户名、站点域名、项目名、API地址 → 全部移除或泛化
   - 真实对话引语若有力可保留（如"我什么时候问过"），但不得含身份指向
   - 具体数字泛化（"5.95折"→"具体到小数点的折扣"），除非数字本身无指向性
2. **制作 SKILL.md**：
   - frontmatter（name + description）**保持纯英文**——description 是触发器，混语言影响匹配
   - description 写法：一句话说清"何时用"，具体到 agent 能自动匹配任务场景
   - 正文结构：失败模式/背景 → 机制分析 → 规则/铁律 → 自检清单 → 正误示例 → 给维护者的建议
   - 双语需求时正文用段落级对照（中文在上英文紧随），不是整篇分开
3. **配套文件**：README.md（背景+适用场景+安装+许可，双语锚点导航 `[中文](#中文) | [English](#english)`，**中文在前**——2026-09-04 用户定的硬性规范，旧库英文在前的要改）+ LICENSE（默认 MIT；用户点名 MIT-0 时从之）
4. **打包交付**：zip 整个目录；本地源文件留档在 outputs/ 下
5. **线上检查**（用户推完后）：
   - ⚠️ **raw.githubusercontent.com 有CDN缓存，刚push拉raw可能拿到旧版**——不能据此判断"没推上"
   - 正确做法：查 commits API（`/repos/{owner}/{repo}/commits?path=文件`）看提交时间，或 raw URL 加 `?nocache=$(date +%s)` 穿透缓存
6. **建议用户设置**：Topics（ai-agents、llm-safety 等精准词+skill等泛词）、仓库描述（双语）
7. **发布后必须更新主页自述文件**（2026-09-07 起强制）：
   - 每次发布新 skill/项目后，把新仓库加入**同名主页仓库**的 skill 列表（`{owner}/{owner}` 仓库 README，渲染到个人主页），并同步更新件套数（如"十四件套"→"十五件套"）
   - 中英两个语言区都要加：中文表格加一行 + 英文表格加一行 + 各自末尾的汇总句同步数字
   - 两平台都要推：GitHub `{owner}/{owner}` + Gitee `{owner}/{owner}`，内容逐字节一致
   - **验证**：匿名 raw 拉取（raw 有 CDN 缓存，加 `?nocache=$(date +%s)` 穿透）或带 token 的 contents API 解码检查，确认新仓库名出现在自述里

## 双平台发布（GitHub + Gitee 同步）

1. **建仓**：GitHub `POST /user/repos`（JSON body）；Gitee `POST /api/v5/user/repos` —— ⚠️ **布尔字段必须走 JSON body**，表单传字符串 `"false"` 会被后端当真值 → 仓库全私有（2026-09-04 实测 5 库全中招）。★ **Gitee 新建空仓库无论如何改不成公开**——PATCH `private:false` 会报 `{"error":{"base":["空仓库不支持设置为公开仓库"]}}`（JSON 里显式 false 也没用）。**正解：先把内容 push 上去，再 PATCH 改公开**（PATCH 必须带 `name`，token 可走 JSON body 的 `access_token`）；改完必须匿名 HTTP 200 复核（2026-10-04 实测）
2. **推送**：令牌走环境变量 + 一次性 URL `https://x-access-token:$GH_TOKEN@github.com/...`（不设 remote，避免令牌落盘 .git/config）；curl 建仓响应不回显原文
3. **更新已有远端仓库**：clone 到临时目录后 `rsync -a --exclude='.git' src/ dst/` —— **`cp -r src/. dst/` 会连 .git 覆盖 clone 历史**，产生 "nothing to commit" 假象（本地看着干净，远程实际没收到更新）
4. **Gitee 改仓库属性**（如 private→public）：PATCH `/api/v5/repos/{owner}/{repo}` **必须带 `name` 字段**否则 400 "name is missing"；布尔同样走 JSON
5. **公开性验证（金标准）**：匿名 HTTP 200（不带 token 直接访问仓库页）——带 token 只证明"你自己能看"，不证明访客能看
6. **Profile 主页**：两平台都支持**与用户名同名的仓库**，其 README 渲染到个人主页（github.com/user/user + gitee.com/user/user）
7. **交叉比对**：拉两平台仓库列表互查缺口，同名不同项目用 alias 映射（如 SilverFox-Detector ↔ silver-fox_-detector_fixed），再逐字比对描述确认是否同项目
8. **令牌期限入档**：GitHub PAT 默认 90 天，发放当天记到期日，临期提醒用户换新
9. **Release 附件管理**（2026-09-05 实战）：附件 id 要走 `GET /repos/{owner}/{repo}/releases/{id}/attach_files`（详情/列表 API 的 assets 字段不含 id）；删除附件 `DELETE .../attach_files/{attach_id}`；上传 `POST .../attach_files` **不认 `application/octet-stream`，用 multipart `-F 'file=@xxx'`**
10. **大文件双平台推送防 504**：5MB+ 附件上传 / 7MB+ contents API PUT，Gitee 侧带宽极慢（~13KB/s 级），前台等必被网关超时杀——一律 `setsid nohup ... & ` 后台跑 + flag 文件轮询（实测 5-7MB 各约 3 分钟）
11. **发布文件名去中文**：Release 资产与仓库内文件名用纯 ASCII（如 `bountifulfares-1.3.0-1.20.1.jar`），下载链接稳定、跨平台兼容；中文描述放 Release 说明里
12. **Gitee raw 必须跟随重定向（`curl -L`）**：`gitee.com/{o}/{r}/raw/{branch}/{file}` 对脚本/文本类文件会返回一个 HTML 存根（`<a href="https://raw.giteeusercontent.com/...">Found</a>`），不加 `-L` 就会把几百字节的存根当真内容，误判"两平台内容不一致"。GitHub raw 无此问题（2026-10-04 实测）
13. **Gitee contents API 的 `branch` 要对**：各自仓库默认分支可能不同（如主页仓库 `mowenqwq/mowenqwq` 默认是 `main` 不是 `master`），PUT 传错分支名报 404 `{"message":"branch"}`。改前先 `GET /repos/{o}/{r}` 读 `default_branch`

## 活文档维护（分叉合流 + 导航层 + 多库路由）

### 双副本分叉合流（同一 skill 出现两份并行迭代的副本时）
1. **先对齐基线再动笔**：diff 两副本全量差异，逐项分类"真增量 vs 旧版已有"（日志先行、正文未落章的内容只有展开成章节才算真增量）
2. **编号冲突让位已发布 tag 链**：用户侧新章节若与线上已发布版本撞号，重编号接入主线（实战：新 §35/§36 → §37/§38），版本号同理，日志里记录让位关系
3. **互补内容全保留**：一侧独有的索引/引导块/反向提醒/双语 description 不覆盖，逐项核对后并入
4. **合流日志记全要素**：重编号映射、版本让位、新增内容摘要、互补保留清单、脱敏声明
5. **合流后自动化校验**：章节连续性、索引引用与正文章节双向覆盖（引用的都存在 + 存在的都被引用）、代码块配对、版本号与日志条目一致

### 三层导航索引（长文档 skill 的检索效率核心）
- **§0.1 症状索引**：按"用户遇到的报错/异常现象"组织，分主题组，备注列标注"必读=真因所在，其余是弯路"
- **§0.2 任务场景索引**：「我要做 X」反向入口，每条给按依赖顺序排好的章节链——解决"不知道症状叫什么"的冷启动
- **§0.3 主题地图**：全部章节按主题域归类，说明编号规则（如时间倒序）与主题无关的事实
- 活文档协议加强制规则：新增章节必须同步更新三层索引，否则索引随迭代腐化（实测：22 行索引漏了 8 个章节）

### 多库路由（改名/拆库运营）
- 改 slug = 新库（旧库数据留存），先和用户确认"新开共存"还是"rename 保留重定向"（CLI 两者都支持）
- 双库分工要用户明确定义并记录：哪个收全量、哪个收垂类、查数据按 slug 区分不混算
- 发布后异步审核：受理回执 ≠ 上线，设一次性 cron 复查任务（判定标准写进任务 prompt：tags.latest + description 正文抽查），完成后自删

## 踩坑记录
- 2026-08-24：检查用户刚 push 的双语版时，拉raw拿到CDN缓存的旧版，误报"没推上"，被指出"你是不是没刷新"。此后检查远端更新一律走 commits API + 时间戳穿透
- 2026-08-24：send_file 发同名更新文件可能失败/不达，改用新文件名（如 README_grounded_summaries.md）再发即可
- 2026-08-29：同一 skill 三次双副本分叉（用户在另一会话迭代，主线不知情）——根因是双方不同步 tag 链就动笔写新章节。合流前必先 diff + 对齐线上已发布版本号
- 2026-08-29：ClawHub inspect 大 JSON 前有 "- Fetching skill" 提示行，raw_decode 前先 `raw.index('{')` 定位起点；重名 slug（如 powershell）要带 @owner/ 前缀查询
- 2026-09-04：Gitee 表单传 private=false → 5 仓库全私有（用户 23 分钟后发现）。修复 PATCH 缺 name 又 400 一次。此后布尔一律 JSON body，公开验证一律匿名 HTTP 200
- 2026-09-04：cp -r src/. dst/ 覆盖 .git → "nothing to commit" 假象，差点误判"已更新"。更新远端仓库一律 rsync --exclude='.git'，推完查 commits API 确认
- 2026-09-04：银狐 GitHub 仓库（SilverFox-Detector，08-25 建）存在但档案只记了 Gitee → 误判"没发过"。档案缺失≠事情没发生，"有没有做过 X"类事实判断先查平台 API
- 2026-09-04：元模式——**跨平台做同一件事，先假设两边语义不一致**（GitHub JSON 语义套 Gitee 表单、GitHub raw 验证套 Gitee raw 403 都是这根因），双平台任务分别验证
- 2026-09-07（三平台发布一次通，3 个新 skill）：① **Gitee POST /user/repos 即使 JSON body 显式 `private:false` 仍建出私有**（此前"JSON body 即可避免"失效）——建仓后必须逐库匿名 HTTP 200 验证（403=私有），再用 PATCH（**必带 name**）改 public；② **Gitee git push URL 不认 `x-access-token:` 前缀**——会报 `The token username invalid` 403，必须 `https://<username>:<token>@gitee.com/...`；③ ClawHub 新发布走 moderation `pending.publication`，search 暂时查不到、但 `inspect @owner/slug` 能看到状态，CLEAN 后自动公开（属预期，勿判失败）

- 2026-10-04：**Gitee 空仓库不能设公开**——建仓后立刻 `private:false` 报「空仓库不支持设置为公开仓库」，正解是先 push 再改（已升为双平台发布第 1 条的硬规则）。同日：Gitee raw 不跟重定向会拿到 HTML 存根、首页仓库默认分支是 `main` 非 `master`（已入第 12/13 条）

## 更新日志
- v1.2.0（2026-10-04）：**补 Gitee 三处平台差异**——① 空仓库无法直接设公开（必须先 push 再 PATCH）；② raw 需 `curl -L` 跟重定向否则拿到 HTML 存根；③ contents API 的 branch 要按各仓库 `default_branch`（主页仓库为 `main`）。来源：开源 nginx-watchdog 时的实测。
- v1.1.0（2026-09-07）：执行步骤新增第 7 条「发布后必须更新主页自述文件」——发布后把新仓库加进同名主页仓库 skill 列表（中英双语区同步 + 件套数 + 双平台推送 + raw/API 验证）。来源：发布 agent-self-rollback 后用户提醒"自述文件更新没，记得把规则加进习惯"。
