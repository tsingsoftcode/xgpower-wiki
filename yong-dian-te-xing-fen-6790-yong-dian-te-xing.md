### 用电特性分析-用电特性
1.功能简介

```
根据用户月电量表（cust_electr_monthly）、电力用户表（cust_customer）、行业字典表（base_dic_industry）、用户月度电费表（cust_electr_charge_monthly）
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
  （1）.总数据（表格）
  （2）.负荷数据（量，柱图）
  （3）负荷数据（率，折线图）
  属性：  
  尖、峰、谷、平、电费、平均销售电价、总用电量、尖峰时段
```
4.数据库关联关系

```
主表：  
   用户月电量表（cust_electr_monthly）
从表：
   电力用户表（cust_customer）、行业字典表（base_dic_industry）、用户月度电费表（cust_electr_charge_monthly）
```
5.数据处理过程

```
以用户月电量表（cust_electr_monthly）为主表获取负荷数据，左连接电力用户表（cust_customer）、行业字典表（base_dic_industry）、用户月度电费表（cust_electr_charge_monthly）获取用户行业、电费数据
```
6.SQL预览

```
SELECT
	b.industryName 行业,
	DATE_FORMAT(concat(a.year,'-',a.month,'-01'),'%Y-%m') 年月,
	sum( a.sharp ) 尖,
	sum( a.peak ) 峰,
	sum( a.flat ) 谷,
	sum( a.valley ) 平,
	c.charge 电费,
	c.charge/electric 平均销售电价
FROM
	`cust_electr_monthly` a
	LEFT JOIN ( SELECT c.CODE, d.aliasname industryName FROM cust_customer c LEFT JOIN base_dic_industry d ON c.industry = d.id ) b ON a.cust_code = b.CODE 
	left join cust_electr_charge_monthly c on a.year = c.year and a.month = c.month and a.cust_code = c.cust_code
GROUP BY
	DATE_FORMAT(concat(a.year,'-',a.month,'-01'),'%Y-%m')
```
