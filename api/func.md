
### 类型映射说明

udf函数本身使用的是spark类型系统

而pandora拥有自己的类型系统，两者类型匹配信息如下：


|Pandora|Spark|
|:--|:--|
|float|double|
|long|bigint|
|string|string|
|date|timestamp|
|boolean|boolean|
|array|array|
|map|struct|

-

### 日期函数

**函数名称**

datediff

**函数声明**

```
int datediff(Date date1, Date date2)
```
**参数说明**

```
date1，date2：Date类型，被减数和减数，若输入为string类型会隐式转换为date类型后参与运算，其它类型抛异常。
返回值：int类型。任一输入参数是NULL，返回NULL。
备注 :
计算时会按照datepart切掉低单位部分，然后再计算结果。
示例：
    若start = 2005-12-31 23:59:59，end = 2006-01-01 00:00:00:
    datediff(end, start) = 1 
```

-

**函数名称**

from_unixtime

**函数声明**

```
string from_unixtime(bigint unixtime)
```
**参数说明**

```
unixtime：Bigint类型，秒数，unix格式的日期时间值，若输入为string，double类型会隐式转换为bigint后参与运算。
返回值：String类型的日期值，unixtime为NULL时返回NULL。
示例：
    from_unixtime(123456789) = 2009-01-20 21:06:29
用途	
```

-

**函数名称**

to_date

**函数声明**

```
Date to_date(string date)
```
**参数说明**

```
date：Datetime类型值，若输入为String类型会自动转换字符串格式为日期值，若为其它类型抛异常，为空串时抛异常。
返回值：Datetime类型。若任一输入为NULL，返回NULL值。
示例：
    to_date('2009-07-30 04:17:52') = 2009-07-30
```

-

**函数名称**

unix_timestamp

**函数声明**

```
bigint unix_timestamp() 返回当前时间的timestamp

bigint unix_timestamp(string date, string dateFormat) 参数dateFormat默认为 yyyy-MM-dd HH:mm:ss

bigint unix_timestamp(string date) 使用默认dateFormat来解析date并返回

```
**参数说明**

```
date：Datetime类型日期值，若输入为string类型会隐式转换为datetime类型后参与运算，其它类型抛异常。
dateFormat：日期值的解析格式，默认 yyyy-MM-dd HH:mm:ss
返回值：Bigint类型，表示unix格式日期值，date为NULL时返回NULL。

示例：
 unix_timestamp('2017-01-02 15:00:00','yyyy-MM-dd')
 返回1483286400 
```

-

**函数名称**

weekofyear

**函数声明**

```
int weekofyear(Date date)
```
**参数说明**

```
需要注意的是，关于这一周算上一年， 还是下一年，主要是看这一周大多数日期（4天以上）在哪一年多。 算在前一年，就是前一年的最后一周。算在后一年就是后一年的第一周。
参数说明:
date：Date类型日期值，若输入为string类型会隐式转换为datetime类型后参与运算，其它类型抛异常。
返回值：int类型。若输入为NULL，返回NULL。
示例说明：
select weekofyear(to_date("20141229", "yyyymmdd")) from dual;  
返回结果：  
+------------+
| _c0        |
+------------+
| 1          |
+------------+
 -虽然20141229属于2014年，但是这一周的大多数日期是在2015年，因此返回结果为1，表示是2015年的第一周。    
 select weekofyear(to_date("20141231", "yyyymmdd")) from dual；--返回结果为1。  
 select weekofyear(to_date("20151229", "yyyymmdd")) from dual；--返回结果为53。
```

-

### 窗口函数

**函数名称**

avg

**函数声明**

```
avg([distinct] expr) over(partition by col1[, col2…]
    [order by col1 [asc|desc] [, col2[asc|desc]…]] [windowing_clause])
```
**参数说明**

```
distinct：当指定distinct关键字时表示取唯一值的平均值。
expr：Double类型，若输入为string，bigint会隐式转换到double类型后参与运算，其它类型抛异常。当值为NULL时，该行不参与计算。 Boolean类型不允许参与计算。
partition by col1[, col2]…：指定开窗口的列。
order by col1 [asc|desc], col2[asc|desc]：不指定order by时返回当前窗口内所有值的平均值，指定order by时返回结果以指定的方式排序， 并且返回窗口内从开始行到当前行的累计平均值。
返回值：Double类型。
备注:
windowing_clause部分可以用rows指定开窗方式，有两种方式：
rows between x preceding|following and y preceding|following表示窗口范围是从前或后x行到前或后y行。
rows x preceding|following窗口范围是从前或后第x行到当前行。
x，y必须为大于等于0的整数常量，限定范围0 ~ 10000，值为0时表示当前行。必须指定order by才可以用rows方式指定窗口范围
指明distinct关键字时不能写order by。
```

-

**函数名称**

count

**函数声明**

```
bigint count([distinct] expr) over(partition by col1[, col2…]
        [order by col1 [asc|desc][, col2[asc|desc]…]] [windowing_clause])
```
**参数说明**

