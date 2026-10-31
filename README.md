# GEE Data Downloader

一个面向 Codex 和 Claude Code 的 Agent Skill，用于选择适合研究时段的地理空间产品，识别 Google Earth Engine 资产路径，并生成可复现的 GEE JavaScript 下载代码、产品信息、官方网址和引用文献。

## 功能

- 根据研究区、时间范围、时间尺度、变量和分辨率选择数据产品。
- 核验 GEE 数据集 ID、实际时间覆盖、波段、比例因子、单位和原生分辨率。
- 从 GEE Assets 截图识别 Cloud Assets 或 Legacy Assets 路径。
- 生成月度、年度或静态 GeoTIFF 导出代码。
- 按变量物理意义处理月累计、月平均、分类数据和质量控制。
- 输出产品官网、数据 DOI、方法论文和规范引用。
- 支持 MODIS、CHIRPS、ERA5-Land、GLEAM、CLCD、OpenLandMap、HWSD 等产品的选择流程。

## 兼容性

| 运行环境 | 个人安装位置 | 直接调用 |
|---|---|---|
| Codex | `~/.codex/skills/gee-data-downloader/` | `$gee-data-downloader` |
| Claude Code | `~/.claude/skills/gee-data-downloader/` | `/gee-data-downloader` |

Skill 使用双方共同支持的 `SKILL.md` Agent Skills 结构。`agents/openai.yaml` 提供 Codex 界面元数据；Claude Code 加载 `SKILL.md` 和 `references/` 中的内容。

## 安装

### Codex

```bash
git clone https://github.com/limyuzhang1022-bit/gee-data-downloader.git "$HOME/.codex/skills/gee-data-downloader"
```

安装后可直接提出“调用 GEE”“使用 GEE”或“下载某数据集”等请求，也可以显式调用：

```text
$gee-data-downloader 下载新疆地区 2002-01 至 2025-12 的 CHIRPS 月降水数据，每月一个 TIFF。
```

### Claude Code

个人级安装：

```bash
git clone https://github.com/limyuzhang1022-bit/gee-data-downloader.git "$HOME/.claude/skills/gee-data-downloader"
```

项目级安装：

```bash
git clone https://github.com/limyuzhang1022-bit/gee-data-downloader.git ".claude/skills/gee-data-downloader"
```

显式调用示例：

```text
/gee-data-downloader 下载新疆地区 2002-01 至 2025-12 的 ERA5-Land 月径流数据，并换算为 mm/month。
```

Claude Code 也可以根据 Skill 的 `description` 自动加载。安装位置和调用规则见 [Claude Code Skills 官方文档](https://code.claude.com/docs/zh-CN/skills)。

## 默认研究配置

当前参考配置面向新疆研究区：

```text
projects/underground-water-488604/assets/xj
```

其他用户应在生成代码时提供自己的研究区资产路径，或上传资产截图供 Skill 识别。

## 输出内容

每次调用按以下结构输出：

1. 产品选择结论与完整时间覆盖判断。
2. 研究区资产 ID 和资产类型。
3. 可粘贴到 GEE Code Editor 的完整 JavaScript。
4. 数据集 ID、机构、变量、单位、比例因子、分辨率、聚合方式和质量控制。
5. 产品官网、数据 DOI 和方法论文。
6. 导出任务数量、Google Drive 目录、文件命名和占位符说明。

## 仓库结构

```text
gee-data-downloader/
|-- SKILL.md
|-- README.md
|-- agents/
|   `-- openai.yaml
`-- references/
    |-- dataset-guide.md
    `-- gee-code-patterns.md
```

## 数据与引用原则

Skill 优先核验 [Google Earth Engine Data Catalog](https://developers.google.com/earth-engine/datasets)、产品机构官网、数据 DOI 和原始论文。产品结束日期会按实际可用影像确认，输出分辨率会同时说明原生分辨率和导出网格尺度。
