# PaperGuard

本地运行的学术参考文献核验工具。导入 LaTeX 项目或 BibTeX 文件，核对文献来源与元数据、检查正文引用，并整理已确认论文的原始首页和核验报告。

**未检索到不等于文献造假。** PaperGuard 保留检索来源、查询时间、匹配依据和字段差异，明确区分未验证、来源失败与需要人工复核的情况。基础核验无需 LLM，原始 `.tex`、`.bib` 和用户 PDF 不会被覆盖。

## 主要功能

| 功能 | 说明 |
| --- | --- |
| 文献与引用解析 | 读取 `.bib`，递归解析 LaTeX 的 `input` / `include`、`bibliography` / `addbibresource` 及常见引用命令，记录引用文件、行号与章节 |
| 在线元数据核验 | 查询 Crossref、OpenAlex、arXiv，比较标题、作者、年份、DOI、期刊或会议等字段，保留来源证据 |
| 引用完整性检查 | 检查缺失或重复的 citation key 等问题；支持不联网的离线模式 |
| PDF 归档 | 从开放获取位置下载论文，或上传有权使用的 PDF；校验文献身份后提取和合并原始首页 |
| 修正与报告 | 导出 HTML、Excel、JSON、BibTeX 和未解决条目清单；修正建议可逐条确认 |
| 可选语义审查 | 使用兼容 chat-completions 的模型服务，根据引用上下文与已匹配全文片段辅助审查引用是否得到支持 |

## 快速开始

### 环境要求

- Python **3.11+**。
- Node.js **20.19+（20.x）或 22.12+**，并安装 npm；版本范围以 [frontend/package.json](frontend/package.json) 为准。
- 首次安装需要网络，用于下载 Python 和前端依赖；安装后可离线检查解析与引用完整性。

当前从源码目录安装和运行。仓库不包含虚拟环境、前端依赖与构建产物、用户配置或论文 PDF；单独安装后端 wheel 不包含 Web 界面。

### Windows

下载仓库 ZIP 并解压，或克隆仓库。在项目根目录先双击 **`安装PaperGuard.cmd`**，安装完成后双击 **`启动PaperGuard.cmd`**。

也可以在 PowerShell 中运行：

```powershell
cd "C:\path\to\PaperGuard"
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\Setup.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\Start.ps1
```

### macOS / Linux

在项目根目录运行：

```sh
sh scripts/setup.sh
.venv/bin/python -m paperguard serve
```

安装脚本会创建 `.venv`、安装依赖、构建前端，并在不存在 `.env` 时从示例创建配置。