```
expr：任意类型，当值为NULL时，该行不参与计算。当指定distinct关键字时表示取唯一值的计数值。
partition by col1[, col2…]：指定开窗口的列。
order by col1 [asc|desc], col2[asc|desc]：不指定order by时，返回当前窗口内expr的计数值，指定order by时返回结果以指定的顺序排序， 并且值为当前窗口内从开始行到当前行的累计计数值。
返回值：Bigint类型。
备注:
windowing_clause部分可以用rows指定开窗方式，有两种方式：
rows between x preceding|following and y preceding|following表示窗口范围是从前或后x行到前或后y行。
rows x preceding|following窗口范围是从前或后第x行到当前行。
x，y必须为大于等于0的整数常量，限定范围0 ~ 10000，值为0时表示当前行。必须指定order by才可以用rows方式指定窗口范围
当指定distinct关键字时不能写order by。
示例：假设存在表test_src，表中存在bigint类型的列user_id，
    select user_id,
        count(user_id) over (partition by user_id) as count
    from test_src;
    +---------+------------+
    | user_id |  count     |
    +---------+------------+
    | 1       | 3          |
    | 1       | 3          |
    | 1       | 3          |
    | 2       | 1          |
    | 3       | 1          |
    +---------+------------+
    -- 不指定order by时，返回当前窗口内user_id的计数值
    select user_id,
        count(user_id) over (partition by user_id order by user_id) as count
    from test_src;
    +---------+------------+
    | user_id | count      |
    +---------+------------+
    | 1       | 1          |      -- 窗口起始
    | 1       | 2          |      -- 到当前行共计两条记录，返回2
    | 1       | 3          |
    | 2       | 1          |
    | 3       | 1          |
    +---------+------------+
    -- 指定order by时，返回当前窗口内从开始行到当前行的累计计数值。
```

-

**函数名称**

dense_rank

**函数声明**

```
bigint dense_rank() over(partition by col1[, col2…]
        order by col1 [asc|desc][, col2[asc|desc]…])
```
**参数说明**

```
artition by col1[, col2..]：指定开窗口的列。
order by col1 [asc|desc], col2[asc|desc]：指定排名依据的值。
返回值：Bigint类型。
```

-

**函数名称**

lag

**函数声明**

```
lag(expr，bigint offset, default) over(partition by col1[, col2…]
        [order by col1 [asc|desc][, col2[asc|desc]…]])
```
**参数说明**

```
expr：任意类型。
offset：Bigint类型常量，输入为string，double到bigint的隐式转换，offset > 0。
default：当offset指定的范围越界时的缺省值，常量，默认值为NULL。
partition by col1[, col2..]：指定开窗口的列。
order by col1 [asc|desc], col2[asc|desc]：指定返回结果的排序方式。
返回值：同expr类型。
```

-

**函数名称**

lead

**函数声明**

```
lead(expr, bigint offset, default) over(partition by col1[, col2…]
        [order by col1 [asc|desc][, col2[asc|desc]…]])
```
**参数说明**

```
expr：任意类型。
offset：Bigint类型常量，输入为string，double到bigint的隐式转换，offset > 0。
default：当offset指一的范围越界时的缺省值，常量。
partition by col1[, col2..]：指定开窗口的列。
order by col1 [asc|desc], col2[asc|desc]：指定返回结果的排序方式。
返回值：同expr类型
```

-

**函数名称**

max

**函数声明**

```
max([distinct] expr) over(partition by col1[, col2…]
        [order by col1 [asc|desc][, col2[asc|desc]…]] [windowing_clause])
```
**参数说明**

```
expr：除Boolean以外的任意类型，当值为NULL时，该行不参与计算。当指定distinct关键字时表示取唯一值的最大值(指定该参数与否对结果没有影响)。
partition by col1[, col2…]：指定开窗口的列。
order by col1 [asc|desc], col2[asc|desc]：不指定order by时，返回当前窗口内的最大值。指定order by时，返回结果以指定的方式排序， 并且值为当前窗口内从开始行到当前行的最大值。
返回值：同expr类型。
备注：
windowing_clause部分可以用rows指定开窗方式，有两种方式：
rows between x preceding|following and y preceding|following表示窗口范围是从前或后x行到前或后y行。
rows x preceding|following窗口范围是从前或后第x行到当前行。
x，y必须为大于等于0的整数常量，限定范围0 ~ 10000，值为0时表示当前行。必须指定order by才可以用rows方式指定窗口范围
当指定distinct关键字时不能写order by。
```

-

**函数名称**

min

**函数声明**

```
min([distinct] expr) over(partition by col1[, col2…]
        [order by col1 [asc|desc][, col2[asc|desc]…]] [windowing_clause])
```
**参数说明**

