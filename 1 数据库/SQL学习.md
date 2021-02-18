## 检索数据

检索单个列： select prod_name from products;

检索多个列： 

检索所有列： *

检索不同的行： distinct

限制结果：limit 5   limit 1, 1  limit 4 offset 3, (从第3行开始的4行，行下标从0开始。也就是说忽略前3行，从第四行开始返回4行。)

使用完全限定的表名： 

​	select products.prod_name from products;

​	select products.prod_name from crashcourse.products;



## 排序检索数据

排序数据：order by 子句

按多个列排序：order by prod_price, prod_name; 排序按所规定的列进行，先排序prod_price再排序prod_name

指定排序方向：

​	order by prod_price desc;    desc关键字降序，默认升序asc

​	order by prod_price desc, prod_name;    多列排序，需要对每个列单独制定desc

​	order by prod_price desc limit 1 找最高



## 过滤数据

使用where子句：where -》 order by

where子句操作符： =, <>,  !=,  <, <=, >, >=, between

检查单个值

不匹配检查： <> !=   单引号用来限定字符串，值与字符串比较时使用。

范围值检查： between 5 and 10;

空值检查： where prod_price is null; where prod_price is not null;



## 数据过滤

组合where子句

​	and操作符： where vend_id = 1003 and prod_price <= 10;

​	or操作符：

​	计算次序：and > or    where (vend_id = 1002 or vend_id = 1003) and prod_price >= 10;

in操作符： where vend_id in (1002, 1003)

Not 操作符：where子句中用来否定后跟条件的关键字，where vend_id not in (1002,1003);



## 用通配符进行过滤

like操作符

​	百分号（%）通配符：where prod_name like 'jet%';    'jet%'是搜索模式

​	下划线（_）通配符：只匹配单个字符

使用通配符的技巧



## 用正则表达式进行搜索

基本字符匹配：where prod_name regexp ‘1000’

进行or匹配：where prod_name regex '1000|2000'

巴拉巴拉。。。。



## 创建计算字段

计算字段

拼接字段：将值联结到一起构成单个值。 

​	Select concat(vend_name, '(', vend_country, ')') from vendors order by vend_name;

​	rtrim() 函数去掉右边空格，trim()去掉左边的空格

使用别名 ：as关键字，可以对表和字段使用，当一个from中有两个相同的表时，可以使用别名对他们进行区别。

执行算术计算：

​	select prod_id, quantity, item_price, quantity * item_price as expanded_price from order items where order_num = 20005;

​	算术运算符： + - * /  也支持取余数%

​	select 3 * 2；



## 使用数据处理函数

文本处理函数

​	select upper(vend_name) as vend_name_upcase from vendors order by vend_name;

![B6DE1F96-19F4-4188-907F-322E0C54E52A](/Users/fanxudong/IdeaProjects/blog/1 数据库/assert/B6DE1F96-19F4-4188-907F-322E0C54E52A.png)

​	![3B1C1C01-93C4-4279-855C-1E6BA5C6579F](/Users/fanxudong/IdeaProjects/blog/1 数据库/assert/3B1C1C01-93C4-4279-855C-1E6BA5C6579F.png)

日期和时间处理函数

![2D040BF7-A399-4FE7-BB79-E7B6A1728A68](/Users/fanxudong/IdeaProjects/blog/1 数据库/assert/2D040BF7-A399-4FE7-BB79-E7B6A1728A68.png)

数值处理函数

![0D1C0E81-B173-4BEB-917C-0CF02189FF9D](/Users/fanxudong/IdeaProjects/blog/1 数据库/assert/0D1C0E81-B173-4BEB-917C-0CF02189FF9D.png)



## 汇总数据

聚集函数：运行在行组上，计算和返回单个值的函数

![C95C73D3-CB10-446F-B87B-7808D7982CAE](/Users/fanxudong/IdeaProjects/blog/1 数据库/assert/C95C73D3-CB10-446F-B87B-7808D7982CAE.png)



## 分组数据

数据分组：把数据分为多个逻辑组，对每个分组进行聚集计算。

创建分组：

​	select **vend_id**, count(*) as num_prods from products group by **vend_id**;

​	除聚集计算语句外，select语句中的每个列都必须在group by子句中给出。

​	group by子句可以包含任意数目的列。分组可进行嵌套。

过滤分组：

​	having子句（where过滤行，having过滤分组; where在分组前，having在分组后）

