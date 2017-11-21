---
title: Superset 分析示例数据
date: 2017-11-22 23:51:00
description: Superset 理解示例中的分片
tags:
- Superset
categories:
copyright: false
---

`superset` 默认提供 `main` 数据库（`sqlite db`），这个数据库存放整个 `superset` 系统的用户、角色、权限、数据集等数据，还存放预备的示例数据；

在数据表栏，默认暴露的数据表有 `multiformat_time_series`、 `birth_france_by_region`、 `long_lat`、 `random_time_series`、 `birth_names`、 `wb_health_population`、 `energy_usage` 六个；只有管理员才有权利暴露数据库中的哪些数据表可以在 `superset`中作图表展示；

切片是每一个要展示图表的最小操作单元，基于暴露的数据表作分析；目前直接在切片页面进行创建，只支持单表；但是在 `SQL 工具箱`自己写 `sql` 来创建分片是支持多表关联；

一系列切片可以放在一个看板（也叫 仪表盘）集中展示；

下面开始分析各个切片；

# 基于 `multiformat_time_series` 数据表的例子
`multiformat_time_series` 表记录时间的各种形式；

| 列名 | 类型| 描述 | 
|: --- :|: --- :|: --- :|
| ds | date | 记录创建时间 （只有日期）|
| ds2 | datetime | 记录创建时间 （日期加时间）|
| epoch_ms | bigint | 记录时间时的毫秒|
| epoch_s | bigint | 记录时间时的秒 |
| string0 | varchar(100) |记录时间，格式如 `2017-07-19 06:23:33.000000` |
| string1 | varchar(100) | 记录时间，格式如 `2017-07-19^06:23:33` |
| string2 | varchar(100) | 记录时间，格式如 `20170719-062333` |
| string3 | varchar(100) | 记录时间，格式如 `2017/07/1906:23:33.000000` |

## 切片
### `Calendar Heatmap multiformat 0`
图表类型：时间热力图
图表意义：类似于 `github` 上的打卡记录；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `ds` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `1 years ago`， `Until` 选择 `now` 
2. `Query` 列
  度量选择 `count(*)`；
3. `Options` 列
  `Domain` 选择 `month`， `subdomain` 选择 `day`；

# 基于 `birth_france_by_region` 数据表的例子
`birth_france_by_region` 表记录法国各城市在历年的人口出生数量；

| 列名 | 类型| 描述 | 
|: --- :|: --- :|: --- :|
| DEPT_ID | varchar(10) | 法国城市代号 |
| ... | bigint | 2003 年 2014 年的数据 |
| date | date | 记录的创建时间 |

## 切片
### `Birth in France by department in 2016`
图表类型：国家地图
图表意义：直接在国家地图上展现人口数量；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `无穷`， `Until` 选择 `无穷` 
2. `Query` 列
  `ISO 3166-2` 选择 `DEPT_ID`，  度量选择 `avg_2004`；
3. `Options` 列
  `Country Name` 选择 `France`；


# 基于 `random_time_series` 数据表的例子
`random_time_series` 表只有一列，记录时间；

| 列名 | 类型| 描述 | 
|: --- :|: --- :|: --- :|
| ds | datetime | 时间 |

## 切片
### `Calendar Heatmap`
图表类型：时间热力图
图表意义：类似于 `github` 上的打卡记录；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `1 years ago`， `Until` 选择 `now` 
2. `Query` 列
  度量选择 `count(*)`；
3. `Options` 列
  `Domain` 选择 `month`， `subdomain` 选择 `day`；

# 基于 `birth_names` 数据表的例子

`birth_names` 表记录一些学生的姓名、姓名、居住地、出生日期等信息；

| 列名 | 类型| 描述 | 
|: --- :|: --- :|: --- :|
| ds | datetime | 出生日期 |
| gender | varchar(16)  | 性别 |
| name | varchar(255) | 姓名 |
| num | bigint | 数量 |
| state | varchar(10) | 所在州 |
| sum_boys | bigint | 相同姓名男生的数量 |
| sum_girls | bigint | 相同姓名女生的数量 |


