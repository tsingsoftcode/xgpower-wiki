### 负荷跟踪分析
1.功能介绍

```
以港区、开闭所、配变等不同维度统计负荷变化情况，包括最大负荷、最小负荷、平均负荷、负荷率、峰谷差、
峰谷差率的分析；根据负荷变化情况排名，从配网的角度分析负荷变化情况，哪些区域设备带来的变化
```
2.输入

```
查询条件：日期选择框
触发条件：进入页面或查询条件改变
```
3.输出

```
最大负荷、最小负荷、平均负荷、负荷率、峰谷差、峰谷差率
```
4.数据库关联关系

```
开闭所：
a.将开闭所负荷表dms_switching_power96按时间、负荷点、开闭所编号进行分组并格式化日期至天
b.按时间、开闭所编号进行分组，计算96点数据最大、最小负荷，并计算负荷总量、平均负荷、峰谷差
c.计算负荷率、峰谷差率，并关联测量点表查询开闭所名称
配变、港区逻辑相似
```
5.sql脚本

```
-- 开闭所
select data_date 日期,
       (select name from dms_station ds where ds.code=t3.switching_code) 开闭所,
       total 总量,
       maxv 最大值,
       minv 最小值,
       num 数量,
       vtp 峰谷差,
       avg 平均值,
       avg/maxv 负荷率,
       vtp/maxv 峰谷差率 
       from (
	select data_date,switching_code,total,maxv,minv,num,total/num avg,maxv-minv vtp from (
		select switching_code, data_date, sum(volume) total,max(volume) maxv,min(volume) minv,count(1) num from(
			select switching_code, DATE_FORMAT(data_date,'%Y-%m')data_date, dataidx, sum(volume) volume from dms_switching_power96
			group by switching_code, data_date, dataidx
		)t1 group by data_date,switching_code
	)t2
-- 配变
select data_date 日期,
	   (select name from dms_station ds where ds.code=t3.transformer_code) 变电站,
	   total 总量,
	   maxv 最大负荷,
	   minv 最小负荷,
	   num 数量,
	   vtp 峰谷差,
	   avg 平均负荷,
	   avg/maxv 负荷率,
	   vtp/maxv 峰谷差率 
from (
	select data_date,transformer_code,total,maxv,minv,num,total/num avg,maxv-minv vtp from (
		select transformer_code, data_date, sum(volume) total,max(volume) maxv,min(volume) minv,count(1) num from(
			select transformer_code, DATE_FORMAT(data_date,'%Y-%m')data_date, dataidx, sum(tp) volume from dms_transformer_power96
			group by transformer_code, data_date, dataidx
		)t1 group by data_date,transformer_code
	)t2
)t3
-- 港区
select datatime 日期,
	total 总量,
	maxv 最大负荷,
	minv 最小负荷,
	num 数量,
	vtp 峰谷差,
	avg 平均负荷,
	avg/maxv 负荷率,
	vtp/maxv 峰谷差率 
from (
	select datatime,total,maxv,minv,num,total/num avg,maxv-minv vtp from (
		select datatime ,sum(volume) total,max(volume) maxv,min(volume) minv,count(1) num from (
			select DATE_FORMAT(cload.datatime,'%Y-%m') datatime,cload.dataidx, sum(cload.volume) volume from cust_load_curve cload
				left join  cust_customer  cust
				on cust.code = cload.cust_code
				group by cload.datatime,cload.dataidx
		)t1 group by datatime
	)t2
)t3
```
6.特殊说明

```
a.最大、最小负荷为该月某负荷点的最大、最小值，无需累加
b.负荷率 = 平均负荷/最大负荷
c.峰谷差率 = 峰谷差/最大负荷
d.配变表中有功总功率为负荷
```



