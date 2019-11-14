# Hive常用命令
## 目录

- [窗口函数](#窗口函数)
- [时间函数](#时间函数)
- [hive的设置](#hive的设置)
- [聚合命令](#聚合命令)

[目录生成工具](https://ecotrust-canada.github.io/markdown-toc/)

## 窗口函数
### 聚合函数+over
### partition by子句
### order by子句
### window子句

- PRECEDING：往前
- FOLLOWING：往后
- CURRENT ROW：当前行
- UNBOUNDED：起点，UNBOUNDED PRECEDING 表示从前面的起点， UNBOUNDED FOLLOWING：表示到后面的终点

### 窗口函数中的序列函数
- NTILE(n)，用于将分组数据按照顺序切分成n片，返回当前切片值
- row_number，值不会存在重复
- rank，排名相等会在名次中留下空位
- dense_rank，生成数据项在分组中的排名，排名相等会在名次中不会留下空位
- LAG和LEAD函数，lag返回向上返回，lead向下返回
  - lag(orderdate,n,'yyyy-mm-dd') over(partition by name order by orderdate ) 回之前第n个orderdate，取不到时默认'yyyy-mm-dd'
- first_value和last_value
 
## 时间函数

### unix_timestamp()
    返回当前时区的unix时间戳
    返回类型：bigint
```    
hive (tmp)> select unix_timestamp() from hive_sum limit 1;
1465875016
```

### from_unixtime(bigint unixtime[,string format])
    时间戳转日期函数
    返回类型：string
```
hive (tmp)> select from_unixtime(unix_timestamp(),’yyyyMMdd’) from hive_sum limit 1;
20160614
```

### unix_timestamp(string date)
    返回指定日期格式的的时间戳
    返回类型：bigint
    注意：如果后面只有date参数，date的形式必须为’yyyy-MM-dd HH:mm:ss’的形式。
```
hive (tmp)> select unix_timestamp(‘2016-06-01’) from hive_sum limit 1;
NULL
hive (tmp)> select unix_timestamp(‘2016-06-01 00:00:00’) from hive_sum limit 1;
1464710400
```

### unix_timestamp(string date,string pattern)
    返回指定日期格式的时间戳
    返回类型:bigint
```
hive (tmp)> select unix_timestamp(‘2016-06-01’,’yyyyMMdd’) from hive_sum limit 1;
1449331200
```

### to_date(string date)
    返回时间字段中的日期部分
    返回类型:string
```
hive (tmp)> select to_date(‘2016-06-01 00:00:00’) from hive_sum limit 1;
2016-06-01
```

### year(string date)
    返回时间字段中的年
    返回类型:int
```
hive (tmp)> select year(‘2016-06-01 00:00:00’) from hive_sum limit 1;
2016
```
### month(string date)
    返回时间字段中的月
    返回类型:int
```
hive (tmp)> select month(‘2016-06-01’) from hive_sum limit 1;
6
```
### day(string date)
    返回时间字段中的天
    返回类型:int
```
hive (tmp)> select day(‘2016-06-01’) from hive_sum limit 1;
1
```

### weekofyear(string date)
    返回时间字段是本年的第多少周
    返回类型:int
```
hive (tmp)> select weekofyear(‘2016-06-01’) from hive_sum limit 1;
22
```
### datediff(string enddate,string begindate)
    返回enddate与begindate之间的时间差的天数
    返回类型:int
```
hive (tmp)> select datediff(‘2016-06-01’,’2016-05-01’) from hive_sum limit 1;
31
```
### date_add(string date,int days)
    返回date增加days天后的日期
    返回类型:string
```
hive (tmp)> select date_add(‘2016-06-01’,15) from hive_sum limit 1;
2016-06-16
```
### date_sub(string date,int days)
    返回date减少days天后的日期
    返回类型:string
```
hive (tmp)> select date_sub(‘2016-06-01’,15) from hive_sum limit 1;
2016-05-17
```

## hive的设置
- 保留表头
  - set hive.cli.print.header=true;
  
## 聚合命令

### GROUPING SETS,GROUPING__ID
在一个GROUP BY查询中，根据不同的维度组合进行聚合，等价于将不同维度的GROUP BY结果集进行UNION ALL
GROUPING__ID的值为<a href="https://www.codecogs.com/eqnedit.php?latex=2^0,2^1,2^2,..." target="_blank"><img src="https://latex.codecogs.com/svg.latex?2^0,2^1,2^2,..." title="2^0,2^1,2^2,..." /></a>
```{sql}
SELECT 
month,
day,
COUNT(DISTINCT cookieid) AS uv,
GROUPING__ID 
FROM lxw1234 
GROUP BY month,day 
GROUPING SETS (month,day) 
ORDER BY GROUPING__ID;
 
month      day            uv      GROUPING__ID
------------------------------------------------
2015-03    NULL            5       1
2015-04    NULL            6       1
NULL       2015-03-10      4       2
NULL       2015-03-12      1       2
NULL       2015-04-12      2       2
NULL       2015-04-13      3       2
NULL       2015-04-15      2       2
NULL       2015-04-16      2       2
 
 
-- 等价于 
SELECT month,NULL,COUNT(DISTINCT cookieid) AS uv,1 AS GROUPING__ID FROM lxw1234 GROUP BY month 
UNION ALL 
SELECT NULL,day,COUNT(DISTINCT cookieid) AS uv,2 AS GROUPING__ID FROM lxw1234 GROUP BY day
```

### CUBE,ROLLUP