## 切片
### `Number of Girls`
图表类型：数字
图表意义：直接显示女生数量；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `100 years ago`， `Until` 选择 `now` 
2. `Query` 列
  度量选择 `sum__num`；
3. `Chart Options` 列
  `Subheader` 选择 `total female participants`
4. 筛选列
  `gender`  `in` `girl`

### `Pivot Table`
图表类型：透视表
图表意义：二维表展示各州各个姓名的人数；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `100 years ago`， `Until` 选择 `now` 
2. `Query` 列
  `Group by` 选择 `name`， `columns` 选择 `state`， `Metrics` 选择 `sum__num`
3. `Pivot Options` 列
  `Aggregation function` 选择 `sum`

### `Name Cloud`
图表类型：词汇云
图表意义：优雅地展示姓名，字体大小反映使用次数的多寡；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `100 years ago`， `Until` 选择 `now` 
2. `Query` 列
  `Series` 使用 `name`， 度量使用 `sum__num`， `Series limit` 使用 100
3. `Options` 列
  `Font Size From` 使用 10， `Font Size To` 使用 70， `Rotation` 使用 `square`；

### `Title`
图表类型：标记
图表意义：可以显示自定义的 `html` 代码；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `100 years ago`， `Until` 选择 `now` 
2. `Code`
  `Markup Type` 选择 `html`
  `Code` 选择

    ```
  <div style="text-align:center">
    <h1>Birth Names Dashboard</h1>
    <p>
        The source dataset came from
        <a href="https://github.com/hadley/babynames" target="_blank">[here]</a>
    </p>
    <img src="/static/assets/images/babytux.jpg">
 </div>
    ```



### `Average and Sum Trends`
图表类型：`Dual Axis Line Chart`
图表意义：反映姓名使用率的平均值与总值随着时间的变化；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `100 years ago`， `Until` 选择 `now` 
2. `Y Axis 1` 列
  `Left Axis Metrics` 选择 `avg__num`；
2. `Y Axis 2` 列
  `Right Axis Metrics` 选择 `sum__num`；

### `Trends`
图表类型：时间序列 - 折线图
图表意义：反映姓名随着时间的使用率；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `100 years ago`， `Until` 选择 `now` 
2. `Query` 列
  `Metrics` 选择 `sum__num`， `Group By` 选择 `name`， `Series limit` 选择 25 （显示多少组数据）；



### `Genders by State`
图表类型：分布柱状图
图表意义：统计各州的男女人数；

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `ds` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `100 years ago`， `Until` 选择 `now` 
2. `Query` 列
    `Metrics` 依次选择 `sum__sum_girls`、`sum__sum_boys`，`Series` 选择 `state`， `Row limit` 选择 50000;
3. 筛选列
  `state`  `not in` `other`


### `Genders`
图表类型：Pie Chart
图表意义：饼图直接反映男女人数的比例大小；

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `ds` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `100 years ago`， `Until` 选择 `now` 
2. `Query` 列
    `Metrics` 选择 `sum__num`，`Group By` 选择 `gender`;


### `Participants`
图表类型：数字和趋势线
图表意义：直观地反映每年参与的人数走势；

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `ds` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `100 years ago`， `Until` 选择 `now` 
2. `Query` 列
    度量选择 `sum__num`

### `Boys`
图表类型：表视图
图表意义：表格形式展现男生姓名及人数；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `ds` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `100 years ago`， `Until` 选择 `now` 
2. `GROUP BY` 列
  `Group By` 选择 `name`， `Metrics` 选择 `sum__num`
3. `Options` 列
  `Row limit` 选择 `50000`， `Page Length` 选择 `0`
4. 筛选列
  `gender` `in` `boy` 

### `Girls`
图表类型：表视图
图表意义：表格形式展现女生姓名及人数；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `ds` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `100 years ago`， `Until` 选择 `now` 
2. `GROUP BY` 列
  `Group By` 选择 `name`， `Metrics` 选择 `sum__num`
