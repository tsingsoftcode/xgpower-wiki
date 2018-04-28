### 用电特性分析-负荷特性跟踪
1.功能简介

```
根据日负荷数据及行业信息分析港区时间段内（维度：月）行业负荷数据，以柱状图及表格展示分析数据；
分析数据包括最大负荷、最小负荷、平均负荷、负荷率、峰谷差、峰谷差率
```
2.输入

```
查询条件：时间日期选择框，行业下拉选择
触发条件：进入主页面及查询条件改变
```
3.输出

```
最大负荷、最小负荷、平均负荷、负荷率、峰谷差、峰谷差率
```
4.数据库关联关系

```
主表：负荷表cust_load_curve获取负荷数据
关联表：cust_customer获取行业信息
```
5.数据处理过程

```
a.关联客户表，按时间、负荷点索引、行业进行分组处理，获取行业96点负荷数据
b.按时间（年-月）行业进行二次分组获取该行业当月总负荷，96点最大、最小负荷，行业，属于该行业的负荷点数量
c.计算平均负荷、负荷率、峰谷差、峰谷差率
```
6.特殊说明

```
a.最大、最小负荷为该月某负荷点的最大、最小值，无需累加
b.负荷率 = 平均负荷/最大负荷
c.峰谷差率 = 峰谷差/最大负荷
```
7.sql脚本

```
select datatime,total,maxv,minv,num,vtp,avg,avg/maxv loadrate,vtp/maxv vtprate,industry from (
    	select datatime,total,maxv,minv,num,total/num avg,maxv-minv vtp,industry from (
    		select datatime ,sum(volume) total,max(volume) maxv,min(volume) minv,count(1) num,industry from (
    			select DATE_FORMAT(cload.datatime,'%Y-%m') datatime,cload.dataidx, sum(cload.volume) volume,industry from cust_load_curve cload
    				left join  cust_customer  cust
    				on cust.code = cload.cust_code
    				-- where cload.datatime BETWEEN '2018-03-01 00:00:00' and '2018-05-31 23:59:00'
    				group by cload.datatime,cload.dataidx,cust.industry
    		)t1 group by datatime,industry
    	)t2
)t3 
-- where industry in (68) and datatime between '2018-03' and '2018-05'
```