启动后打开 [http://127.0.0.1:8765](http://127.0.0.1:8765)。保持服务终端开启，按 `Ctrl+C` 停止。服务仅监听本机，面向单用户使用。

### 完成第一次核验

1. 点击 **新建核验**，填写论文项目目录或 `.bib` 文件的绝对路径，也可以上传 `.bib` 或 LaTeX 项目 ZIP。可先使用仓库中的 [examples/resnet](examples/resnet) 真实文献示例。
2. 本地目录含多个主 `.tex` 文件时，填写主文件，如 `main.tex`。
3. 选择核验方式：在线核验查询所选学术数据源；离线模式仅做解析和引用完整性检查，文献真实性保留为未验证。PDF 获取和语义审查可按需选择。
4. 在文献详情查看来源证据与逐字段差异，必要时重试，并逐条确认可靠字段的修正建议。
5. 在 **PDF 归档** 查看下载结果或上传对应论文 PDF。可能匹配的文件需要人工确认；错误匹配不进入首页汇总。
6. 在报告页下载核验结果。示例不会预填检索结果或统计值，在线结果取决于实际来源响应。

## 如何理解结果

文献核验、PDF 身份和语义审查分别记录状态；元数据核验通过并不表示一定能取得全文。

| 文献核验状态 | 含义 |
| --- | --- |
| `verified` | 找到强匹配来源，未发现关键字段冲突 |
| `metadata_mismatch` | 找到强匹配来源，但输入元数据与来源存在关键差异，需要查看具体字段 |
| `possible_match` | 存在可能对应的候选，证据尚不足以确认 |
| `manual_review` | 强匹配来源之间存在身份或元数据冲突，需要人工复核 |
| `not_found` | 查询已成功完成，但所查来源没有足够的匹配证据 |
| `unverified` | 离线、未查询，或来源失败且没有足够匹配证据 |

PDF 身份为 `confirmed` 时可进入汇总，`probable` 需要人工确认，`mismatch` 始终排除。没有可读取的 `.aux` / `.bbl` 编译产物时，首页按源码引用出现顺序排列，不能保证等于最终参考文献编号；报告会说明排序依据。

语义审查以可追溯的原文片段为依据。未配置模型时为 `not_checked`，缺少充分全文证据时为 `insufficient_evidence`；模型输出仍需人工复核，不能作为文献真实性的数据源。

## 命令行使用

先在项目根目录激活虚拟环境：

```powershell
# Windows PowerShell
.\.venv\Scripts\Activate.ps1
```

```sh
# macOS / Linux
. .venv/bin/activate
```

激活后可使用相同的命令（路径按实际位置替换）：

```sh
# 解析并在线核验示例项目，默认尝试获取开放获取 PDF
paperguard audit ./examples/resnet --main main.tex

# 核验单个 BibTeX 文件
paperguard audit --bib /path/to/references.bib

# 仅检查解析与引用完整性，不发起在线核验
paperguard audit ./examples/resnet --offline

# 在线核验但不下载 PDF；或启用已配置的语义审查
paperguard audit ./examples/resnet --no-pdf
paperguard audit ./examples/resnet --semantic

# 恢复任务、重试指定 citation key、上传并确认可能匹配的 PDF
paperguard resume TASK_ID
paperguard retry TASK_ID CITATION_KEY
paperguard upload-pdf TASK_ID CITATION_KEY /path/to/reference.pdf
paperguard approve-pdf TASK_ID CITATION_KEY

# 刷新报告、指定数据目录或清理查询缓存
paperguard report --task TASK_ID --format html
paperguard --data-dir /path/to/output audit ./examples/resnet
paperguard clear-cache
```

- 未激活环境时，Windows 可使用 `.\.venv\Scripts\paperguard.exe`，macOS / Linux 可使用 `.venv/bin/paperguard`。路径包含空格时请加引号。
- `--data-dir` 是全局选项，必须放在子命令之前。
- `report --format` 接受 `html`、`xlsx`、`json` 或 `all`；当前实现均刷新全部报告，以保持内容一致。
- `collect-pdf` 与 `audit` 一样会创建新任务。继续已有任务或补充 PDF 请使用 `resume`、`upload-pdf` 或 `report`。
- `resume` 跳过已处理条目；重新查询某条文献请用 `retry`。输入文件变化后需要新建任务。
- 正常或部分结果退出码为 `0`，任务失败为 `1`，输入错误为 `2`。退出码 `0` 不表示全部文献均已核验通过。

## 输出文件

默认任务目录为 `data/tasks/<task_id>/`；配置数据目录后位于对应目录的 `tasks/<task_id>/`。

| 文件或目录 | 内容 |
| --- | --- |
| `Reference_First_Pages.pdf` | 已确认论文的原始首页，无占位页、标签或水印；没有合格页面时不生成空 PDF |
| `Reference_First_Pages_Indexed.pdf` | 相同首页，附参考文献编号与标题书签 |
| `Reference_Verification_Report.html` | 可离线打开，支持搜索、状态筛选与展开来源证据 |
| `Reference_Metadata.xlsx` | Summary、References、Metadata_Differences、Citation_Issues、Semantic_Audit、PDF_Status 六个工作表 |
| `Verification_Records.json` | 完整记录、原始来源数据与汇总信息 |
| `Corrected_References.bib` / `Correction_Changes.json` | 可靠字段的建议修正版及字段变更记录 |
| `Approved_References.bib` | 仅应用在界面中逐条确认的修正建议 |
| `unresolved_references.csv` | 尚未解决的元数据或 PDF 条目 |
| `Original_PDFs/` / `First_Pages/` / `Logs/` | 归档 PDF、独立首页与任务日志 |

## 配置与隐私

本地配置保存在项目根目录的 `.env`，参见 [.env.example](.env.example)。安装脚本会调用 `paperguard init` 创建该文件，不覆盖已有配置。修改 `.env` 后请重启服务。

| 配置项 | 用途与默认值 |
| --- | --- |
| `PAPERGUARD_SOURCES` | 安装生成的配置启用 `crossref,openalex,arxiv` |
| `PAPERGUARD_EMAIL` | 可选的检索联系邮箱 |
| `PAPERGUARD_TIMEOUT` | 来源请求超时，默认 `20` 秒 |
| `PAPERGUARD_CACHE_TTL` | 查询缓存有效期，默认 `604800` 秒（7 天） |
| `PAPERGUARD_DATA` | 可选数据目录，默认项目根目录下的 `data/` |
| `OPENALEX_API_KEY` | OpenAlex 密钥，按数据源要求配置 |
| `PAPERGUARD_LLM_BASE` | 可选模型服务地址，需包含 `/v1` |
| `PAPERGUARD_LLM_MODEL` | 模型名称 |
| `PAPERGUARD_LLM_KEY` | 模型服务密钥；本地服务可不要求密钥 |

例如，使用已安装模型的本地 Ollama 服务：

```dotenv
PAPERGUARD_LLM_BASE=http://127.0.0.1:11434/v1
PAPERGUARD_LLM_MODEL=your-installed-model
PAPERGUARD_LLM_KEY=
```

配置模型后，还需在新任务中启用语义审查，或使用 CLI 的 `--semantic`。vLLM 等兼容 chat-completions 的服务也可配置。基础核验不依赖模型服务。

在线检索向所选来源发送文献标题与 DOI；启用语义审查时，会向配置的模型服务发送必要的引用上下文与检索出的原文片段。PDF 仅通过来源标明的开放获取位置或用户授权上传获取，不绕过付费墙、登录或验证码。论文全文的再分发权限由其版权和许可决定。

本地配置、缓存和任务数据不应提交到仓库。可在设置页清理查询缓存，在报告页删除任务。来源鉴权、限流和网络失败会明确记录，不会被当作“没有这篇文献”。

## 常见问题

- **8765 端口已占用**：Windows 使用 `powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\Start.ps1 -Port 8766`；其他平台使用 `.venv/bin/python -m paperguard serve --port 8766`，然后访问对应端口。
- **界面未构建或修改前端后没有更新**：在项目根目录运行 `npm ci --prefix frontend` 和 `npm run build --prefix frontend`，再启动服务。
- **任务中断或来源请求失败**：使用界面的“继续任务”恢复尚未处理条目；已处理但需重新查询的条目使用“重试此文献”。取消操作在当前文献处理结束后生效。
- **没有生成首页汇总**：检查 PDF 是否可用、能否读取文本，以及文献身份是否已确认。`probable` 状态需要人工确认；不匹配的 PDF 不会合并。

## 开发与验证

后端使用 Python、FastAPI、Pydantic 和 SQLite；前端使用 React、TypeScript 和 Vite，不依赖外部 CDN。后端按解析、数据源、核验、PDF、语义审查、报告与存储分模块，任务在单机串行队列中运行。

激活虚拟环境后，在项目根目录执行：

```sh
python -m pytest backend/tests -q
npm test --prefix frontend
npm run build --prefix frontend
python scripts/check_release.py
```

默认测试使用隔离数据，不依赖外部学术 API。单独执行 `python scripts/online_smoke.py` 可验证真实来源；该命令会联网并下载公开 PDF。CI 配置包含 Windows、macOS、Linux 的 Python 3.11 / 3.12 后端测试，以及前端测试与构建；历史实测结果参见验收文档。

- [开发与接口说明](docs/README.md)：数据恢复、JSON 合约、HTTP API、数据源与依赖许可。
- [JSON Schema](docs/audit.schema.json)：核验记录的机器可读结构。
- [实际验收记录](docs/VALIDATION.md) / [发布检查](docs/RELEASE_CHECK.md)：已执行的验证及其范围。
- [贡献指南](CONTRIBUTING.md) / [安全说明](SECURITY.md)：开发检查与安全问题反馈方式。

当前不支持 RIS、OCR、从整篇投稿 PDF 自动重建参考文献、任意自定义 TeX 宏展开、复杂 biblatex 多重引用与分区编号。应用不执行 TeX、不自动编译项目；扫描 PDF 的文本识别需要外部 OCR。语义审查使用启发式片段检索，无法替代人工阅读。

项目自有代码采用 [Apache-2.0](LICENSE) 许可证，参见 [NOTICE](NOTICE)。第三方依赖、来源内容与论文保留各自许可。