```
expr：除boolean以外的任意类型，当值为NULL时，该行不参与计算。当指定distinct关键字时表示取唯一值的最小值(指定该参数与否对结果没有影响)。
partition by col1[, col2..]：指定开窗口的列。
order by col1 [asc|desc], col2[asc|desc]：不指定order by时，返回当前窗口内的最小值。指定order by时，返回结果以指定的方式排序， 并且值为当前窗口内从开始行到当前行的最小值。
返回值：同expr类型。
备注：
windowing_clause部分可以用rows指定开窗方式，有两种方式：
rows between x preceding|following and y preceding|following表示窗口范围是从前或后x行到前或后y行。
rows x preceding|following窗口范围是从前或后第x行到当前行。
x，y必须为大于等于0的整数常量，限定范围0 ~ 10000，值为0时表示当前行。必须指定order by才可以用rows方式指定窗口范围
当指定distinct关键字时不能写order by。
```

-

**函数名称**

percent_rank

**函数声明**

```
percent_rank() over(partition by col1[, col2…]
        order by col1 [asc|desc][, col2[asc|desc]…])
```
**参数说明**

```
partition by col1[, col2..]：指定开窗口的列。
order by col1 [asc|desc], col2[asc|desc]：指定排名依据的值。
返回值：Double类型，值域为[0, 1]，相对排名的计算方式为为：(rank-1)/(number of rows -1)。
备注:
目前限制单个窗口内的行数不超过10,000,000条。
```

-

**函数名称**

rank

**函数声明**

```
bigint rank() over(partition by col1[, col2…]
        order by col1 [asc|desc][, col2[asc|desc]…])
```
**参数说明**

```
partition by col2[, col2..]：指定开窗口的列。
order by col1 [asc|desc], col2[asc|desc]：指定排名依据的值。
返回值：Bigint类型。
```

-

**函数名称**

row_number

**函数声明**

```
row_number() over(partition by col1[, col2…]
        order by col1 [asc|desc][, col2[asc|desc]…])
```
**参数说明**

```
partition by col1[, col2..]：指定开窗口的列。
order by col1 [asc|desc], col2[asc|desc]：指定结果返回时的排序的值。
返回值：Bigint类型。
```

-

**函数名称**

stddev

**函数声明**

```
double stddev([distinct] expr) over(partition by col1[, col2…]
        [order by col1 [asc|desc][, col2[asc|desc]…]] [windowing_clause])
decimal stddev([distinct] expr) over(partition by col1[,col2…] [order by col1 [asc|desc][, col2[asc|desc]…]] [windowing_clause])
```
**参数说明**

```
expr：Double类型或Decimal类型。若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。当输入值为NULL时忽略该行。 当指定distinct关键字时表示计算唯一值的总体标准差。
partition by col1[, col2..]：指定开窗口的列。
order by col1 [asc|desc], col2[asc|desc]：不指定order by时，返回当前窗口内的总体标准差。指定order by时，返回结果以指定的方式排序， 并且值为当前窗口内从开始行到当前行的总体标准差。
返回值：输入为Decimal类型时返回Decimal类型，否则返回Double类型。
备注：
windowing_clause部分可以用rows指定开窗方式，有两种方式：
rows between x preceding|following and y preceding|following表示窗口范围是从前或后x行到前或后y行。
rows x preceding|following窗口范围是从前或后第x行到当前行。
x，y必须为大于等于0的整数常量，限定范围0 ~ 10000，值为0时表示当前行。必须指定order by才可以用rows方式指定窗口范围
当指定distinct关键字时不能写order by。
```

-

**函数名称**

stddev_samp

**函数声明**

```
double stddev_samp([distinct] expr) over(partition by col1[, col2…]
        [order by col1 [asc|desc][, col2[asc|desc]…]] [windowing_clause])
```
**参数说明**

```
expr：Double类型。若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。当输入值为NULL时忽略该行。 当指定distinct关键字时表示计算唯一值的样本标准差。
partition by col1[, col2..]：指定开窗口的列。
order by col1 [asc|desc], col2[asc|desc]：不指定order by时，返回当前窗口内的样本标准差。指定order by时，返回结果以指定的方式排序， 并且值为当前窗口内从开始行到当前行的样本标准差。
返回值：Double类型。
备注：
windowing_clause部分可以用rows指定开窗方式，有两种方式：
rows between x preceding|following and y preceding|following表示窗口范围是从前或后x行到前或后y行。
rows x preceding|following窗口范围是从前或后第x行到当前行。
x，y必须为大于等于0的整数常量，限定范围0 ~ 10000，值为0时表示当前行。必须指定order by才可以用rows方式指定窗口范围
当指定distinct关键字时不能写order by。
```

-

**函数名称**

sum

**函数声明**

```
sum([distinct] expr) over(partition by col1[, col2…]
        [order by col1 [asc|desc][, col2[asc|desc]…]] [windowing_clause])
```
**参数说明**

