### 用电跟踪-行业电量跟踪分析
1.功能简介

```
系统根据所选择的查询条件，查询出相应的表格数据，并生成对应的柱状图和曲线图和饼图。
其中，两个时间选择分别为开始时间和结束时间，用以筛选出一段时间范围内的数据；
选择行业默认选中售电公司，默认展示的是售电公司的总体数据。在根据具体需求选择
具体行业之后（选择行业可多选），会查询出所选择行业的具体数据。
```
2.输入

```
查询条件：开始时间、结束时间、行业(多选)
触发条件：【查询按钮】
```

3.输出

```
从行业电量结构表(stat_industry_constitution)根据输入条件查询满足要求的结果数据，显示结果列表。
显示列：本期电量、同期电量、环比增量、整体占比、同比增长率、环比增长率
```

4.存储结构


```
主表：
行业电量结构表：stat_industry_constitution a
关联表：
行业电量结构表：stat_industry_constitution b
行业电量结构表：stat_industry_constitution c

SQL详情：

SELECT
	DATE_FORMAT(
		concat(a. YEAR, "-", a. MONTH, '-01'),
		"%Y-%m"
	) 年月,
	a.industry_name 行业,
	sum(a.electric) 本期电量,
	sum(b.electric) 同期电量,
	sum(c.electric) 上月电量,
	(
		sum(a.electric) - sum(c.electric)
	) 环比增量,
	(
		sum(a.electric) / (
			SELECT
				sum(d.electric)
			FROM
				`stat_industry_constitution` d
			WHERE
				d.`year` = a.`year`
			AND d.`month` = a.`month`
		)
	) 整体占比,
	(
		sum(a.electric) - sum(b.electric)
	) / sum(b.electric) 同比增长率,
	(
		sum(a.electric) - sum(c.electric)
	) / sum(c.electric) 环比增长率
FROM
	`stat_industry_constitution` a
LEFT JOIN `stat_industry_constitution` b ON a.`year` = b.`year` + 1
AND a.`month` = b.`month`
AND a.industry = b.industry
LEFT JOIN `stat_industry_constitution` c ON a.industry = c.industry
AND DATE_FORMAT(
	concat(c. YEAR, "-", c. MONTH, '-01'),
	'%Y-%m'
) = DATE_FORMAT(
	DATE_add(
		concat(a. YEAR, "-", a. MONTH, '-01'),
		INTERVAL - 1 MONTH
	),
	'%Y-%m'
)

GROUP BY
	年月

```

5.特殊说明
```
本期电量：本月电量
同期电量：上一年度的本月电量
**公式：**
环比增量 = 本月电量 - 上一个月的电量
整体占比 = 选择行业的本月电量 / 所有行业的总电量
同比增长率 = （本月电量 - 上一年度的本月电量）/ 上一年度的本月电量
环比增长率 = （本月电量 - 上一个月的电量）/ 上一个月的电量

```