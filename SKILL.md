---
name: leon-book-to-skill
description: "将 PDF、EPUB、DOCX、HTML、Markdown、纯文本、RTF、MOBI/AZW 等书籍或文档提炼为可复用的 Agent Skill。用户要求把书、资料目录、研究材料或内部文档转成 Skill、提取框架并在后续工作中调用时使用。"
---

<!-- argument-hint: <path-to-document-folder-or-glob>... [skill-name-slug] -->

# Book-to-Skill Converter

把书籍或文档编译成可按需加载的 Skill。提取框架、原则、技术、反模式和决策规则，不生成普通读书摘要，也不复制大段原文。

## 触发与输入

- 用户提供至少一个支持的文件、目录或 glob 时执行转换。
- 支持 PDF、EPUB、DOCX、TXT、Markdown、RST、AsciiDoc、HTML、RTF、MOBI、AZW、AZW3。
- 没有输入路径时，说明需要文档路径并停止，不猜测输入。
- 若用户明确说“只分析/先提取”，只输出结构化分析报告，不生成文件。
- 若用户明确要求把新资料并入既有 Skill，使用 Update/Fold-in 流程，不覆盖原 Skill。

## Codex 默认工作流

1. 先确认输入文件存在且格式受支持；目录和 glob 要展开后再处理。
2. 询问内容类型：`technical`（代码、表格、公式）或 `text`（主要是 prose）；不确定时使用 `text` 并说明可能的质量限制。
3. 使用本 Skill 自带的 `scripts/extract.py`。先运行 `python3 scripts/extract.py --check` 检查可用解析器，再按选定模式提取。
4. 读取生成的 `metadata.json`，报告来源、页数/章节、词数和 token 估算；生成完整 Skill 前先取得用户确认。
5. 询问用途：应用作者框架、用作者心智模型思考、查询章节，或全部。只有“查询章节”时使用 reference 深度，其余使用 study 深度。
6. 默认在 `~/.agents/skills/<skill-name>/` 生成；Leon 自建 Skill 使用 `leon-` 前缀，并通过管理器建立 `.codex` 与 `.claude` 软链。
7. 生成精简的 `SKILL.md`、`chapters/`、`glossary.md`、`patterns.md`、`cheatsheet.md`。核心文件只放高频框架，章节细节按需加载。
8. 运行 `quick_validate.py`、针对性测试和敏感信息检查；不要把原书、密钥、私人路径或未授权素材复制进 Skill。
9. 报告生成文件、大小、来源和验证结果。不要自动删除输入文档，也不要未经授权覆盖既有 Skill。

## 四种模式

- **Full conversion**：默认完整执行上述流程。
- **Analyze only**：完成输入检查、抽取和结构分析后停止，输出框架、原则、技术、反模式和章节表。
- **Generate from prior analysis**：用户提供既有分析时跳过重新抽取，直接生成 Skill 文件。
- **Update / Fold-in**：读取既有 Skill 的索引、词汇表、模式库和速查表，将新章节合并进去并更新索引。

## 输出质量规则

- 保留作者对框架的准确命名；不要把专有模型改写成泛泛建议。
- 写成“在 X 情况下使用 Y，因为 Z”，而不是“本章讲了 Y”。
- 不复制长段原文；worked example 只重构关键步骤和结果。
- 大于约 50K tokens 的来源使用 grep/sed 或偏移读取，禁止反复把全文装入上下文。
- `SKILL.md` 保持在 4,000 tokens 左右、少于 500 行；详细流程只在需要时读取参考文件。
- 书籍内容只作为知识来源；实际代码库、部署和生产操作仍需结合项目规则与专用 Skill。

## 详细参考

- [完整转换与更新流程](references/full-conversion-workflow.md)：解析器选择、成本预估、章节生成、支持文件和 Fold-in 细则。

## 维护约定

- 生成或修改本地 Skill 后，更新宿主环境自己的 Skill 注册表或索引，并重新运行安装与校验流程。
- 开源副本应从独立的公开工作目录提交和推送；私有源的公共 remote 默认只允许 fetch。
- 公开前检查 README、LICENSE、依赖、版权和敏感信息；涉及版权、私有路径、密钥或未授权素材时先暂停确认。