3. `Options` 列
  `Row limit` 选择 `50000`， `Page Length` 选择 `0`
4. 筛选列
  `gender` `in` `girl` 

# 基于 `wb_health_population` 数据表的例子

`wb_health_population` 表记录不同地域不同国家历年来的人口数量；

| 列名 | 类型| 描述 | 
|: --- :|: --- :|: --- :|
| country_code | varchar(3) |  国家代号|
| country_name | varchar(255) | 国家名 |
| region | varchar(255) | 地区 |
| year | datetime  | 年份 |
| ... | float | 该国家在该年份的统计数据  |

## 切片
### `Parallel Coordinates`
图表类型：平行坐标
图表意义：未知；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `2011-01-01 00:00:00`， `Until` 选择 `2011-01-01 00:00:00` 
2. `Query` 列
  ` Series` 选择 `country_name`， `Metrics` 依次选择 `sum__SP_POP_TOTL` 、`sum__SP_RUR_TOTL_ZS` 和 `sum__SH_DYN_AIDS`；


### `Treemap`
图表类型：树状图
图表意义：根据国家所占长方形面积的大小，直观看出各国家总人口占世界总人口的比例；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `1960-01-01 00:00:00`， `Until` 选择 `now` 
2. `Query` 列
  `Metrics` 选择 `sum__SP_POP_TOTAL`， `Group By` 选择 `region` 和 `country_name`；也就是以地区和国家分组，统计总人数；


### `Box plot`
图表类型：箱线图
图表意义：粗略地看出每年地区的人口数是否具有有对称性，分布的分散程度等信息；

> 箱线图是用数据中的五个统计量：最小值、第一四分位数（Q1）、中位数（Q2）、第三四分位数（Q3）与最大值来描述数据的一种方法

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `1960-01-01 00:00:00`， `Until` 选择 `now` 
2. `Query` 列
  `Metrics` 选择 `sum__SP_POP_TOTAL`， `Group By` 选择 `region`， `Series limit` 选择 25 （显示多少组数据）；

### `World's Pop Growth`
图表类型：时间序列-堆积图
图表意义：以年为维度，体现世界总人口的变化，以及每个区域人口在任意一个时间大概的占比；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `1960-01-01 00:00:00`， `Until` 选择 `now` 
2. `Query` 列
  `Metrics` 选择 `sum__SP_POP_TOTAL`， `Group By` 选择 `region`， `Series limit` 选择 25 （显示多少组数据）；

### `Rural Breakdown`
图表类型：环状层次图
图表意义：查看某一年，国家人口占世界人口的百分比，国家人口占地域人口的百分比，国家农村人口占国家总人口的百分比；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `2011-01-01 00:00:00`， `Until` 选择 `2011-01-01 00:00:00` 
2. `Query` 列
  `Hierarchy` 依次选择 `region`、`country_name`,   `Primary Metric` 选择 `sum__SP_POP_TOTAL`，  `Secondary Metric` 选择 `sum__SP_RUR_TOTAL`， `Row limit` 选择 50000 （数据记录 `Limit` 值）；

### `Life Expectancy VS Rural %`
图表类型：气泡图
图表意义：未知；
国表特点：根据国家的两个数据作 `x轴` 和 `y轴` 的定位，气泡大小取决于给定的 `sum__SP_POP_TOTL` 值

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `2011-01-01 00:00:00`， `Until` 选择 `2011-01-02 00:00:00` 
2. `Query` 列
   `Series` 选择 `region`， `Entity` 选择 `country_name`， `Bubble Size` 选择 `sum_SP_POP_TOTL`， `Series limit`选择 0（列出所有组数据）
3. 筛选列
  `country_code` `not in`  [ `TCA`、 `MNP`、 `DMA`、 `MHL`、`MCO`、`SXM`、`CYM`、 `TUV`、`IMY`、`KNA`、`ASM`、`ADO`、`AMA`、`PLW` ]
4. `Bubbles` 列
  `Bubbles Size` 选择 `sum__SP_POP_TOTL`， `Max Bubble Size` 选择 25
