---
title: ibatis使用记录
date: 2020-05-20 15:38:06
---


### ibatis使用记录
#### 动态更新：
```sql
update table_name
<dynamic prepend="set">
	<isNotEmpty property="gmtCreate" prepend=",">
		name = #name#
	</isNotEmpty>
</dynamic>
```
**注意：**实体类的数据类型不要是基本数据类型，例：int-->Integer
#### 批量删除
```sql
delete from table_name
where id in
<iterate property="idList" open="(" close=")" conjunction=",">
	#idList[]#
</iterate>
```
