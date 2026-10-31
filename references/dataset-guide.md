# 常用产品指南

这里的条目是候选产品起点。每次使用时仍需通过官方页面核验版本、最后可用日期、波段和引用。

## 当前研究配置

- 研究区：新疆
- 已确认资产：`projects/underground-water-488604/assets/xj`
- 常用研究时段：2002-01 至 2025-12
- 月度动态变量：每月一个 GeoTIFF
- 年度或静态变量：保持产品原始时间尺度导出

## 候选产品

| 变量 | 首选候选产品 | GEE ID / 访问方式 | 原生分辨率 | 月尺度处理 | 关键注意事项 |
|---|---|---|---:|---|---|
| 地表温度 | MODIS Terra LST 8-day L3 v061 | `MODIS/061/MOD11A2` | 1 km | 有效合成值月平均 | `LST_Day_1km`/`LST_Night_1km` 乘 0.02 K；使用 QC 波段 |
| NDVI/EVI | MODIS Terra Vegetation Indices v061 | `MODIS/061/MOD13A2` | 1 km | 月平均 | NDVI/EVI 乘 0.0001；使用 `SummaryQA` 或 `DetailedQA` |
| LAI | MODIS MCD15A3H v061 | `MODIS/061/MCD15A3H` | 500 m | 月平均 | `Lai` 乘 0.1；使用 `FparLai_QC` |
| 积雪覆盖度 | MODIS Terra Snow Cover Daily v061 | `MODIS/061/MOD10A1` | 500 m | `NDSI_Snow_Cover` 月平均 | 输出可解释为月平均雪盖百分比；先屏蔽无效类别值 |
| 降水 | CHIRPS v2 Daily | `UCSB-CHG/CHIRPS/DAILY` | 0.05° | 日降水月求和 | 原始单位通常为 mm/day，月累计为 mm/month |
| 径流 | ERA5-Land Monthly Aggregated | `ECMWF/ERA5_LAND/MONTHLY_AGGR` | 约 11 km | 直接选月累计波段 | `runoff_sum` 原始单位 m，乘 1000 得 mm/month；核验最后可用月 |
| 温度/降水等 | ERA5-Land Monthly Aggregated | `ECMWF/ERA5_LAND/MONTHLY_AGGR` | 约 11 km | 状态量取月均，累计量用 `_sum` | K 转摄氏度需减 273.15；水深 m 转 mm 乘 1000 |
| DEM/坡度/坡向 | SRTM v3 | `USGS/SRTMGL1_003` | 30 m | 静态导出一次 | 坡度/坡向由 `ee.Terrain.products` 计算 |
| 土地覆盖 | CLCD | 外部下载并上传 GEE Assets | 30 m | 按年度导出 | 中国年度产品；不得虚构公共 GEE ID |
| 蒸散发与根区土壤水分 | GLEAM4 | 官网/SFTP 下载，转换后上传 | 0.1° | E/Ep/Eb/Es 月累计；SMrz 月平均 | 选择覆盖全期的单一一致版本；只有单一版本确实无法覆盖且用户接受方法差异时才分段；不得写成公开 GEE 集合 |
| 土壤质地 | OpenLandMap USDA Soil Texture v02 | `OpenLandMap/SOL/SOL_TEXTURE-CLASS_USDA-TT_M/v02` | 250 m | 静态导出一次 | 地下水、入渗或根区研究默认将 `b0,b10,b30,b60,b100,b200` 全部导出为一个六波段文件；仅在用户明确只需代表层时选单层；1–12 为类别编码 |
| 全球土壤类型/属性 | HWSD v2 | FAO 下载后处理/上传 | 约 1 km | 静态导出一次 | FAO/WRB 土壤单元及七深度属性；用途不同于 USDA 质地类别 |

## 已知引用起点

引用前核对产品版本和官方推荐格式。

- CHIRPS：Funk, C. et al. (2015). *The climate hazards infrared precipitation with stations: a new environmental record for monitoring extremes*. Scientific Data, 2, 150066. https://doi.org/10.1038/sdata.2015.66
- ERA5-Land：Muñoz-Sabater, J. et al. (2021). *ERA5-Land: a state-of-the-art global reanalysis dataset for land applications*. Earth System Science Data, 13, 4349–4383. https://doi.org/10.5194/essd-13-4349-2021
- CLCD：Yang, J. and Huang, X. (2021). *The 30 m annual land cover dataset and its dynamics in China from 1990 to 2019*. Earth System Science Data, 13, 3907–3925. https://doi.org/10.5194/essd-13-3907-2021
- OpenLandMap 土壤质地：GEE 官方目录推荐引用 Hengl, T. (2018). *Soil texture classes (USDA system) for 6 soil depths at 250 m, Version v02*. Zenodo. https://doi.org/10.5281/zenodo.1475451。Zenodo `2525817` 是另一个产品记录页，只有在实际依据该记录时才按其页面元数据另行引用，不要把两个 DOI 标成同一记录的概念 DOI和版本 DOI。
- SoilGrids 方法：Hengl, T. et al. (2017). *SoilGrids250m: Global gridded soil information based on machine learning*. PLOS ONE, 12, e0169748. https://doi.org/10.1371/journal.pone.0169748
- GLEAM 方法：Miralles, D. G. et al. (2011). *Global land-surface evaporation estimated from satellite-based observations*. HESS, 15, 453–469. https://doi.org/10.5194/hess-15-453-2011
- SRTM：Farr, T. G. et al. (2007). *The Shuttle Radar Topography Mission*. Reviews of Geophysics, 45, RG2004. https://doi.org/10.1029/2005RG000183
- HWSD：引用 FAO/IIASA/ISRIC/ISSCAS/JRC 发布的对应版本技术报告；v2.x 与 v1.2 的层数和属性结构不同。

## 选择规则

1. 时间完整性优先于“最新版本”标签。产品缺少研究开始年或结束月时，选择覆盖完整时段的稳定版本。
2. GEE 目录产品优先用于直接导出；外部产品只有在变量不可替代或用户明确指定时采用上传流程。
3. 分辨率按原生像元报告。导出到 250 m、30 m 等更小像元只改变采样网格，不增加信息量。
4. 静态分类变量按类别输入模型；月度连续变量按物理量选择求和或平均。
5. 地下水长期趋势研究优先采用全时段处理链一致的产品版本。跨版本拼接必须说明切换日期、强迫数据或算法差异，并建议敏感性检验。