```
expr：Double类型，当输入为string，bigint时隐式转换为double参与运算，其它类型报异常。当值为NULL时，该行不参与计算。 指定distinct关键字时表示计算唯一值的汇总值。
partition by col1[, col2..]：指定开窗口的列。
order by col1 [asc|desc], col2[asc|desc]：不指定order by时，返回当前窗口内expr的汇总值。指定order by时， 返回结果以指定的方式排序，并且返回当前窗口从首行至当前行的累计汇总值。
返回值：输入参数是bigint返回bigint，输入参数为double或string时，返回double类型。
备注：
windowing_clause部分可以用rows指定开窗方式，有两种方式：
rows between x preceding|following and y preceding|following表示窗口范围是从前或后x行到前或后y行。
rows x preceding|following窗口范围是从前或后第x行到当前行。
x，y必须为大于等于0的整数常量，限定范围0 ~ 10000，值为0时表示当前行。必须指定order by才可以用rows方式指定窗口范围
当指定distinct时不能用order by。
```

-

### 字符串函数

**函数名称**

concat

**函数声明**

```
string concat(string a, string b...)
```
**参数说明**

```
a，b等为String类型，若输入为bigint,decimal，double或datetime类型会隐式转换为string后参与运算，其它类型报异常。
返回值：String类型。如果没有参数或者某个参数为NULL，结果均返回NULL。
示例：
    concat('ab', 'c') = 'abc'
    concat() = NULL
    concat('a', null, 'b') = NULL
```

-

**函数名称**

get_json_object

**函数声明**

```
string get_json_object(string json,string path)
```
**参数说明**

```
json:String类型，标准的json格式字符串。
path:String类型，用于描述在json中的path，以$开头。
返回值：String类型
注解： 如果json为空或者非法的json格式，返回NULL 如果path为空或者不合法（json中不存在）返回NULL 如果json合法，path也存在则返回对应字符串
示例1
+----+
 json
+----+
{"store":
   {"fruit":[{"weight":8,"type":"apple"},{"weight":9,"type":"pear"}],
      "bicycle":{"price":19.95,"color":"red"}
   },
   "email":"amy@only_for_json_udf_test.net",
   "owner":"amy"
 }
通过以下查询，可以提取json对象中的信息：
SELECT get_json_object(src_json.json, '$.owner') FROM src_json;
 amy
SELECT get_json_object(src_json.json, '$.store.fruit[0]') FROM src_json;
 {"weight":8,"type":"apple"}
SELECT get_json_object(src_json.json, '$.non_exist_key') FROM src_json;
 NULL
示例2
get_json_object('{"array":[[aaaa,1111],[bbbb,2222],[cccc,3333]]}','$.array[1].[1]') = "2222"
get_json_object('{"aaa":"bbb","ccc":{"ddd":"eee","fff":"ggg","hhh":["h0","h1","h2"]},"iii":"jjj"}','$.ccc.hhh[*]') = "["h0","h1","h2"]"
get_json_object('{"aaa":"bbb","ccc":{"ddd":"eee","fff":"ggg","hhh":["h0","h1","h2"]},"iii":"jjj"}','$.ccc.hhh[1]') = "h1"
```

-

**函数名称**

instr

**函数声明**

```
bigint instr(string str1, string str2[, bigint start_position[, bigint nth_appearance]])
```
**参数说明**

```
str1：String类型，搜索的字符串，若输入为bigint，decimal，double或datetime类型会隐式转换为string后参与运算，其它类型报异常。
str2：String类型，要搜索的子串，若输入为bigint，decimal，double或datetime类型会隐式转换为string后参与运算，其它类型报异常。
start_position：Bigint类型，其它类型会抛异常，表示从str1的第几个字符开始搜索，默认起始位置是第一个字符位置1。开始位置如果小于等于0会引发异常。
nth_appearance：Bigint类型，大于0，表示子串在字符串中的第nth_appearance次匹配的位置，如果nth_appearance为其它类型或小于等于0会抛异常。
返回值：Bigint类型
备注:
如果在str1中未找到str2，返回0。
任一输入参数为NULL返回NULL
如果str2为空串时总是能匹配成功，因此instr(‘abc’, ‘’) 会返回1
示例：
    instr('Tech on the net', 'e') = 2
    instr('Tech on the net', 'e', 1, 1) = 2
    instr('Tech on the net', 'e', 1, 2) = 11
    instr('Tech on the net', 'e', 1, 3) = 14
```

-

**函数名称**

keyvalue

**函数声明**

```
keyvalue(string srcstr，string split1，string split2， string key)
keyvalue(string srcstr， string key) //split1 = ";"，split2 = ":"
```
**参数说明**

```
srcStr 输入待拆分的字符串
key 返回该指定键的值
split1, split2 拆分的字符串，只有两个参数时，默认split1为’;’, split2为’:’。当某个被split1拆分后的字符串中有多个split2时，返回结果未定义；
返回值:
String类型
Split1或split2 为NULL时，返回NULL.
srcStr,key为NULL或者没有匹配的key时，返回NULL
如果有多个key-value匹配，返回第一个匹配上的key对应的value
示例：
keyvalue('0:11:2', 1) = '2'
keyvalue("decreaseStore:1xcard:1isB2C:1	f:21910cart:1shipping:2pf:0market:shoesinstPayAmount:0", "",":","tf") = "21910"
keyvalue("七牛=pandora=2;七牛=数据平台", ";","=", "七牛云") 返回未知结果，请用户避免这种用法
```

