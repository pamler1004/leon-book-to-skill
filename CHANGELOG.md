# 变更记录

本文件记录 **book-to-skill** 的所有显著变更。

格式基于 [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)，本项目遵循[语义化版本](https://semver.org/spec/v2.0.0.html)。

## [未发布]

### 新增
- 为本地 Skill 校验器新增可移植的 Codex descriptor 与结构化 eval fixture。

### 变更
- 厘清 Leon 的本地布局：私有 Skill 源码放在 `~/Project/Skills/`，而 `leon-<name>` 保留给运行时软链和本地调用。
- 将详细转换规范拆分到单独的 reference 文件，让常驻加载的 `SKILL.md` 保持精简。

### 文档
- 厘清两条安装路径以免混淆：**`git clone` 到 skills 目录**注册 `/leon-book-to-skill` agent skill（Codex / Claude Code / Copilot CLI / Amp），而 **`pip install book-to-skill`** 只安装独立的提取 CLI，不注册 skill。README 和文档首页现在都明确列出两者。
- README 现在以实测数据（比 context-dump 少 24×–51× token）和 3 步「工作原理」开头，让价值在首屏落地，而不是埋在页面中部。

### 安全
- **DOCX XXE / Billion Laughs 加固**--DOCX 提取器现在会在解析前扫描归档，拒绝任何声明 DTD 或 entity 的 XML part，阻断 XML 外部实体和实体膨胀攻击（#53、#54）。
- **子进程参数注入加固**--文件路径在传给 `pdftotext` / `pdfinfo` / `ebook-convert` 前先转为绝对路径，避免以 `-` 开头的文件名被当作命令行选项（#53、#54）。
- **PR 上的依赖 CVE 审查**--`dependency-review` CI job 会标记任何带中危及以上 CVE（或被拒许可证）的新依赖，并将结果作为 PR 评论发出。Dependabot 现在也覆盖 `pip` 生态。

### 变更
- **`pdf` extra 现在安装 `pypdf` 而非已弃用的 `PyPDF2`**（`pip install book-to-skill[pdf]`）。`pypdf` 是维护中的继任者；`PyPDF2` 已 end-of-life，不再收到安全修复（#54）。

### 修复
- 以 UTF-16 或 UTF-32 保存的文本文件（`.txt`、`.md`、`.rst`、`.adoc`、`.html`、`.rtf`，例如 Windows 记事本的「Unicode」或 PowerShell 输出）现在按 BOM 解码，而非被当作 `cp1252`/`latin-1` 乱码读取。
- 无依赖的 RTF fallback（未安装 `striprtf` 时使用）现在能解码 `\uN` unicode 转义--智能引号、破折号、重音字母--而不是丢弃它们只留 ASCII fallback 字符。
- 标准库 HTML parser（未安装 BeautifulSoup 时用于 HTML 文件和 EPUB 提取的 fallback）不再二次解码 HTML entity，因此 `&amp;amp;` 这类双重编码的 entity 能完好保留。
- 无依赖的 DOCX fallback（未安装 `python-docx` 时使用）现在按文档顺序把表格重建为 tab 连接的行，而不是把每个单元格摊到单独一行。
- 无依赖的 EPUB 提取器（未安装 `ebooklib` 时使用）现在按真正的 spine（阅读）顺序读取内容，而非 manifest 顺序，章节不再错乱。未列入 spine 的内容文档仍会包含（追加在 spine 内容之后）。

## [1.2.0] - 2026-06-17

### 新增
- **可安装的 Python 包。** 提取器现在是规范的 `book_to_skill` 包，带 `pyproject.toml`（hatchling 构建后端）、`book-to-skill` console script 和 `python -m book_to_skill`。可选提取器以 extras 暴露（`epub`、`pdf`、`docx`、`rtf`、`technical`、`all`）；基础安装保持零依赖，带标准库 fallback。`requires-python = ">=3.9"`。`scripts/extract.py` 作为薄 shim 保留，现有 skill 流程不变（#34、#35、#48）。
- **Markdown / AsciiDoc 标题检测。** 结构检测在没有数字「Chapter N」标题时，会把 ATX 标题（`#`、`==`）识别为章节，修复 `.md` / `.adoc` 源返回零章节的问题。代码块内的标题会被忽略（#44）。
- **setext / reStructuredText 下划线标题**--一行 `=` 或 `-` 上方的标题行现在会被检测到，`.rst` 和 setext 风格 Markdown 不再返回零章节。对 thematic break、表格边框和 YAML front matter 做了防护（#51）。
- **更多章节语言。** 章节词检测现在覆盖法语、德语、意大利语和荷兰语（`Chapitre`、`Kapitel`、`Capitolo`、`Hoofdstuk`），以 `Ü`/`Û`/`Ý`/`Þ` 开头的标题（如「Überblick」）也被接受（#49）。
- **多语言目录检测**--中文、日文、法文、德文、意大利文、荷兰文（#44）。

### 修复
- **CJK 章节标题中的全角阿拉伯数字**--`第１章`（U+FF10–FF19，日文排版常见）现在像 `第1章` 一样被检测到（#46）。
- **解析器错误不再被静默吞掉。** 任何提取器的意外异常都会输出到 stderr（提取器名 + 异常类型），同时 fallback 链仍返回 `None` 并继续，损坏的文件和编码错误因此可诊断（#47、#50）。
- **全标点 ATX「标题」**（如 `=====   =====` 表格边框）不再被误计为章节（#51）。
- **在急切求值注解的解释器上包可正常导入。** 给每个使用 PEP 604 union（`str | None`）的模块加了 `from __future__ import annotations`，包在 Python 3.9 上能干净导入并运行（#34）。

### 安全
- **CI 安全扫描**--CodeQL（Python，security-and-quality + 每周计划）、Bandit（在 HIGH 严重度设门禁；MEDIUM+ 仅信息性报告）和 Zizmor（GitHub Actions workflow 审计，信息性），加上覆盖 `github-actions` 生态的 Dependabot 配置。已知待加固项：Bandit B314（DOCX parser 中的 `xml.etree.ElementTree.fromstring`）。

### 变更
- CI 测试矩阵现在包含 Python 3.9，上述导入路径得到守护，不会静默回退。

## [1.1.0] - 2026-06-12

### 新增
- **GitHub Copilot CLI 成为一等目标**--同一份 `SKILL.md` 现在能通过开放的 Agent Skills 标准在 GitHub Copilot CLI、Amp 和 Claude Code 上发现、安装和运行。Skill Locations 覆盖 8 条发现路径，脚本探针会遍历全部（#30）。
- **`validate_skill.py --lens claude|copilot|amp`**--按各宿主规则审计生成的 SKILL.md；`claude` 仍为 CI 向后兼容的默认值（#30）。
- **署名 banner**--`scripts/banner.txt` 在每次运行开始时打印（best-effort，不会让运行失败）。

### 变更
- `SKILL.md` frontmatter 向开放标准最小集裁剪，description 现在点名全部三个宿主，让每个 agent 的自动加载器都能识别（#30）。
- README 标题 +「Agent Skills」徽章；安装/用法章节覆盖全部三个宿主。`docs/ARCHITECTURE.md` 展示各宿主的目标路径（#30）。

### 备注
- 为宿主中立性，frontmatter 中去掉了 `allowed-tools`；skill 在三个宿主上都合规（用三个 lens 全部验证过）。如果 Claude 用户遇到权限提示摩擦，#18 的 Bash 授权会用 Claude 原生 token 恢复（Copilot 反正会忽略该 key）。

## [1.0.0] - 2026-06-08

首个正式打 tag 的 release。转换器稳定、多格式、已在真实书籍上验证。

### 新增
- **多格式提取**--PDF、EPUB、DOCX、HTML、Markdown、reStructuredText、AsciiDoc、RTF 和 MOBI/AZW/AZW3（经 Calibre），通过模块化的 `extractor` 包实现，含按格式的 parser 和优雅的标准库 fallback。
- **`extract.py --check`**--预检，报告每种格式装了哪些提取器，以及安装缺失项的确切命令（#21）。
- **自适应每章深度**--token 预算随 `BOOK_TYPE × DEPTH` 缩放；study 深度章节要求一个 worked example，cheatsheet 作为决策/推理层生成（决策规则、树、权衡、阈值、tells），而非关键词列表（#20）。
- **`tools/discovery_tax.py`**--度量「Discovery Loop Tax」：在真实书籍上，context-dump vs discovery loop vs book-to-skill 各自为回答一个问题放入 context 的 token 数（#23）。
- **Update / fold-in 工作流**--把新源合并进已有 skill，保持章节索引、主题索引、glossary、patterns 和 cheatsheet 同步。
- **GitHub Actions CI**--lint（ruff）、测试矩阵（py3.10–3.13）、无依赖 smoke test 和 SKILL.md Claude 合规验证（#15、#18）。

### 变更
- **README 定位**--版权 & fair-use 章节、「Beyond books」用例、context-dump / RAG / 1M-window FAQ，以及实测的 Discovery Loop Tax + 四本书的真实单次转换成本表（#19、#27）。
- 默认输出目标对 Claude Code 是 `~/.claude/skills/`，也支持 Amp skill 目录（#13、#14）。

### 修复
- **章节检测**--扫描全文（原来上限 50k 字符），计数明确的 `Chapter N` / `Capítulo N` 标题，拒绝编号列表项、行内交叉引用和年份；新增葡萄牙语支持（#26）。
- **罗马数字标题**--`I: Loomings`、`II. The Carpet-Bag` 现在带规范数字验证地被检测到（#28）。
- **EPUB 提取**--在标准库 zipfile fallback 中解析 OPF 相对 href（#11、#12）。
- **批量韧性**--一个坏源会被跳过并告警，而非中止整次运行；显式输入顺序得到保留（#7）。

### 已知限制
- 章节自动检测需要明确的 `Chapter N` / `Capítulo N` 或罗马数字标题。章首只有裸标题（如 *Moby-Dick*，数字只在目录里）或使用节标题（如 Pro Git）的书无法自动分段。
- 技术类 PDF 用文本模式提取可能丢失标题结构；用 technical 模式（Docling）以保留表格、代码和标题。

[1.2.0]: https://github.com/virgiliojr94/book-to-skill/releases/tag/v1.2.0
[1.1.0]: https://github.com/virgiliojr94/book-to-skill/releases/tag/v1.1.0
[1.0.0]: https://github.com/virgiliojr94/book-to-skill/releases/tag/v1.0.0
