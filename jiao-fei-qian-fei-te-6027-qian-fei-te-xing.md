### 缴费欠费特性-欠费特性
1.功能简介

```
描述：  
    欠费金额比例、欠费户数比例、欠费用户分布
分析目标：  
    欠费金额排名、欠费时间排名，通过欠费用户分析可发现哪些用户喜欢欠费
实现：  
    根据缴费表（cust_payment_detail）、用户月度电费表（cust_electr_charge_monthly）连接查询
以图形及表格展示分析数据；
```
2.输入

```
查询条件：  
  分析时间、行业
触发条件：进入主页面及查询条件改变
```
3.输出

```
  图形：  
  （1）.欠费数量（饼图）
  （2）.欠费金额（饼图）
  （3）.数据（表格）
  属性：  
    年月、行业、欠费用户数比例、欠费金额比例
```
4.数据库关联关系

```
主表：  
   用户月度电费表（cust_electr_charge_monthly）
从表：
  缴费表（cust_payment_detail）
```
5.数据处理过程

```
以用户月度电费表（cust_electr_charge_monthly）为主表获取应交电费数据，左连接缴费表（cust_payment_detail）获取实收电费数据
```
6.SQL预览

```
SELECT
	a.`年月`,
	a.`行业`,
	( a.`欠费金额` / b.`总欠费金额` * 100 ) 欠费金额比例,
	( a.`数量` / b.`总数量` * 100 ) 欠费用户数比例 
FROM
	(
SELECT
	a.年月,
	b.行业,
	count( * ) 数量,
	sum( a.欠费金额 ) 欠费金额 
FROM
	(
SELECT
	a.cust_code,
	DATE_FORMAT( concat( YEAR, '-', MONTH, '-01' ), '%Y-%m' ) 年月,
	( a.charge - ( CASE WHEN b.pay_amount IS NULL THEN 0 ELSE b.pay_amount END ) ) 欠费金额 
FROM
	cust_electr_charge_monthly a
	LEFT JOIN ( SELECT cust_code, DATE_FORMAT( pay_date, '%Y-%m' ) ym, pay_amount FROM cust_payment_detail ) b ON a.cust_code = b.cust_code 
	AND DATE_FORMAT( concat( a.YEAR, '-', a.MONTH, '-01' ), '%Y-%m' ) = b.ym 
WHERE
	( a.charge - ( CASE WHEN b.pay_amount IS NULL THEN 0 ELSE b.pay_amount END ) ) > 0 
	) a
	LEFT JOIN ( SELECT a.`code` cust_code, b.aliasname 行业 FROM cust_customer a LEFT JOIN base_dic_industry b ON a.industry = b.id ) b ON a.cust_code = b.cust_code 
GROUP BY
	a.`年月`,
	b.`行业` 
	) a
	LEFT JOIN (
SELECT
	a.年月,
	count( * ) 总数量,
	sum( a.欠费金额 ) 总欠费金额 
FROM
	(
SELECT
	a.cust_code,
	DATE_FORMAT( concat( YEAR, '-', MONTH, '-01' ), '%Y-%m' ) 年月,
	( a.charge - ( CASE WHEN b.pay_amount IS NULL THEN 0 ELSE b.pay_amount END ) ) 欠费金额 
FROM
	cust_electr_charge_monthly a
	LEFT JOIN ( SELECT cust_code, DATE_FORMAT( pay_date, '%Y-%m' ) ym, pay_amount FROM cust_payment_detail ) b ON a.cust_code = b.cust_code 
	AND DATE_FORMAT( concat( a.YEAR, '-', a.MONTH, '-01' ), '%Y-%m' ) = b.ym 
WHERE
	( a.charge - ( CASE WHEN b.pay_amount IS NULL THEN 0 ELSE b.pay_amount END ) ) > 0 
	) a 
GROUP BY
	a.`年月` 
	) b ON a.年月 = b.年月
```
