---
name: gee-data-downloader
description: Use when the user says "调用GEE", "使用GEE", or "下载", or requests Google Earth Engine data selection, asset-path recognition, JavaScript export code, product metadata, units, aggregation, URLs, and literature citations for remote-sensing, reanalysis, or static geospatial products.
---

# GEE Data Downloader

## 目标

为遥感、再分析和静态环境数据生成可复现的 Google Earth Engine 下载方案。核心原则是先验证产品能覆盖用户要求的完整时间段，再生成与变量物理意义一致的聚合和单位换算代码。

## 运行环境

- Codex：安装到 `~/.codex/skills/gee-data-downloader/`，可自动触发或使用 `$gee-data-downloader` 调用。
- Claude Code：安装到 `~/.claude/skills/gee-data-downloader/`，可自动触发或使用 `/gee-data-downloader` 调用；项目级安装路径为 `.claude/skills/gee-data-downloader/`。
- 核心 `SKILL.md` 和 `references/` 遵循 Agent Skills 标准。`agents/openai.yaml` 只提供 Codex 界面元数据，Claude Code 可直接忽略该文件。

## 工作流

1. 从用户文字和截图提取研究区、起止年月、时间尺度、变量、分辨率、输出格式和命名要求。截图中的 Cloud Assets 路径写为 `projects/<project>/assets/<asset>`；Legacy Assets 写为 `users/<user>/<asset>`。示例研究区资产是 `projects/underground-water-488604/assets/xj`，使用时按用户提供的路径调整。
2. 使用当前运行环境可用的联网检索能力，并优先核验 GEE 官方数据目录、产品机构官网、数据 DOI 和原始论文。Codex 可使用已安装的 `agent-reach` 或网页检索工具；Claude Code 可使用 WebSearch、WebFetch 或已配置的等效工具。记录实际数据集 ID、版本、首末日期、更新延迟、波段、比例因子、单位、空间分辨率、质量控制和许可。
3. 按“完整覆盖时间范围、变量语义匹配、空间分辨率、质量与可引用性”选择产品。结束日期必须按当前实际可用影像核验；目录标注“至今”不等于用户要求的最后一个月已经发布。
4. 判定访问路径：GEE 公共目录可直接调用；外部产品需明确写出下载、转换、上传步骤，并把上传后的资产 ID 标为用户需替换的占位符。不得把外部产品写成公开 GEE ImageCollection。
5. 判定时间处理：累计量按月求和，状态量按月平均，极值变量按研究目标选择最大/最小值，分类和地形数据导出一次。静态数据只有在用户明确要求时才复制成月文件。
   - 对 OpenLandMap USDA Soil Texture 的地下水、入渗或根区研究，默认把 `b0,b10,b30,b60,b100,b200` 全部导出为一个六波段静态 GeoTIFF，以保留 0–200 cm 垂向剖面。只有用户明确要求单一代表深度时才选择单层，不得自行只选 `b30` 或 `b200`。
6. 读取 [常用产品指南](references/dataset-guide.md) 选择候选产品；生成代码前读取 [GEE 代码模式](references/gee-code-patterns.md)。
7. 检查代码中的资产路径、结束日期排他性、比例因子、单位换算、波段名称、分类数据重采样、导出尺度、CRS、NoData、文件数和文件名。

## 输出合同

回答按以下顺序给出：

1. **选择结论**：产品名称、版本、选择理由、时间覆盖结论，以及是否覆盖到用户要求的最后一个月。
2. **研究区资产**：从截图识别出的完整资产 ID 和资产类型；截图无法确认类型时明确说明假设。
3. **完整代码**：可直接粘贴到 GEE Code Editor 的 JavaScript；默认每月一个 TIFF，静态或年度数据按其原始时间尺度导出。
4. **产品信息**：数据集 ID、机构、变量/波段、原始与输出单位、比例因子、空间/时间分辨率、聚合方法、质量控制、许可和官方网址。
5. **引用文献**：数据集引用与方法论文分开列出，给出作者、年份、题名、期刊或仓储、DOI 链接。引用必须由官方页面或 DOI 核验。
6. **执行说明**：任务数量、输出目录和命名示例，以及需要用户替换的占位符。

## 约束

- 代码默认不包含 `print()`、日志写入或与请求无关的文件输出。
- 分类变量保持整数编码并采用最近邻重采样；连续变量才使用双线性等连续型重采样。
- 不把粗分辨率产品通过设置更小 `scale` 描述成高分辨率数据；同时报告原生分辨率和导出分辨率。
- 不编造数据集 ID、最后可用月份、单位、DOI 或论文。信息冲突时以产品机构和 GEE 官方目录为主，并指出版本差异。
- 同一产品存在概念 DOI、版本 DOI 或多个 Zenodo 记录时，引用文字必须与实际记录逐项对应；GEE 目录推荐引用作为直接调用版本的首选数据集引用。
- 用户要求的时段超出产品覆盖范围时，给出最接近的合适产品或分段方案，并明确每段覆盖时间。

## 交付检查

| 检查项 | 合格条件 |
|---|---|
| 时间 | 开始月和结束月均有数据；`filterDate` 的结束日期为下一月首日 |
| 变量 | 波段含义、比例因子、聚合和输出单位一致 |
| 空间 | 使用截图中的完整资产路径；原生分辨率表达准确 |
| 代码 | 资产存在性或占位符状态明确，可在 GEE JavaScript 环境运行 |
| 引用 | 产品页、数据 DOI、方法论文均可访问且版本一致 |
