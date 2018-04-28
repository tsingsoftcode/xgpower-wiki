### 缴费欠费特性-缴费特性
1.功能简介

```
描述：  
    针对不同行业，各自统计其预存情况、电费缴纳及时性、缴费金额、缴费渠道（金额、数量、各小区数量、时间）等信息，梳理不同行业的信用特征。
分析目标：  
    从不同维度分析缴费行为，发现缴费特征
实现：  
    （1）缴费方式：缴费表（cust_payment_detail）、系统缴费方式表（cust_dic_payment_mode）  
    （2）行业：缴费表（cust_payment_detail）、电力用户表（cust_customer）、行业字典表（base_dic_industry）  
以图形及表格展示分析数据；
```
2.输入

```
查询条件：  
  缴费方式：分析时间、缴费方式  
  行业：分析时间、行业
触发条件：进入主页面及查询条件改变
```
3.输出

```
  图形：  
    缴费方式：
  （1）.缴费金额（饼图）
  （2）.预存（饼图）
  （3）.数据（表格）
    行业：
  （1）.缴费金额（饼图）
  （2）.预存（饼图）
  （3）.数据（表格）
  属性：  
    缴费方式：缴费方式、缴费日期、缴费金额、预存  
    行业：行业、缴费日期、缴费金额、预存  
    
```
4.数据库关联关系

```
缴费方式：
    主表：  
       缴费表（cust_payment_detail）
    从表：
       系统缴费方式表（cust_dic_payment_mode）  

行业：
    主表：  
       缴费表（cust_payment_detail）
    从表：
       电力用户表（cust_customer）、行业字典表（base_dic_industry）  
```
5.数据处理过程

```
缴费方式：
    以缴费表（cust_payment_detail）为主表获取缴费、预存数据，左连接系统缴费方式表（cust_dic_payment_mode）  获取缴费方式信息。

行业：
    以缴费表（cust_payment_detail）为主表获取缴费、预存数据，左连接电力用户表（cust_customer）、行业字典表（base_dic_industry）  获取行业信息。
```
6.SQL预览

```
缴费方式：
    SELECT
    	DATE_FORMAT( pay_date, '%Y-%m' ) 年月,
    	sum( a.balance ) 预存,
    	sum( a.pay_amount ) 缴费金额,
    	b.NAME 缴费方式 
    FROM
    	cust_payment_detail a
    	LEFT JOIN cust_dic_payment_mode b ON a.payment_mode = b.`code` 
    WHERE
    	pay_date IS NOT NULL 
    GROUP BY
    	DATE_FORMAT( pay_date, '%Y-%m' ),
    	b.`name`

行业：
    SELECT
      c.industryName 行业,
    	DATE_FORMAT(pay_date,'%Y-%m') 年月,
    	sum(a.balance) 预存,
    	sum(a.pay_amount) 缴费金额
    FROM
    	cust_payment_detail a
    	left join (select a.code, b.aliasname industryName from cust_customer a left join base_dic_industry b on a.industry = b.id) c on a.cust_code = c.`code`
    	where pay_date is not null and industryName is not null
    	GROUP BY DATE_FORMAT(pay_date,'%Y-%m'), c.industryName
```
