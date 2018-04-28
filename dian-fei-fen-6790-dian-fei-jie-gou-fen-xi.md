### 电费分析-电费结构分析
1.功能简介

```
根据每月的行业电量结构（stat_pricetype_constitution）、行业电量结构（stat_industry_constitution）表，
以图形及表格展示分析数据；
通过两个维度进行分析  
  （1）.行业（stat_industry_constitution）
  （2）.用电类别（stat_pricetype_constitution）
```
2.输入

```
查询条件：  
  （1）分析时间，行业
  （2）分析时间，用电类别
触发条件：进入主页面及查询条件改变
```
3.输出

```
  （1）.行业（散点图、表格）
  （2）.用电类别（散点图、表格）
```
4.数据库关联关系

```
主表：  
  （1）行业电量结构（stat_pricetype_constitution）
  （2）行业电量结构（stat_industry_constitution）
```
5.数据处理过程

```
stat_pricetype_constitution获取用电类别的电费数据
stat_industry_constitution获取行业的电费数据
```

6.SQL预览

用电类别
```
SELECT
	DATE_FORMAT( concat( YEAR, '-', MONTH, '-01' ), '%Y-%m' ) 年月,
	charge 电费,
	industry_name 用电类别 
FROM
	stat_pricetype_constitution
```

行业
```
SELECT
	DATE_FORMAT( concat( YEAR, '-', MONTH, '-01' ), '%Y-%m' ) 年月,
	charge 电费,
	industry_name 行业 
FROM
	stat_industry_constitution
```