​	select cust_id, count(*) as orders from orders group by custi_id having count( * ) >= 2 # having子句中使用聚集函数, select 查询的列中也可以同时使用聚集函数

分组和排序：

Select order_num, sum(quantity * item_price) as ordertotal 

from order items 

Group by order_num

Having sum(quantity * item_price) >= 50

Order by ordertotal;

select子句顺序

![402E12A4-613F-4B04-BED2-8DC5EB39DF9B](/Users/fanxudong/IdeaProjects/blog/1 数据库/assert/402E12A4-613F-4B04-BED2-8DC5EB39DF9B.png)

![8A4BCD34-22DC-48A0-9081-2ACC8CA1AFD0](/Users/fanxudong/IdeaProjects/blog/1 数据库/assert/8A4BCD34-22DC-48A0-9081-2ACC8CA1AFD0.png)



## 使用子查询 （MySQL 4.1及以后的版本）

子查询：嵌套在其他查询中的查询。

利用子查询进行过滤：子查询可以作为where子句

作为计算字段使用子查询：子查询作为计算字段

​	**SELECT** cust_name , cust_state, 

​	(**SELECT** **count**(*) **from** orders **where** orders.cust_id = customers.cust_id) **as** orders 

​	**from** customers **order** **by** cust_name ;

​	相关子查询：涉及外部查询的子查询

​	**如何构建子查询：**

![B0CD3D36-1A92-4F4D-B60B-56280857A68B](/Users/fanxudong/IdeaProjects/blog/1 数据库/assert/B0CD3D36-1A92-4F4D-B60B-56280857A68B.png)

![029586C5-4CAF-4F93-A4BD-593018E03DEC](/Users/fanxudong/IdeaProjects/blog/1 数据库/assert/029586C5-4CAF-4F93-A4BD-593018E03DEC.png)



## 连接表

外键：外键为某个表中的一列，它包含另一个表的主键值，定义了两个表之间的关系。

创建联结：

​	**SELECT** vend_name, prod_name, prod_price **from** vendors, products 

​	**where** vendors.vend_id = products.vend_id 

​	**order** **by** vend_name, prod_name;

​	使用from联结表时，用where子句对表进行联结，第一个表的每一行与第二个表中的每一行配对，where子句过滤匹配的行。

​	没有联结条件的表关系返回的结果为笛卡尔积。检索出的行的数目将是第一个表中的行数乘以第二个表中的行数。

内部联结：(和上一个sql完全相同，首选inner join语法)

​	**select** vend_name, prod_name, prod_price **from** vendors 

​	**inner** **join** products **on** vendors.vend_id = products.vend_id;

INNER JOIN 两边表同时有对应的数据，即任何一边缺失数据就不显示。
LEFT JOIN 会读取左边数据表的全部数据，即便右边表无对应数据。(可以使用is null来获取左边表不在右边表的行)
RIGHT JOIN 会读取右边数据表的全部数据，即便左边表无对应数据。



1、可以联结多个表。当需要employees.emp_no非空也输出时，需要全用left join，否则会在后面某个join把空数据过滤掉。

select employees.last_name, employees.first_name, departments.dept_name from employees 
left join dept_emp on employees.emp_no = dept_emp.emp_no
left join departments on dept_emp.dept_no = departments.dept_no

2、 获取员工其当前的薪水比其manager当前薪水还高的相关信息(SQL25): 连接salaries表两次，一次为员工工资，一次为主管工资。



## 插入数据

数据插入

​	insert ignore into actor values () 插入时如果主键已存在则忽略。

插入完整的行

插入多个行： insert into customers() values(), (); 单条insert插入多行能够提高性能。



## 创建和操纵表

表创建基础

create table customers (cust_id int not null, cust_name char(50) null, pricemary key (cust_id)) ENGINE=InnoDB

if not exists 表名不存在时才创建。

使用NULL值：create表语句中，可以指定列允许NULL值或不允许NULL值（NULL或NOT NULL）。NULL值表示没有值，空串是有效值，不是NULL值。



## 牛客上比较难的SQL题

1. 对所有员工的薪水按照salary进：

   对所有员工的薪水按照salary进行按照1-N的排名，相同salary并列且按照emp_no升序排列。

   select s1.emp_no, s1.salary, **count(distinct s2.salary)**    #使用distinct的话，使排名连续。

   from salaries s1, salaries s2 where s1.salary <= s2.salary  # 这个联结使本题的精髓
   **group by s1.emp_no**   # group by以后才能进行count计算
   order by s1.salary desc, s1.emp_no;   #多列排序

   