-

**函数名称**

length

**函数声明**

```
bigint length(string str)
```
**参数说明**

```
str：String类型，若输入为bigint，decimal，double或datetime类型会隐式转换为string后参与运算，其它类型报异常。
返回值：Bigint类型。若str是NULL返回NULL。如果str非UTF-8编码格式，返回-1。
示例
    length('hi! 中国') = 6
```


-

**函数名称**

md5

**函数声明**

```
string md5(string value)
```
**参数说明**

```
value：String类型，如果输入类型是bigint，decimal，double或者datetime会隐式转换成string类型参与运算，其它类型报异常。输入为NULL，返回NULL。
返回值：String类型。
```

-

**函数名称**

parse_url

**函数声明**

```
string parse_url(string url, string part[,string key])
```
**参数说明**

```
url或part为NULL则返回NULL，url为无效抛异常。
part：String 类型，支持HOST, PATH, QUERY, REF, PROTOCOL, AUTHORITY, FILE, 和 USERINFO，不区分大小写，不在此范围抛异常
当part为QUERY时根据key的值取出在query string中的value值，否则忽略key参数。
返回值：String类型。
示例：
url = file://username:password@example.com:8042/over/there/index.dtb?type=animal&name=narwhal#nose
parse_url('url', 'HOST') = "example.com"
parse_url('url', 'PATH') = "/over/there/index.dtb"
parse_url('url', 'QUERY') = "type=animal&name=narwhal"
parse_url('url', 'QUERY', 'name') = "narwhal"
parse_url('url', 'REF') = "nose"
parse_url('url', 'PROTOCOL') = "file"
parse_url('url', 'AUTHORITY') = "username:password@example.com:8042"
parse_url('url', 'FILE') = "/over/there/index.dtb?type=animal&name=narwhal"
parse_url('url', 'USERINFO') = "username:password"
```


-

**函数名称**

regexp_extract

**函数声明**

```
string regexp_extract(string source, string pattern[, bigint occurrence])
```
**参数说明**

```
	
source：String类型，待搜索的字符串。
pattern：String类型常量，pattern为空串时抛异常，pattern中如果没有指定group，抛异常。
occurrence：Bigint类型常量，必须>=0，其它类型或小于0时抛异常，不指定时默认为1，表示返回第一个group。若occurrence = 0，返回满足整个pattern的子串。
返回值：String类型，任一输入为NULL返回NULL。
示例：
    regexp_extract('foothebar', 'foo(.*?)(bar)', 1) = the
    regexp_extract('foothebar', 'foo(.*?)(bar)', 2) = bar
    regexp_extract('foothebar', 'foo(.*?)(bar)', 0) = foothebar
    regext_extract('8d99d8', '8d(\d+)d8') = 99
    regexp_extract('foothebar', 'foothebar')
    -- 异常返回，pattern中没有指定group
```


-

**函数名称**

regexp_replace

**函数声明**

```
string regexp_replace(string str, string regexp, string rep)
```
**参数说明**

```
str：String类型，待替换的字符串。
regexp：String类型，正则表达式。
req：String类型，替换字符串。
返回值：String类型，当str中完全与regexp不匹配，不进行替换。当输入str，regexp，rep存在NULL时返回NULL，若str中不存在与regexp匹配子串，则返回原字符串str。
示例：
    regexp_replace('100-200', '(\\d+)', 'num') = num-num
    regexp_replace("100-200", "-", "num") = 100num200
    regexp_replace("100-200", "-", NULL) = NULL
    regexp_replace("100-200", NULL, "num") = NULL
```


-

**函数名称**

substr

**函数声明**

```
string substr(string str, bigint start_position[, bigint length])
```
**参数说明**

```
str：String类型，若输入为bigint，decimal，double或者datetime类型会隐式转换为string后参与运算，其它类型报异常。
start_position：Bigint类型，当start_position为负数时表示开始位置是从字符串的结尾往前倒数，最后一个字符是-1，起始位置为1。其它类型会隐式转换为Bigint类型。
length：Bigint类型，大于0，其它类型或小于等于0抛异常。子串的长度。
返回值：String类型。若任一输入为NULL，返回NULL。
备注 :
当length被省略时，返回到str结尾的子串。
示例
    substr("abc", 2) = "bc"
    substr("abc", 2, 1) = "b"
```

-

**函数名称**

lower

**函数声明**

```
string lower(string source)
```
**参数说明**

```
source：String类型，若输入为bigint，decimal，double或者datetime类型会隐式转换为string后参与运算，其它类型报异常。
返回值：String类型。输入为NULL时返回NULL。
示例：
    lower("aBcd") = "abcd"
    lower("哈哈Cd") = "哈哈cd"
```

-

**函数名称**

upper

**函数声明**

```
string upper(string source)
```
**参数说明**

```
source：String类型，若输入为bigint，decimal，double或者datetime类型会隐式转换为string后参与运算，其它类型报异常。
返回值：String类型。输入为NULL时返回NULL。
示例：
    upper("aBcd") = "ABCD"
    upper("哈哈Cd") = "哈哈CD"
```