5. `X Axis` 列
  `X Axis` 选择 `sum__SP_RUR_TOTL_ZS`
6. `Y Axis` 列
   `Y Axis` 选择 `sum__SP_DYN_LE00_IN`

### `% Rural`
图表类型：世界地图
图表意义：以世界地图的方式展现国家总人口和农村人口比例；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `2014-01-01 00:00:00`， `Until` 选择 `2014-01-02 00:00:00` 
2. `Query` 列
   `Country Control` 选择 `country_code`， `Country Field Type` 选择 `code ISO 3166-1 alpha-3(cca3)`， `Metric for color` 选择 `sum_SP_RUR_TOTL_ZS`；
3. `Bubbles` 列
  `Bubbles Size` 选择 `sum__SP_POP_TOTL`， `Max Bubble Size` 选择 25； 气泡显示国家总人口；


### `Growth Rate`
图表类型：时间序列-折线图
图表意义：折线的方式体现各个国家人口增长的走势；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `1960-01-01 00:00:00`， `Until` 选择 `2014-01-02 00:00:00` 
2. `Query` 列
  `Metrics` 选择 `sum__SP_POP_TOTAL`， `Group By` 选择 `country_name`， `Series limit` 选择 25 （显示多少组数据）；
3. `Advanced Analytics` 列
  `Peroid Ratio` 选择 `10` (意义还不清楚)；

### `Most Populated Countries`
图表类型：表视图
图表意义：表格形式展现国家人口；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `2000-01-01 00:00:00`， `Until` 选择 `2014-01-02 00:00:00` 
2. `GROUP BY` 列
  `Group By` 选择 `country_name`， `Metrics` 选择 `sum__SP_POP_TOTAL`
3. `Options` 列
  `Row limit` 选择 `50000`， `Page Length` 选择 `0`

### `World's Population`
图表类型：数字和趋势线
图表意义：暴力展示世界总人口的走势和数字；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `2014-01-01 00:00:00`， `Until` 选择 `2014-01-02 00:00:00` 
2. `Query` 列
  度量选择 `sum__SP_POP_TOTAL`
3. `Chart Options` 列
  `Comparison Period Lag` 选择 `1`， `Comparison suffix` 选择 `年增长率`

### `Region Filter`
图表类型：`Filter Box`
图表显示：一个输入框，列出地域名（阴影表示人口数）或国家名（阴影表示人口数）；

复盘：

1. `Time` 列
    对指定的时间列作筛选，时间字段选择 `year` ， `Time Grain` 时间间隔选择 `Time Column`值（也就是每一个记录值都是一个时间节点）， `Since` 选择 `2014-01-01 00:00:00`， `Until` 选择 `2014-01-02 00:00:00` 
2. `Query` 列
  `Filter controls` 选择 `region` 和 `country_name`； 度量选择 `sum_SP_POP_TOTAL`


# 基于 `energy_usage` 数据表的例子

`energy_usage` 表记录能源的流通；

| 列名 | 类型| 描述 | 
|: --- :|: --- :|: --- :|
| source | varchar(255) | 能源的来源|
| target | varchar(255) | 能源周转后的形态 |
| value |  float | 值 |

## 切片
### `Heatmap`
图表类型：热力图
图表意义：看出能源互转的热度与比例；

复盘：

1. `Query` 列
 `X` 轴选择 `source`，`Y` 轴选择 `target`， 度量选择 `sum__value`

### `Energy Force Layout`
图表类型：有向图
图表意义： 直观地看出能源互转

复盘：

1. `Query` 列
  `Source / Target` 依次选择 `source` 和 `target`， 度量选择 `sum__value`，
  `Row limit` 选择 5000 （表示读取多少数据作分析）



### `Energy Sankey`
图表类型：蛇形图
图表意义： 更好地体现能源周转的值与比例

复盘：

1. `Query` 列
  `Source / Target` 依次选择 `source` 和 `target`， 度量选择 `sum__value`，
  `Row limit` 选择 5000 （表示读取多少数据作分析）
