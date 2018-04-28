### 用电跟踪-电价分类电量占比

1.功能简介

```

售电量变化情况排名，通过排名看出哪些行业、哪些用户变化大，从而发现总量的变化是由哪些用户或行业的变化造成的。
使用图表展示同比电量、环比电量、环比增量、同比增长率、环比增长率、行业占比等信息。

```

2.输入

```

查询条件：开始年月、结束年月、行业

触发条件：改变查询参数

```

3.输出

```

本期电量、同期电量、环比增量、整体占比、同比增长率、环比增长率

```

4.SQL脚本

```

SELECT t___1.*,
  t___2.sumElectric 本期总电量,
  本期电量/t___2.sumElectric 本期占比
FROM
(SELECT
   t__1.*,
   t__2.year            环比年,
   t__2.month           环比月,
   t__2.electric        环比电量,
   本期电量 - t__2.electric 环比增量,
   FORMAT((本期电量 - t__2.electric)/t__2.electric,2) 环比增长率
 FROM
   (
     SELECT
       t1.year                   本年,
       t1.month                  本月,
       t1.industry_name          行业,
       t1.electric               本期电量,
       t2.year                   同比年,
       t2.month                  同比月,
       t2.electric               同期电量,
       t1.electric - t2.electric 同比增量,
      FORMAT((t1.electric - t2.electric)/t2.electric,2) 同比增长率
     FROM
       stat_pricetype_constitution t1 LEFT JOIN
       stat_pricetype_constitution t2
         ON
           t1.industry_name = t2.industry_name
           AND t1.year - 1 = t2.year
           AND t1.month = t2.month
   ) t__1 LEFT JOIN stat_pricetype_constitution t__2
     ON 行业 = t__2.industry_name
        AND (
          CASE 本月
          WHEN 1
            THEN
              (本年 - 1 = t__2.year AND t__2.month = 12)
          ELSE
            (t__2.month = 本月 - 1 AND t__2.year = 本年)
          END)
)t___1 LEFT JOIN
(
SELECT year,month,sum(electric) sumElectric FROM stat_pricetype_constitution GROUP BY year,month
)t___2 ON 本年 = t___2.year AND 本月 = t___2.month ORDER BY 本年,本月;


```

5.特殊说明

```

本期电量 : 本月电量

同期电量 : 上一年度的本月电量

环比增量 : 本月电量 - 上一个月的电量

整体占比 : 选择行业的本月电量 / 所有行业的总电量

同比增长率 : （本月电量 - 上一年度的本月电量）/ 上一年度的本月电量

环比增长率 : （本月电量 - 上一个月的电量）/ 上一个月的电量

```