-

**函数名称**

trim

**函数声明**

```
string trim(string str)
```
**参数说明**

```
str：String类型，若输入为bigint，decimal，double或者datetime类型会隐式转换为string后参与运算，其它类型报异常。
返回值：String类型。输入为NULL时返回NULL。
```

-

### 数学函数

**函数名称**

abs

**函数声明**

```
double abs(double number)
bigint abs(bigint number)
```
**参数说明**

```
number：double或bigint类型，输入为bigint时返回bigint，输入为double时返回double类型。 若输入为string类型会隐式转换到double类型后参与运算，其它类型抛异常。
返回值：double或者bigint类型，取决于输入参数的类型。若输入为null，返回null。
备注：
当输入bigint类型的值超过bigint的最大表示范围时，会返回double类型，这种情况下可能会损失精度。
示例：
abs(null) =  null
abs(-1) = 1
abs(-1.2) = 1.2
abs("-2") = 2.0
abs(122320837456298376592387456923748) = 1.2232083745629837e32
```
-

**函数名称**

acos

**函数声明**

```
double acos(double number)
decimal acos(decimal number)
```
**参数说明**

```
number：Double类型或Decimal类型，-1≤number≤1。若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。
返回值：Double类型或Decimal类型，值域在0 ~ π 之间。若number为NULL，返回NULL。
示例：
    acos("0.87") = 0.5155940062460905
    acos(0) = 1.5707963267948966
```

-

**函数名称**

asin

**函数声明**

```
double asin(double number)
decimal asin(decimal number)
```
**参数说明**

```
number：Double类型或Decimal类型，-1≤number≤1。若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。
返回值：Double类型或Decimal类型，值域在-π/2 ~π/2之间。若number为NULL，返回NULL。
示例：
    asin(1) = 1.5707963267948966
    asin(-1) = -1.5707963267948966
```

-

**函数名称**

atan

**函数声明**

```
double atan(double number)
decimal atan(decimal number)
```
**参数说明**

```
number：Double或Decimal类型，若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。
返回值：若输入为Decimal则返回Decimal类型，否则返回Double类型，值域在-π/2 ~π/2之间。若number为NULL，返回NULL。
示例：
    atan(1) = 0.7853981633974483
    atan(-1) = -0.7853981633974483
```

-

**函数名称**

ceil

**函数声明**

```
bigint ceil(double value)
bigint ceil(decimal value)
```
**参数说明**

```
value：Double类型或Decimal类型，若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。
返回值：Bigint类型。任一输入为NULL，返回NULL。
示例：
    ceil(1.1) = 2
    ceil(-1.1) = -1
```

-

**函数名称**

conv

**函数声明**

```
string conv(string input, bigint from_base, bigint to_base)
```
**参数说明**

```
input：以string表示的要转换的整数值，接受bigint，double的隐式转换。
from_base，to_base：以十进制表示的进制的值，可接受的的值为2，8，10，16。接受string及double的隐式转换。
to_base，to_base：以十进制表示的进制的值，可接受的的值为2，8，10，16。接受string及double的隐式转换。
返回值：String类型。任一输入为NULL，返回NULL。转换过程以64位精度工作，溢出时报异常。输入如果是负值，即以’-‘开头，报异常。 如果输入的是小数，则会转为整数值后进行进制转换，小数部分会被舍弃。
示例:
    conv('1100', 2, 10) = '12'
    conv('1100', 2, 16) = 'C'
    conv('ab', 16, 10) = '171'
    conv('ab', 16, 16) = 'AB'
```

-

**函数名称**

cos

**函数声明**

```
double cos(double number)
decimal cos(decimal number)
```
**参数说明**

```
number：Double类型或Decimal类型。若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。输入为弧度值。
返回值：Double类型或Decimal类型。若number为NULL，返回NULL。
示例：
    cos(3.1415926/2)=2.6794896585028633e-8
    cos(3.1415926)=0.9999999999999986
```

-

**函数名称**

cosh

**函数声明**

```
double cosh(double number)
decimal cosh(decimal number)
```
**参数说明**

```
number：Double类型或Decimal类型。若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。 返回值：Double类型或Decimal类型。若number为NULL，返回NULL。
```

-

**函数名称**

exp

**函数声明**

```
double exp(double number)
decimal exp(decimal number)
```
**参数说明**

```
number：Double类型或Decimal类型。若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。 返回值：Double类型或Decimal类型。若number为NULL，返回NULL
```

-

**函数名称**

floor

**函数声明**

```
bigint floor(double number)
bigint floor(decimal number)
```
**参数说明**

```
number：Double类型或Decimal类型，若输入为string类型或bigint型会隐式转换到double类型后参与运算，其他类型抛异常 返回值：返回Bigint类型。若number为NULL，返回NULL。
示例:
    floor(1.2)=1
    floor(1.9)=1
    floor(0.1)=0
    floor(-1.2)=-2
    floor(-0.1)=-1
    floor(0.0)=0
    floor(-0.0)=0
```

