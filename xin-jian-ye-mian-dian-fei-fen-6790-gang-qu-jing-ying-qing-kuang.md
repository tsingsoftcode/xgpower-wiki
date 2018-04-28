### 电费分析-港区经营情况
1.功能简介

```
根据每月的电费数据（缴费表cust_payment_detail，用户月度电费表cust_electr_charge_monthly），  
以图形及表格展示分析数据；
分析数据包括收费户数、应收电费、实收电费、欠费金额、电费回收率、欠费户数
```
2.输入

```
查询条件：时间，行业
触发条件：进入主页面及查询条件改变
```
3.输出

```
收费户数、应收电费、实收电费、欠费金额、电费回收率、欠费户数
```
4.数据库关联关系

```
主表：用户月度电费表cust_electr_charge_monthly
关联表：缴费表cust_payment_detail
```
5.数据处理过程

```
cust_electr_charge_monthly表获取应收电费，用户列表  
cust_payment_detail表获取实收电费  
总户数 = sum(cust_electr_charge_monthly)
收费户数 = sum(cust_payment_detail where pay_date is not null)    
欠费户数 = 总户数 - 收费户数
欠费金额 = 应收电费 - 实收电费  
电费回收率 = 实收电费/应收电费
```

6.SQL预览

```
SELECT
	DATE_FORMAT( concat( YEAR, '-', MONTH, '-01' ), '%Y-%m' ) 年月,
	c.`实收户数` 收费户数,
	(c.应收户数 - c.`实收户数`) 欠费户数,
	sum( b.charge ) 应收电费,
	a.实收电费,
	( sum( b.charge ) - ( CASE WHEN a.实收电费 IS NULL THEN 0 ELSE a.实收电费 END ) ) 欠费金额,
	(a.实收电费/sum( b.charge )*100) 电费回收率
FROM
	cust_electr_charge_monthly b
	LEFT JOIN (
SELECT
	DATE_FORMAT( pay_date, '%Y-%m' ) 年月,
	sum( a.pay_amount ) 实收电费
	
FROM
	cust_payment_detail a 
GROUP BY
	DATE_FORMAT( pay_date, '%Y-%m' ) 
	) a ON a.年月 = DATE_FORMAT( concat( YEAR, '-', MONTH, '-01' ), '%Y-%m' )
	LEFT JOIN (
	SELECT
		DATE_FORMAT( concat( YEAR, '-', MONTH, '-01' ), '%Y-%m' ) 年月,
		count( * ) 应收户数,
		b.`实收户数` 
	FROM
		cust_electr_charge_monthly a
		LEFT JOIN (
		SELECT
			DATE_FORMAT( pay_date, '%Y-%m' ) ym,
			count( * ) 实收户数 
		FROM
			cust_payment_detail 
		WHERE
			pay_date IS NOT NULL 
		GROUP BY
			DATE_FORMAT( pay_date, '%Y-%m' ) 
		) b ON DATE_FORMAT( concat( YEAR, '-', MONTH, '-01' ), '%Y-%m' ) = b.ym 
	GROUP BY
		DATE_FORMAT( concat( YEAR, '-', MONTH, '-01' ), '%Y-%m' ) 
	) c ON DATE_FORMAT( concat( YEAR, '-', MONTH, '-01' ), '%Y-%m' ) = c.年月 
GROUP BY
DATE_FORMAT( concat( YEAR, '-', MONTH, '-01' ), '%Y-%m' )
```