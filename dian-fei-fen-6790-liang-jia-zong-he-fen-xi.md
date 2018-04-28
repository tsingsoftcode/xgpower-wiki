### 电费分析-量价综合分析
1.功能简介

```
根据系统月度电费表（stat_system_electr_charge）、系统月购电信息表（stat_system_buy_power）
以图形及表格展示分析数据；
通过两个维度进行分析  
  （1）.售电（stat_system_electr_charge）
  （2）.购电（stat_system_buy_power）
```
2.输入

```
查询条件：  
  分析时间
触发条件：进入主页面及查询条件改变
```
3.输出

```
  （1）.售电（散点图）
  （2）.购电（散点图）
  （3）.购售电（表格）
```
4.数据库关联关系

```
主表：  
   系统月度电费表（stat_system_electr_charge）  
从表：
   系统月购电信息表（stat_system_buy_power）
```
5.数据处理过程

```
以系统月度电费表（stat_system_electr_charge）为主表获取售电数据，左连接系统月购电信息表（stat_system_buy_power）获取购电数据。
```
6.SQL预览

```
SELECT
	DATE_FORMAT( concat( a.YEAR, '-', a.MONTH, '-01' ), '%Y-%m' ) 年月,
	a.electric 售电量,
	a.electric / a.charge 售电均价,
	b.buy_electric 购电量,
	b.buy_electric / b.buy_fee 购电均价 
FROM
	stat_system_electr_charge a
	LEFT JOIN stat_system_buy_power b ON a.YEAR = b.YEAR 
	AND a.MONTH = b.MONTH
```