-

**函数名称**

ln

**函数声明**

```
double ln(double number)
decimal ln(decimal number)
```
**参数说明**

```
number：Double类型或Decimal类型，若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。若number为NULL返回NULL, 若number为负数或零，则抛异常。 返回值：Double类型或Decimal类型。
```

-

**函数名称**

log

**函数声明**

```
double log(double base, double x)
decimal log(decimal base, decimal x)
```
**参数说明**

```
	
base：Double类型或Decimal类型，若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。
x：Double类型或Decimal类型，若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。 返回值：Double类型或Decimal类型的对数值，若base和x中存在NULL，则返回NULL；若base和x中某一个值为负数或0，会引发异常；若base为1(会引发一个除零行为)也会引发异常。
```

-

**函数名称**

pow

**函数声明**

```
double pow(double x, double y)
decimal pow(decimal x, decimal y)
```
**参数说明**

```
	
X：Double类型或Decimal类型，若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。
Y：Double类型或Decimal类型，若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。 返回值：Double类型或Decimal类型。若x或y为NULL，则返回NULL
```

-

**函数名称**

rand

**函数声明**

```
double rand(bigint seed)
decimal rand(decimal seed)
```
**参数说明**

```
seed：Bigint类型，随机数种子，决定随机数序列的起始值。 返回值：Double类型或Decimal类型
```

-

**函数名称**

round

**函数声明**

```
double round(double number, [bigint decimal_places])
decimal round(decimal number, [bigint decimal_places])
```
**参数说明**

```
number：Double类型或Decimal类型，若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。
decimal_place：Bigint类型常量，四舍五入计算到小数点后的位置，其他类型参数会引发异常. 如果省略表示四舍五入到个位数。默认值为0。 返回值：返回Double类型或Decimal类型。若number或decimal_places为NULL，返回NULL。
备注：
decimal_places可以是负数。负数会从小数点向左开始计数，并且不保留小数部分；如果decimal_places超过了整数部分长度，返回0. 示例：
    round(125.315) = 125.0
    round(125.315, 0) = 125.0
    round(125.315, 1) = 125.3
    round(125.315, 2) = 125.32
    round(125.315, 3) = 125.315
    round(-125.315, 2) = -125.32
    round(123.345, -2) = 100.0
    round(null) = null
    round(123.345, 4) = 123.345
    round(123.345, -4) = 0.0
```

-

**函数名称**

sin

**函数声明**

```
double sin(double number)
decimal sin(decimal number)
```
**参数说明**

```
number：Double类型或Decimal类型。若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。 返回值：Double类型或Decimal类型。若number为NULL，返回NULL。输入为弧度值。
```

-

**函数名称**

sinh

**函数声明**

```
double sinh(double number)
decimal sinh(decimal number)
```
**参数说明**

```
number：Double类型或Decimal类型。若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。 返回值：Double类型或Decimal类型。若number为NULL，返回NULL。
```

-

**函数名称**

sqrt

**函数声明**

```
double sqrt(double number)
decimal sqrt(decimal number)
```
**参数说明**

```
number：Double类型或Decimal类型，必须大于0。小于0时引发异常。若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。 返回值：返回Double类型或Decimal类型。若number为NULL，返回NULL
```

-

**函数名称**

tan

**函数声明**

```
double tan(double number)
decimal tan(decimal number)
```
**参数说明**

```
number：Double类型或Decimal类型。若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。 返回值：Double类型或Decimal类型。若number为NULL，返回NULL。
```

-

**函数名称**

tanh

**函数声明**

```
double tanh(double number)
decimal tanh(decimal number)
```
**参数说明**

```
number：Double类型或Decimal类型。若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。 返回值：Double类型或Decimal类型。若number为NULL，返回NULL。
```


-

### 聚合函数

**函数名称**

avg

**函数声明**

```
double avg(double value)
decimal avg(decimal value)
```
**参数说明**

```
Double或者Decimal类型，若输入为String或Bigint会隐式转换到Double类型后参与运算，其它类型抛异常。当value值为NULL时，该行不参与计算。Boolean类型不允许参与计算。
返回值：若输入Decimal类型，返回Decimal类型，其他合法输入类型返回Double类型
示例：
tbla输入数据
 +-------+
 | value |
 +-------+
 | 1      |
 | 2      |
 | NULL|
 +-------+
SELECT AVG(value) AS avg FROM tbla;
+------+
| avg  |
+------+
| 1.5  |
+------+"
```

-

**函数名称**

count

**函数声明**

```
bigint count([distict|all] value)
```
**参数说明**

