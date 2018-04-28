### 电量跟踪分析
1.功能简介

```
以港区、开闭所、配变等不同维度统计电量变化情况，包括本期电量、同期电量、同比增长率、环比增长率、
环比增量的分析；根据电量变化情况排名，从配网的角度分析电量变化情况，哪些区域设备带来的变化
```
2.输入

```
查询条件：时间日期选择框
触发条件：进入主页面及查询条件改变
```
3.输出

```
本期电量、同期电量、同比增长率、环比增长率、环比增量
```
4.数据库关联关系

```
开闭所：
a.表dms_switching_electr_monthly自关联t1表示本期，t2表示同期，t3表示环比
b.根据自关联表计算本期电量、同期电量、同比增长率、环比增长率、环比增量
港区及配变逻辑相同
```
5.输出

```
本期电量、同期电量、同比增长率、环比增长率、环比增量
```
6.特殊说明

```
本期电量：本月电量
同期电量：上一年度的本月电量
****
环比增量 = 本月电量 - 上月电量
同比增长率 = （本月电量-上一年度的本月电量）/ 上一年度的本月电量
环比增长率 = （本月电量-上一个月的电量）/ 上一个月的电量

```
7.sql脚本

```
select distinct t1.id,
	(select name from dms_station ds where ds.code=t1.switching_code) 开闭所,
	 t1.year 年,
        t1.month 月,
        concat(t1.year, "-",t1.month) 年月,
        t1.volume 本期电量,  
        ifnull(t2.volume,0) 同期电量, 
        ifnull(t3.volume,0) 上月电量,
	    t1.volume - ifnull(t3.volume,0) 环比增量,
	    ifnull((t1.volume - t2.volume)/t2.volume,0) 同比增长率,
	    ifnull((t1.volume - t3.volume)/t3.volume,100) 环比增长率
 from dms_switching_electr_monthly t1 -- 当前
left join dms_switching_electr_monthly t2 -- 上一年
on t1.year = t2.year +1 and t1.month = t2.month
left join dms_switching_electr_monthly t3 -- 上一月
on t1.year = t3.year and t1.month= t3.month + 1 

-- 配变
select distinct t1.id,
	(select name from dms_station ds where ds.code=t1.transformer_code) 变电站,
	t1.year 年,
        t1.month 月,
        concat(t1.year, "-",t1.month) 年月,
        t1.volume 本期电量,  
        ifnull(t2.volume,0) 同期电量, 
        ifnull(t3.volume,0) 上月电量,
	    t1.volume - ifnull(t3.volume,0) 环比增量,
	    ifnull((t1.volume - t2.volume)/t2.volume,0) 同比增长率,
	    ifnull((t1.volume - t3.volume)/t3.volume,100) 环比增长率
 from dms_transformer_electr_monthly t1 -- 当前
left join dms_transformer_electr_monthly t2 -- 上一年
on t1.year = t2.year +1 and t1.month = t2.month
left join dms_transformer_electr_monthly t3 -- 上一月
on t1.year = t3.year and t1.month= t3.month + 1

-- 港区
select distinct t1.id,t1.year 年,
       t1.month 月,
       concat(t1.year, "-",t1.month) 年月,
       t1.electric 本期电量,
       ifnull(t2.electric,0) 上月电量, 
       ifnull(t3.electric,0) 同期电量,
       t1.electric - ifnull(t3.electric,0) 环比增量, 
       ifnull((t1.electric - t2.electric)/t2.electric,0) 同比增长率,
       ifnull((t1.electric - t3.electric)/t3.electric,0) 环比增长率
 from stat_system_electr_charge t1 -- 当前
left join stat_system_electr_charge t2 -- 上一年
on t1.year = t2.year +1 and t1.month = t2.month
left join stat_system_electr_charge t3 -- 上一月
on t1.year = t3.year and t1.month= t3.month + 1 
```
