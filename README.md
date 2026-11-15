# PaperGuard

本地运行的学术参考文献核验工具。支持导入 LaTeX/BibTeX、递归解析正文引用，查询真实学术数据源，逐字段比对元数据，合法获取或手动上传原始论文 PDF，核对文献身份、提取首页并合并，导出 HTML / Excel / JSON / BibTeX。

**未检索到不等于文献造假。** 不确定、网络失败、缺乏全文、模型不可用均有明确状态。LLM 不是文献真实性的数据源；原始 `.tex`、`.bib` 和用户 PDF 不会被覆盖。

## Windows 启动

从 GitHub 下载 ZIP 并解压，或克隆仓库后，先双击 **`安装PaperGuard.cmd`**。安装完成后双击 **`启动PaperGuard.cmd`**，打开 **http://127.0.0.1:8765**。命令窗口保持开启，Ctrl+C 停止服务。源码仓库不包含虚拟环境、前端依赖、构建文件、用户配置和论文 PDF。

首次在另一台电脑安装，需要 Python 3.11+、Node.js 20.19+（或 22.12+）与网络，然后双击 `安装PaperGuard.cmd`。PowerShell 等价命令：

```powershell
cd "C:\path\to\PaperGuard"
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\Setup.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\Start.ps1
```

如果 8765 已占用：`powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\Start.ps1 -Port 8766`。

## 使用流程

1. 新建核验，填写论文项目目录或 `.bib` 的绝对路径；也可以上传 `.bib` 或 LaTeX 项目 ZIP。
2. 多个主 `.tex` 文件时指定主文件。标准的 input/include、bibliography/addbibresource 和常见单组引用命令可解析，正文引用保存文件、行号与章节。
3. 选择在线或离线模式。在线检索发送标题与 DOI；离线模式仅做解析和引用完整性，真实性保持未验证。
4. 查看文献详情中的来源、查询时间、匹配依据、逐字段差异。每条可重试，可靠字段修正可逐条确认。
5. PDF 归档页查看开放获取结果，或为指定文献上传你有权使用的原始论文 PDF。可能匹配的文件需人工确认，错误匹配会被排除。
6. 报告页导出全部结果。原始首页按编译参考文献顺序合并；没有可读取的 `.aux/.bbl` 时，明确采用源码引用出现顺序，不能保证等于最终编号。

示例论文项目在 `examples/resnet`。这是可在线检索的真实文献示例，默认不预填任何检索结果或统计值。

## CLI

```powershell
.\.venv\Scripts\paperguard.exe init
.\.venv\Scripts\paperguard.exe audit .\examples\resnet --main main.tex
.\.venv\Scripts\paperguard.exe audit --bib C:\Research\references.bib
.\.venv\Scripts\paperguard.exe audit C:\Research\paper --semantic
.\.venv\Scripts\paperguard.exe collect-pdf .\examples\resnet
.\.venv\Scripts\paperguard.exe resume TASK_ID
.\.venv\Scripts\paperguard.exe retry TASK_ID CITATION_KEY
.\.venv\Scripts\paperguard.exe upload-pdf TASK_ID CITATION_KEY C:\Research\reference.pdf
.\.venv\Scripts\paperguard.exe approve-pdf TASK_ID CITATION_KEY
.\.venv\Scripts\paperguard.exe report --task TASK_ID --format html
.\.venv\Scripts\paperguard.exe audit .\examples\resnet --offline
```

`--data-dir` 是全局选项，放在子命令之前。report 的 format 指定使用者目标格式；当前统一刷新全部报告以保证内容一致。collect-pdf 创建新的完整任务；续用已验证 PDF 请使用 resume、upload-pdf 或 report。正常/部分结果退出码 0，任务失败 1，输入错误 2。

## 输出

每次任务输出：`data/tasks/<task_id>/`。

- `Reference_First_Pages.pdf`：原始首页，无占位页、标签或水印；没有合格页面时不生成空 PDF。
- `Reference_First_Pages_Indexed.pdf`：相同页面，附编号与标题书签。
- `Reference_Verification_Report.html`：离线可打开、搜索、按状态筛选与展开来源证据。
- `Reference_Metadata.xlsx`：Summary、References、Metadata_Differences、Citation_Issues、Semantic_Audit、PDF_Status 六个表。
- `Verification_Records.json`：完整记录、原始来源数据与 summary。
- `Corrected_References.bib` / `Correction_Changes.json`：可靠字段的建议修正版与字段变更记录。
- `Approved_References.bib`：只应用在界面逐条确认的修正建议。
- `unresolved_references.csv`：未解决的元数据或 PDF 条目。
- `Original_PDFs/`、`First_Pages/`、`Logs/`：校验后的原始文件、独立首页与任务日志。

## 配置与隐私

复制 `.env.example` 为 `.env`（init 会自动完成）。默认 Crossref、OpenAlex、arXiv，无需 LLM。可配置联系邮箱、超时、缓存、数据目录以及 OpenAlex key。来源条款和可用额度可能变化，出现鉴权/限流时状态明确降级，不将失败伪装成无文献。

可选语义审查设置 `PAPERGUARD_LLM_BASE`、`PAPERGUARD_LLM_MODEL`、`PAPERGUARD_LLM_KEY`，base 包含 `/v1`。Ollama 如 `http://127.0.0.1:11434/v1`；vLLM 或其他兼容 chat-completions 服务同理。仅勾选语义审查时发送必要正文上下文和检索原文片段。没有模型、API key 或充分全文证据时基础功能正常运行，语义结果标明 not_checked / insufficient_evidence。

PDF 只通过数据源标明的开放获取位置和用户授权上传获取，不绕过付费墙、登录或验证码。请遵守各来源条款与论文版权，下载成功不意味着可再分发全文。本地数据和日志可从报告页删除任务、设置页清理缓存。服务只监听 127.0.0.1；不能作为公网多用户应用。

## Linux / macOS

```sh
sh scripts/setup.sh
.venv/bin/python -m paperguard serve
```

代码与脚本支持三平台；本次实机测试在 Windows 进行。

## 测试、架构与限制

```powershell
.\.venv\Scripts\python.exe -m pytest .\backend\tests -q
npm.cmd run build --prefix frontend
.\.venv\Scripts\python.exe .\scripts\online_smoke.py
```

离线测试独立于外部 API，覆盖解析、匹配冲突、网络失败、PDF 校验与合并、任务取消和恢复、API、ZIP 路径校验及报告转义。在线 smoke 单独记录真实服务结果。详细架构、JSON schema、HTTP API、数据源官方文档与限制见 [docs/README.md](docs/README.md)。实际验收记录见 [docs/VALIDATION.md](docs/VALIDATION.md)，GitHub 发布前检查见 [docs/RELEASE_CHECK.md](docs/RELEASE_CHECK.md)。

当前尚不支持 RIS、OCR、投稿 PDF 的自动参考文献重建、任意自定义宏展开、复杂 biblatex 多重引用与分区编号。没有匹配全文时不以摘要缺失信息断言“不支持”。语义判断是带出处的辅助审查，需人工复核；本次未配置真实 LLM 服务。

后端模块：parsers / sources / verification / pdf / semantic / reporting / storage / engine / api / cli。Pydantic 约束模型、SQLite 缓存与逐条快照、单机串行任务队列。前端 React + TypeScript + Vite，不依赖外部 CDN。前端代码与构建均在 frontend 目录

贡献前请运行默认离线测试，新增匹配规则应补充错误 DOI、歧义候选和边界情况测试。自有代码采用 Apache-2.0；第三方依赖及文献本身保留各自许可，参见 LICENSE 与 docs 的依赖清单
