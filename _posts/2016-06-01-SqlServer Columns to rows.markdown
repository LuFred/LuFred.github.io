---
layout: post
title: "SQL Server实现行列转换"
subtitle: ""
subtitleAuthor: ""
date: 2016-06-01
author: LuJiangBo
category: SqlServer
tags: SqlServer
finished: true
---

## 场景
 早上一个朋友问我这样一个问题，如下图：  
![问题]({{ post.url| prepend: site.url  }}/content/images/201606/2016-06-01-SqlServerColumnstorows.png)  
一看其实就是sql中的列转行问题。东西也不是很复杂，就不废话了直接上脚本，一看就明白了。

## 重点  
* Sql Server 2000通过聚合函数＋ SELECT...CASE 语句实现行列转换。
* Sql Server2005之后加了PIVOT和 UNPIVOT函数来实现行转列、列转行。这里就不多说，可以看微软的[文档](https://technet.microsoft.com/zh-cn/library/ms177410(v=sql.105).aspx)。
 
## 创建测试表及测试数据  
{% highlight Sql Server %}

if object_id('a_testTable') is not null
drop table a_testTable
go
create table a_testTable
(
[no] int,
[name] nvarchar(10),
[age] int
)
go
insert a_testTable values(1,'a',25)
insert a_testTable values(2,'a',30)
insert a_testTable values(1,'b',18)
insert a_testTable values(2,'b',20)
insert a_testTable values(2,'b',30)
go

{% endhighlight %}  

## Sql Server2000实现
{% highlight Sql Server %}

--静态sql
--在列值不多的情况下可以这样写，相当于hardcode
select [no],
    min(case name when 'a' then age else null end) as a,
    min(case name when 'b' then age else null end) as b
    from a_testTable
    group by [no]
 
--动态sql
--在列值比较多的情况下需要拼接出执行脚本字符串
go
declare @sql nvarchar(400)
set @sql=''
select @sql=@sql+', min(case name when '+''''+name+''''+ ' then age else null end) as '+name
from (select distinct name from a_testTable) as tb
set @sql='select [no]'+@sql+' from a_testTable group by [no]'
print @sql
exec(@sql)

{% endhighlight %}
## Sql Server2005实现  
{% highlight Sql Server %}

--静态sql
--在列值不多的情况下可以这样写，相当于hardcode
select * from a_testTable pivot(min(age) for name in(a,b))as t
go

--动态sql
--在列值比较多的情况下需要拼接出执行脚本字符串
declare @sql nvarchar(400)
set @sql=''
select @sql=@sql+','+name from a_testTable group by name
set @sql=substring(@sql,2,len(@sql))
 
set @sql='select * from a_testTable pivot (min(age) for name in('+@sql+'))as t'
 
exec(@sql)
go

{% endhighlight %}