```
distinct|all：指明在计数时是否去除重复记录，默认是all，即计算全部记录，如果指定distinct，则可以只计算唯一值数量。
value：可以为任意类型，当value值为NULL时，该行不参与计算，value可以为，当count()时，返回所有行数。
返回值：Bigint类型。
示例：
    -- 如表tbla有列col1类型为bigint
    +------+
    | COL1 |
    +------+
    | 1    |
    +------+
    | 2    |
    +------+
    | NULL |
    +------+
    select count(*) from tbla;  -- 值为3,
    select count(col1) from tbla;  -- 值为2
聚合函数可以和group by一同使用，例如：假设存在表test_src，存在如下两列：key string类型，value double类型，
    -- test_src的数据为
    +-----+-------+
    | key | value |
    +-----+-------+
    | a   | 2.0   |
    +-----+-------+
    | a   | 4.0   |
    +-----+-------+
    | b   | 1.0   |
    +-----+-------+
    | b   | 3.0   |
    +-----+-------+
    -- 此时执行如下语句，结果为：
    select key, count(value) as count from test_src group by key;
    +-----+-------+
    | key | count |
    +-----+-------+
    | a   | 2     |
    +-----+-------+
    | b   | 2     |
    +-----+-------+
```

-

**函数名称**

max

**函数声明**

```
max(value)
```
**参数说明**

```
value：可以为任意类型，当列中的值为NULL时，该行不参与计算。Boolean类型不允许参与运算。
返回值：与value类型相同。
示例：
    -- 如表tbla有一列value，类型为bigint
    +------+
    | value |
    +------+
    | 1    |
    +------+
    | 2    |
    +------+
    | NULL |
    +------+
    select max(value) from tbla; -- 返回值为2
```

-

**函数名称**

min

**函数声明**

```
min(value)
```
**参数说明**

```
value：可以为任意类型，当列中的值为NULL时，该行不参与计算。Boolean类型不允许参与计算。
示例：
    -- 如表tbla有一列value，类型为bigint
    +------+
    | value|
    +------+
    | 1    |
    +------+
    | 2    |
    +------+
    | NULL |
    +------+
    select min(value) from tbla; -- 返回值为1
```

-

**函数名称**

stddev

**函数声明**

```
double stddev(double number)
decimal stddev(decimal number)
```
**参数说明**

```
number：Double类型或者Decimal类型。若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。当输入值为NULL时忽略。
返回值：Double类型或者Decimal类型
```

-

**函数名称**

stddev_samp

**函数声明**

```
	
double stddev_samp(double number)
decimal stddev_samp(decimal number)
```

**参数说明**

```
number：Double类型或者Decimal类型。若输入为string类型或bigint类型会隐式转换到double类型后参与运算，其他类型抛异常。当输入值为NULL时忽略。
返回值：Double类型或者Decimal类型
```

-

**函数名称**

sum

**函数声明**

```
sum(value)
```
**参数说明**

```
value：double、decimal或Bigint类型，若输入为string会隐式转换到double类型后参与运算，当列中的值为NULL时，该行不参与计算。Boolean类型不允许参与计算。
返回值：输入为bigint时返回bigint，输入为double或string时返回double类型。
示例：
    -- 如表tbla有一列value，类型为bigint
    +------+
    | value|
    +------+
    | 1    |
    +------+
    | 2    |
    +------+
    | NULL |
    +------+
    select sum(value) from tbla; -- 返回值为3
```


-

### 其他函数

**函数名称**

cast

**函数声明**

```
cast(expr as)
```
**参数说明**

```
将表达式的结果转换成目标类型，如cast(‘1’ as bigint)将字符串‘1’转为整数类型的1，如果转换不成功或不支持的类型转换会引发异常
cast(double as bigint)，将double值转换成bigint。
cast(string as bigint) 在将字符串转为bigint时，如果字符串中是以整型表达的数字，会直接转为bigint类型。 如果字符串中是以浮点数或指数形式表达的数字，则会先转为double类型，再转为bigint类型。
cast(string as datetime) 或 cast(datetime as string)时，会采用默认的日期格式yyyy-mm-dd hh:mi:ss。
```

-

**函数名称**

coalesce

**函数声明**

```
coalesce(expr_1, expr_2, ...)
```
**参数说明**

```
expr_i(i=1,2,3...)是要测试表达式。
返回值：返回第一个非空的表达式值。
备注:
所有这些值类型必须相同或为NULL，否则会尝试进行隐式转换。
```

-

**函数名称**

greatest

**函数声明**

```
greatest(var1, var2, …)
```
**参数说明**

```
var1，var2可以为bigint，double，datetime或者string。若所有值都为NULL则返回NULL。
返回值:
输入参数中的最大值，当不存在隐式转换时返回同输入参数类型。
NULL为最小值。
当输入参数类型不同时，double，bigint，string之间的比较转为double；string，datetime的比较转为datetime。不允许其它的隐式转换
```

-

**函数名称**

least

**函数声明**

```
least(var1, var2, …)
```
**参数说明**

```
var1，var2可以为bigint，double，datetime或者string。若所有值都为NULL则返回NULL。
返回值：
输入参数中的最小值，当不存在隐式转换时返回同输入参数类型。
NULL为最小。
有类型转换时，double，bigint，string之间的转换返回double。string，datetime之间的转换返回datetime。不允许其它的隐式类型转换。
```

-
