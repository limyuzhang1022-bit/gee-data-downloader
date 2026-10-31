# GEE 代码模式

生成代码前，用官方目录替换模板中的数据集 ID、波段、比例因子、单位、尺度和质量控制。所有日期范围采用左闭右开。

## 月度动态变量：每月一个 TIFF

```javascript
var roiFc = ee.FeatureCollection(
  'projects/underground-water-488604/assets/xj'
);
var roi = roiFc.geometry();

var source = ee.ImageCollection('DATASET_ID')
  .select('SOURCE_BAND')
  .filterDate('2002-01-01', '2026-01-01');

for (var year = 2002; year <= 2025; year++) {
  for (var month = 1; month <= 12; month++) {
    var start = ee.Date.fromYMD(year, month, 1);
    var end = start.advance(1, 'month');

    // 累计量用 sum()；状态量用 mean()。
    var monthly = source
      .filterDate(start, end)
      .sum()
      .multiply(SCALE_FACTOR)
      .add(OFFSET)
      .rename('OUTPUT_BAND')
      .clip(roi)
      .toFloat();

    var mm = ('0' + month).slice(-2);
    var name = 'PRODUCT_' + year + '_' + mm;

    Export.image.toDrive({
      image: monthly,
      description: name,
      folder: 'DRIVE_FOLDER',
      fileNamePrefix: name,
      region: roi,
      scale: NATIVE_SCALE_METERS,
      crs: 'EPSG:4326',
      maxPixels: 1e13,
      fileFormat: 'GeoTIFF',
      formatOptions: {cloudOptimized: true}
    });
  }
}
```

若源产品已经是月累计或月平均影像，每个月使用 `.first()` 选取对应影像，并保留产品定义的单位换算。代码交付前确认该月只有一幅目标影像。

## 静态多波段变量：导出一次

```javascript
var roiFc = ee.FeatureCollection(
  'projects/underground-water-488604/assets/xj'
);
var roi = roiFc.geometry();

var image = ee.Image('DATASET_ID')
  .select(
    ['SOURCE_BAND_1', 'SOURCE_BAND_2'],
    ['OUTPUT_BAND_1', 'OUTPUT_BAND_2']
  )
  .clip(roi);

Export.image.toDrive({
  image: image,
  description: 'PRODUCT_STATIC',
  folder: 'DRIVE_FOLDER',
  fileNamePrefix: 'PRODUCT_STATIC',
  region: roi,
  scale: NATIVE_SCALE_METERS,
  crs: 'EPSG:4326',
  maxPixels: 1e13,
  fileFormat: 'GeoTIFF',
  formatOptions: {cloudOptimized: true}
});
```

分类数据保留整数类型和原始类别编码。不要对土地覆盖、土壤质地等类别数据使用双线性重采样。

## 年度变量：每年一个 TIFF

```javascript
var roiFc = ee.FeatureCollection(
  'projects/underground-water-488604/assets/xj'
);
var roi = roiFc.geometry();

for (var year = START_YEAR; year <= END_YEAR; year++) {
  var image = ee.Image('YEARLY_ASSET_PREFIX_' + year)
    .select('SOURCE_BAND')
    .rename('OUTPUT_BAND')
    .clip(roi);

  var name = 'PRODUCT_' + year;
  Export.image.toDrive({
    image: image,
    description: name,
    folder: 'DRIVE_FOLDER',
    fileNamePrefix: name,
    region: roi,
    scale: NATIVE_SCALE_METERS,
    crs: 'EPSG:4326',
    maxPixels: 1e13,
    fileFormat: 'GeoTIFF'
  });
}
```

## 外部产品上传后的调用

```javascript
// 该集合必须由用户先上传；用实际资产 ID 替换占位符。
var uploaded = ee.ImageCollection(
  'projects/PROJECT_ID/assets/UPLOADED_COLLECTION'
).filterDate('2002-01-01', '2026-01-01');
```

回答中必须把前置步骤和占位符列出来。若用户要求“只用 GEE 公共数据”，应选择功能上最接近且时间覆盖完整的公共产品，并说明变量定义差异。

## 质量控制与单位

- 比例因子和偏移量在聚合前还是聚合后应用，按产品定义决定。线性比例因子对均值可前后等价；偏移量不得在求和后只加一次。
- 降水、径流和蒸散发等通量累计值通常按月求和；温度、土壤湿度、NDVI、LAI 和雪盖百分比通常按月平均。
- 对 QC 波段构造掩膜后再聚合。代码注释写明保留哪些质量等级。
- NoData 仅在用户需要固定值时使用 `unmask()`；同时在产品信息中声明该值。
- `scale` 默认使用原生尺度。需要统一网格时说明重投影只统一像元网格，不提升有效空间分辨率。
