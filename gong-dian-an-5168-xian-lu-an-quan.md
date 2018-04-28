### 线路安全-概要设计
1.功能简介

```
所选择的查询条件，查询出相应的表格数据。
其中，选择线路会提供数据库里面存在数据的线路以供选择；
选择日期则会具体到某日。
```
2.输入

```
查询条件：选择线路，选择日期（具体到日）
触发条件：条件改变自动触发
```

3.输出

```
线路表格1显示列：配变，日期，时刻点，最大供电能力，最大供电能力利用率，供电能力裕度
线路表格2显示列：配变，日期，最大负荷，负荷率
```

4.存储结构


```
**主表：**

线路功率数据（96点）： dms_line_power96 a 
关联表：
线路表： dms_line b 
96点时刻字典表： base_dic_time96_point c 

**SQL详情：**

-- --线路 1--- START---------------------------------------------------------------------

SELECT 
    a.line_code 线路, 
		DATE_FORMAT( 
        a.data_date, 
        "%Y-%m-%d" 
    ) 日期 ,
    c.`name` 时刻点, 
    b.capacity 最大供电能力, 
    a.tp/(b.capacity*a.ft) 最大供电能力利用率, 
    (b.capacity-(a.tp/a.ft))/b.capacity 供电能力裕度
    
FROM 
    dms_line_power96 a 
LEFT JOIN 
    dms_line b 
ON a.line_code = b.`code` 
LEFT JOIN 
    base_dic_time96_point c 
ON a.dataidx = c.`code`
-- --线路 1--- END---------------------------------------------------------------------

-- --线路 2--- START---------------------------------------------------------------------

SELECT 
		a.line_code 线路,
		DATE_FORMAT( 
        a.data_date, 
        "%Y-%m-%d" 
    ) 日期 ,
		MAX(a.tp) 最大负荷,
    AVG(a.tp)/MAX(a.tp) 负荷率
    
FROM 
    dms_line_power96 a 
LEFT JOIN 
    dms_line b 
ON a.line_code = b.`code` 
LEFT JOIN 
    base_dic_time96_point c 
ON a.dataidx = c.`code`
GROUP BY a.line_code,日期

-- --线路 2--- END---------------------------------------------------------------------

```

5.特殊说明
```
**变量：**

线路额定容量(dms_line.capacity)
最大允许负荷率（当前我们看作为1，即默认值为1，数据库无该字段）
负荷(dms_line_power96.tp)
功率因数(dms_line_power96.ft)

**公式：**

最大供电能力 = 线路额定容量*最大允许负荷率（当前我们看作为1，即默认值为1，数据库无该字段）;
供电能力裕度 =（最大供电能力-负荷/功率因数(cosφ)）/最大供电能力;

功率因数=有功功率/sqrt（有功功率^2 + 无功功率^2）;

最大供电能力利用率=负荷/（最大供电能力* 功率因数cosφ）

```