### 配变安全-概要设计
1.功能简介

```
所选择的查询条件，查询出相应的表格数据。
其中，选择配变会提供数据库里面存在数据的配变以供选择；
选择日期则会具体到某日。
```
2.输入

```
查询条件：选择变电站，选择日期（具体到日）
触发条件：条件改变自动触发
```

3.输出

```
配变1显示列：配变，日期，时刻点，最大供电能力，最大供电能力利用率，供电能力裕度
配变2显示列：配变，日期，最大负荷，负荷率
```

4.存储结构


```
**主表：**

配变功率数据（96点）表： dms_transformer_power96 a 
关联表：
变压器信息表： dms_transformer b 
96点时刻字典表： base_dic_time96_point c 

**SQL详情：**

-- --配变 1--- START---------------------------------------------------------------------

SELECT 
    a.transformer_code 配变, 
    DATE_FORMAT( 
        a.data_date, 
        "%Y-%m-%d" 
    ) 日期 ,
    c.`name` 时刻点, 
    b.capacity 最大供电能力, 
    a.tp/(b.capacity*a.ft) 最大供电能力利用率, 
    (b.capacity-(a.tp/a.ft))/b.capacity 供电能力裕度
    
FROM 
    dms_transformer_power96 a 
LEFT JOIN 
    dms_transformer b 
ON a.transformer_code = b.`code` 
LEFT JOIN 
    base_dic_time96_point c 
ON a.dataidx = c.`code`
-- --配变 1--- END---------------------------------------------------------------------

-- --配变 2--- START---------------------------------------------------------------------

SELECT 
		a.transformer_code 配变,
		DATE_FORMAT( 
        a.data_date, 
        "%Y-%m-%d" 
    ) 日期 ,
		MAX(a.tp) 最大负荷,
    AVG(a.tp)/MAX(a.tp) 负荷率
    
FROM 
    dms_transformer_power96 a 
LEFT JOIN 
    dms_transformer b 
ON a.transformer_code = b.`code` 
LEFT JOIN 
    base_dic_time96_point c 
ON a.dataidx = c.`code`
GROUP BY a.transformer_code,日期

-- --配变 2--- END---------------------------------------------------------------------

```

5.特殊说明
```
**变量：**

配变额定容量(dms_transformer.capacity)
最大允许负荷率（当前我们看作为1，即默认值为1，数据库无该字段）
负荷(dms_transformer_power96.tp)
配变额定容量(dms_transformer.capacity)
功率因数(dms_transformer_power96.ft)

**公式：**

最大供电能力 = 配变额定容量*最大允许负荷率（当前我们看作为1，即默认值为1，数据库无该字段）;
供电能力裕度 =（最大供电能力-负荷/功率因数(cosφ)）/最大供电能力;

功率因数=有功功率/sqrt（有功功率^2 + 无功功率^2）;

最大供电能力利用率=负荷/（最大供电能力* 功率因数cosφ）

```