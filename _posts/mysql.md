# 1 mysql 数据库
- 范式（为建立冗余小、结构合理）是数据库设计遵循的规则；常见的有三个（三大范式），第一范式（确保每列保持原子性），第二范式（确保表中的每列都和主键相关），第三范式:确保每列都和主键列直接相关） 
- mysql存储引擎,Innodb: 查询速度较myisam慢,但是更安全，支持事务，支持行锁，支持外键;​ myisam 不支持事务、行锁、外键、​ memory、blackhole；
- 事务四大特性ACID（事务的基本要素），原子性（atomicity）：事务不可分割、诸操作要么都做，要么都不做；一致性（consistency）：一致性状态变到另一个一致性状态；隔离性（isolation）：并发事务直接互不干扰；持久性（durability），持久性也称永久性（permanence）；
## 线程安全问题
线程安全指的是内存的安全；主流操作系统都是多任务的，即多进程同时运行；为保证安全，每个进程分配独立的内存空间，其他进程不能访问，有操作系统保障；
在每个进程的内存空间中都会有一块特殊的公共区域，通常称为堆（内存）；
进程内的所有线程都可以访问到该区域，在堆内存中的数据由于可以被任何线程访问到，即堆内存空间在没有保护机制的情况下，对多线程来说是不安全的地方
- 操作系统级保障：操作系统会为每个线程分配属于它自己的内存空间，通常称为栈内存，其它线程无权访问，如函数内的局部变量
- 主流编程语言的规定，类的成员变量不能再分配在线程的栈内存中，而应该分配在公共的堆内存中，即位置发生改变，为了解决不安全每个线程都拷贝一份，如ThreadLocal类

## 乐观锁、悲观锁
- 悲观锁：他觉有人会修改我读的数据，那我就再查的时候，对数据进行锁定；在数据库中：select * from user where id =1 for update；在Django中：select_for_update()； 
- 乐观锁：在更新的时候会判断一下在此期间别人有没有去更新这个数据（适用于多读的场景，提高吞吐量TPS，增加并发量） data = user.objects.flilter(id=1).first()  ->  info = user.objects.filter(id=1,age=data.age).updata(age=data.age+1) -> 影响行数（info)为0，证明被人修改过；如果为1，证明数据没人动过，修改成功；
- 乐观锁，冲突检测+数据更新，经典的实现方式 CAS（Compare and Swap），但是会有ABA问题，可以通过 CAS+版本号解决（操作前拿到版本号，操作后版本号+1；并配合失败重试次数）
- 乐观锁适合查多改少，经常被并发修改的数据可能老是出现版本不一致导致有的线程操作常常失败； 悲观锁适合短事务（长事务导致其它事务一直被阻塞，影响系统性能），查少改多；
## 事务隔离级别
- 事务隔离级别：可重复读（repeatable-read）（默认）、不可重复读（read-committed）、读未提交（read-uncommitted）、串行化（serializable）；

事务隔离级别 | 脏读 |  不可重复读  |  幻读
:-:| :-: | :-: | :-: 
读未提交（read-uncommitted）  | 是  |  是  |  是
不可重复读（read-committed）  |  否  |  是  |  是
可重复读（repeatable-read）  |  否  |  否  |  是
串行化（serializable）  |  否  |  否  |  否

- 脏读：事务A读取了事务B更新的数据，然后B回滚操作，那么A读取到的数据是脏数据；
- 不可重复读：事务 A 多次读取同一数据，事务 B 在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果 不一致；--> 侧重于修改      --> 行锁解决
- 可重复读：原来的数据就算别其他的事务修改了，还是能读取到原来没有修改的数据；
- 幻读：A将所有学生的成绩修改，但B就在这个时候插入了一条记录，当A改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样，这就叫幻读； --> 侧重于新增、删除 --> 表锁解决
``` javascript
from django.db import transaction # 方法一
with transaction.atomic(): # 开启事务
    ...
```
``` javascript
@transaction.atomic # 保证在该函数中所有的数据库操作都在一个事物中
def post(self,request): # 方法二
   ...
   sid=transaction.savepoint()  # 设置事物保存点（可设多个）
   ...
   transaction.savepoint_rollback(sid)  # 如果有异常情况可 回滚 到指定的保存点
   ...
   transaction.savepoint_commit(sid)  # 如果没有异常可 提交 事务
```
#  tips MYSQL
```
SHOW DATABASES;
SHOW TABLES;  # SHOW STATUS; SHOW CREATE DATABASE; SHOW CREATE TABLE;
SHOW GRANTS;  # 显示授权用户（所有用户或特定用户）的安全权限
SHOW ERRORS;  # SHOW WARNINGS;  # 显示服务器错误和警告信息
HELP SHOW;  # 显示允许的 SHOW  语句
DESCRIBE customers; SHOW COLUMNS FROM customers;

SELECT DISTINCT vend_id FROM products; 
LIMIT 5, 3  #  LIMIT 5 OFFSET 3 # 检索从第5行开始的5行
ORDER  BY   #  DESC, ASC
WHERE vend_id <> 1003  # 不等于 ！=
WHERE prod_price BETWEEN 5 AND 10  # 包括开始和结束值
IS NULL  # 空值检查
_ 下划线匹配单个字符  % 匹配多个字符
WHERE prod_name REGEXP '.000'  # 进行正则匹配，点表示任意一个字符，\\n 匹配换行
SELECT CONCATE(RTRIM(row_1), LTRIM(row_2)) from  # 进行串拼接，并且去除左右空格  # 常用的文本处理 UPPER, LENGTH, LOWER, LOCATE, LEFT
SELECT cust_id, order_num FROM orders WHERE Date(order_date) = '2018-02-22';  # 只是取出 order_date 的日期部分进比较  # 日期时间处理 Time, Year, Second, Now, Month, Minute, Hour, DayOfWeek, Day, Date_Format, Date_Add, DateDiff, CurDate, CurTime, AddDate, AddTime
# 数值处理函数 Abs, Cos, Exp, Mod, Pi, Rand, Sin, Sqrt, Tan
SELECT '年' as '日期部分', TIMESTAMPDIFF(YEAR, '2018-02-23', CURRENT_TIMESTAMP()) as '数值'  #  SECOND, MINUTE, HOUR, DAY, WEEK, MONTH, QUARTER, YEAR
select * from * UNION ALL select * from *  # 组合结果

Aggregate Function 聚集函数  # AVG, COUNT, MAX, MIN, SUM  # 忽略 NULL 值
select vend_id, count(*) as num_prods from products where prod_price >= 10 
group by vend_id having count(*) >=2  # HAVING 和 WHERE 的区别，前者在数据分组后进行过滤，后者在数据分组前过滤
非主键索引的叶子节点存放的是主键的值，而主键索引的叶子节点存放的是整行数据，其中非主键索引也被称为二级索引，而主键索引也被称为聚簇索引。
```
子查询：由内向外, 除了in、也可与不等于<>、等于=结合
```
select order_num from orderitems where prod_id = '999';  # 20005， 20007
select cust_id from orders where order_num in (20005， 20007);  # 10001， 10004
select cust_name, cust_contact from customers where cust_id in (10001, 10004);

select cust_name, cust_contact from customers where cust_id in (
    select cust_id from orders where order_num in (
        select order_num from orderitems where prod_id = '999'));
```

相关子查询，涉及外部查询的子查询
```
select empno,ename,ename,sal,deptno from emp
where sal > (
            select  max (avg(sal)) from emp where deptno in
                    (
                    select deptno from dept
                    where loc in ('NEW YORK','CHICAGO')
                    )
            group by deptno
          );
```
等值联结，即内部联结
```
select vend_name, prod_name, prod_price  # 笛卡尔积 
    from vendors, products
        where vendors.vend_id = products.vend_id
            order by vend_name, prod_name

select vend_name, prod_name, prod_price
    from vendors inner join products 
        on vendors.vend_id = products.vend_id;
```
联结多个表
``` 二者等价
select cust_name, cust_contact
    from customers, orders, orderitems
        where customers.cust_id = order.cust_id
            and orderitems.order_num = orders.order_num
            and prod_id = '999';

select cust_name, cust_contact  # 更有效
    from customers, orders, orderitems
        where customers.cust_id = orders.cust_id
            and orderitems.order_num = order.order_num
            and prod_id = '999'
```
自联结
```
采用自联结查询：（可能更快）
select p1.prod_id, p1.prod_name
    from products as p1, products as p2
        where p1.vend_id = p2.vend_id
            and p2.prod_id = '999';
采用子查询:
select prod_id, prod_name
    from products 
        where vend_id = (sellect vend_id
                            from products
                                where prod_id = '999');
```
自然联结：每一列只返回一次；即对表使用*，其他表的列使用明确的自己(内联结有的列可能会多次出现)
```
select c.*, o.order_num, o.order_date, oi.prod_id, oi.quantity, oi.item_price
    from customers as c, orders as o, orderitems as oi
        where c.cust_id = o.cust_id
            and oi.order_num = o.order_num
            and prod_id = '999';
```
外部联结：当需要包含在相关表中没有关联的那些行(必须使用 right 或 left)
```
# 检索了所有客户，包含没有订单的客户
select customers.cust_id, orders.order_num
    from customers left outer join orders  # 筛选结果中orders列可能为NULL
        on customers.cust_id = orders.cust_id
        
# 使用带聚集函数的联结
# 检索所有客户及每个客户所下的订单数,包含没有下单的客户
select customers.cust_name, customers.cust_id, count(orders.order_num) as num_ord
    from customers left outer join orders
        on customers.cust_id = orders.cust_id
            group by customers.cust_id
```
组合查询：并（union）或复合查询（compound query）
规则：
- 必须由两条及以上的SELECT语句，语句直接用 UNION 分隔；
- 每个查询必须包含相同的列、表达式或聚集函数，可以顺序不同
- 列数据类型必须兼容
- 默认结果中重复的行被自动取消，如果要返回所有比配的行，使用 UNION ALL
- 只能使用一条ORDER BY , 在最后一条 SELECT 语句之后，作用于整个结果集
- UNION 可以极大的简化复杂的 WHERE 子句，简化多表检索数据的工作
```
select vend_id, prod_id, prod_price 
    from products 
        whree prod_price <= 5
            or vend_id in (10001, 10002);
```
全文本搜索：创建数据表时 FULLTEXT 进行指定；MYSQL 会自动维护该索引（IUD时）
- 性能：通配符LIKE和正则要尝试匹配所有行（可能没有索引），随着行数的增加，会非常耗时
- 明确控制：通配符和正则很难明确控制匹配什么和不匹配什么
- 智能化的结果：通配符和正则不能提供一种智能化的选择结果的方法
```
create table productnotes
{
    note_id int not null auto_increment,
    prod_id char(10) not null,
    note_date datetime not null,
    note_text text null,
    primary key (note_id),
    fulltext(note_text)  # 也可以逗号分隔多个
}ENGINE=MyISAM;

# Matcch() 指定被艘多的列，Against() 指定要使用的搜索表达式
select note_text
    from productnotes
        where Match(note_text) Against('rabbit');  # where note_text like '%rabbit%';
    
# 全文本搜索的一个重要部分就是对结果排序，具有较高等级的行先返回（可能就是你想要的）
select note_text,
    Match(note_text) Against('rabbit') as rank;
        from productnotes;
```
插入、更新、删除数据
```
insert into customers 
    values (null, 'a', 'b', 'c', NULL, NULL, 'd', 'e');  # 第一个是自增id， MYSQL 自己完成

insert into customers (cust_name, b, c, m, n, d, e)  # 更安全的方式，推荐（即时表结构发生变化）
    values ('a', 'b', 'c', NULL, NULL, 'd', 'e');  

update customers  # update ignore customers 忽略错误
    set cust_email = 'abc@163.com'
        where cust_id = '999';

delete from customers 
    where cust_id = '999';
```
视图：是虚拟的表，只包含使用时动态检索数据的查询；本身不包含数据
- 重用SQL语句;一次编写好基础SQL，根据需要多次使用
- 简化复杂SQL操作；编写查询后可以方便的重用，而不必知道细节
- 使用表的一部分，而不是整个表
- 保护数据，可以只授权表的特定部分的访问权限
- 更改数据格式和表示；返回与底层表的表示和格式不同的数据(联结)
```
create view viewname;
show create view viewname;
drop view viewname;

create view productcustomers as 
    select cust_name, cust_contact, prod_id
        from customers, orderrs, orderitems
            where customers.cust_id = orders.cust_id
                and orderitems.order_num = orders.order_num;

# 列出订购任意产品的客户
select * from productcustomers;
```
存储过程: 为之后使用而保存的一条或多条MYSQL语句的集合（简单、安全、高性能）
- 将处理封装在容易使用的单元内，简化复杂的操作
- 不需要反复建立一系列处理步骤，保证数据的完成性
- 简化了变动的管理；如表明、列名或业务逻辑有变化，只需要改变存储过程代码，使用它的人甚至
- 不需要知道这些边变化
- 安全性：限制了对基础数据的访问、减少了数据讹误的机会
- 提高了性能，比单独的SQL语句要快
```
create procedure productpricing()  
begin
    select avg(prod_price) as priceaverage
        from products;
end;

call productpricing();
drop procedure productpricing;  # if exists

create procedure productpricing(  # 可以有参数
    out p1ow decimal(8, 2), out phigh decimal(8, 2), out pavg decimal(8, 2))
begin
    select min(prod_price) into p1ow from products
    select max(prod_price) into phigh from products
    select avg(prod_price) into pavg from products;
end;

call productpricing(@pricelow, @pricehigh, @priceaverage);

# out 指出相应的参数用来从存储过程传出一个值，返回给调用者；in 指传递给存储过程
# inout 指对存储过程传入和传出
# into 指保存到相应额变量
# 记录集不是允许的类型，不能一个参数返回多个行和列；如：前面使用了3个参数

# 调用语句并不返回显示任何数据，它返回以后可显示（或以后其他处理中使用）的变量
select @priceaverage, @pricelow, @pricehigh


# 下面是：接受订单号并返回该订单的合计
create procedure ordertotal(
    in onumber int, 
    out ototal decimal(8, 2)
)
begin 
    select sum(item_price * quantity)
        from orderitems
            where order_num = onumber
        into ototal;
end;

call ordertotal(200099, @total);
select @total;
```
建立智能存储过程：包含业务规则和智能处理
```
# 获得订单合计，对合计增加营业税，但只针对某些顾客
create procedure ordertotal(
    in onumber int,
    in taxable boolean,
    out ototal decimal(8, 2)
) comment 'Obtain order total, optionally adding tax'
begin
    declare total decima(8, 2);
    declare taxrate int default 6;
    select sum(item_price * quantity)
        from order items
            where order_num = onumber
        into total;
        
    if taxable then
        select total + (total / 100 * taxrate) into total
    end if;
    
    select total into ototal;
end;

call ordertotal(200099, 0, @total);
select @total;

call ordertotal(200099, 1, @total);
select @total;

show create procedure ordertotal;
```
MySQL 游标只能用于存储过程（和函数）：主要用于交互式应用，滚动数据、浏览并更改
- 游标使用前，必须声明（定义），并未检索数据，只是定义要使用的SELECT语句
- 一旦声明后，必须打开游标以供使用，使用之前定义的SELECT语句将数据检索出来
- 对于填有数据的游标，根据需要取出（检索）各行
- 结束游标使用时，必须关闭游标（支持频繁的操作打开和关闭）
```
create procedure processorders()
begin
    declare done boolean default 0;
    declare o int;
    declare t decimal(8, 2);
    
    declare ordernumbers cursor
    for 
    select order_num from orders;
    
    declarecontinue handler for sqlstate '02000' set done=1;  # 直到未找到
    
    create table if not exists ordertotals  # store the results
        (order_num int, total deciaml(8, 2));
        
    open ordernumbers;
    repeat
        fetch ordernumbers into o;
        
        call ordertotal(o, 1,  t);
        
        insert into ordertotal(order_num, total)
        values(o, t);
    until done end repeat;
    
    close ordernumbers;
end;
```
触发器：在事件发生时自动执行(支持响应：DELETE, INSERT, UPDATE 语句)
- 电话号码自动检查，自动减库存，自动保存删除数据的副本等
- 创建时需要给出：唯一的触发器名、关联的表、响应的活动（DIU）、执行时间（之前还是之后）
- 只有表支持，视图和临时表不支持
- 每个表每个事件每次只允许一个触发器；即每个表最多6个触发器（IDU、之前和之后）
- MYSQL的触发器不支持（现在）CALL语句，表示不能从触发器内调用存储过程，存储过程的代码需要复制到触发器内
```
# 对每一个成功的插入，将显示 product added消息
create trigger newproduct after insert on products
    for each row select 'product added';

drop trigger newproduct;

create trigger neworder after insert on orders 
    fro each row select new.order_num;

# 在DELETE触发器代码内，可以引用一个OLD的虚拟表，访问被删除的行
create trigger deleteorder before delete on orders
for each row
begin
    insert into archive_orders(order_num, order_date, cust_id)
    values(OLD.order_num, OLD.order_date, OLD.cust_id);
end;

# 在UPDATE触发器中，可以引用一个名为OLD的虚表访问U之前的值，NEW的虚表访问U之后的内容
# OLD中的值都是只读的，不能更新
create trigger updateeventdor before update on vendors
for each row set NEW.vend_name = Upper(NEW.vend_name);
```
事务处理（transaction processing）：维护数据库的完成性，保证呈批的SQL操作要么完全执行，要么完全不执行
```
select * from ordertotals;
start transaction;
delete from ordertotals;
select * from ordertotals;
rollback;
select * from ordertotals;

# 一般的mysql语句都是直接对数据库表执行和编写的，所谓的隐含提交，即提交（写或保存）操作
# 是自动执行的；事务处理块中，提交不会隐含的进行，使用commit进行明确的提交
# 更改默认的自动提交：set autocommit = 0;
start transaction;
delete from orderitems whre order_num = 999;
delete from orders where order_num = 999;
commit;
```
添加字符集和校对：
```
show character set;  # 查看所支持的字符集
show collation;  # 查看所支持校对

create table mytable
(
    collumn1 int,
    collumn2 varchar(10)
)default character set hebrew
 collate hebrew_qeneral_ci;
```
mysql为了提高其性能，部分数据时缓存在内存中，因此要刷新表（清除缓存），就需要使用
当然，如果是需要备份数据库，同时防止备份时候有新数据写入，且备份的是最新的
```
FLUSH TABLES;
FLUSH TABLES WITH READ LOCK;
```
having 子句：筛选成组（group by）后的数据、必须要与group by，where子句在聚合前进行筛选

## 索引操作：普通索引、唯一索引、主键索引、组合索引
### 普通索引(index)
```
1、先创建一个表
create table tab1(
    nid int not null auto_increment primary key,
    name varchar(32) not null,
    email varchar(64) not null,
    extra text,
    index ix_name (name))
2、创建索引:create index 索引名称 on 表名(列名)
3、删除索引:drop 索引名称 on 表名;
4、查看索引:show index from 表名;
5、注意事项(对于创建索引时如果是BLOB 和 TEXT 类型，必须指定length。)
create index index_name on tab1(extra(32));
```
### 唯一索引(unique):增加了一层唯一约束,添加唯一性索引的数据列可以为空，但是只要存在数据值，就必须是唯一的
```
1、创建表+唯一索引
create table tab2(
    nid int not null auto_increment primary key,
    name varchar(32) not null,
    email varchar(64) not null,
    extra text,
    unique ix_name (name)  -- 重点在这里)
2、创建索引: create unique index 索引名 on 表名(列名)
3、删除索引: drop unique index 索引名 on 表名
```
### 主键索引: 是唯一索引的特殊类型，但是数据不能为空
```
1、创建表+主键索引
create table in1(
    nid int not null auto_increment,
    name varchar(32) not null,
    email varchar(64) not null,
    extra text,
    primary key(nid),
    index zhang (name))
2、创建主键: alter table 表名 add primary key(列名);
3、删除主键: alter table 表名 drop primary key;
```
### 组合索引: 将两列或者多列组合成一个索引进行查询
```
1、创建表
create table in3(
    nid int not null auto_increment primary key,
    name varchar(32) not null,
    email varchar(64) not null,
    extra text)
2、创建组合索引:create index ix_name_email on in3(name,email);
如上创建组合索引之后，查询有的会使用索引，有的不会：(最左匹配)
name and email  -- 使用索引
name                 -- 使用索引
email                 -- 不使用索引
```
- 负向查询不能使用索引，比如 status!=0 , not in / not exists 不使用索引，可以优化使用in查询, in (2, 3)
- 前导模糊查询不能使用索引, like '%xxx'; 非前导则可以， 'xxx%'
- 经验上，能过滤80%数据时就可以使用索引，即数据区分不明显的不建议创建索引，如性别、状态值
- 在属性上进行计算不能命中索引，where YEAR(date) < = '2017'，但是可以优化where date < = CURDATE()
- 如果明确知道只有一条记录返回，可以使用，username='xxxx' limit 1 提高效率，让数据库停止游标移动，停止全表扫描
- 强制类型转换会全表扫描 where phone=13800001234，虽然可以查出数据但是索引会失效，需要修改为：where phone='13800001234'
- 把计算放到业务层而不是数据库层，除了节省数据的CPU，还有意想不到的查询缓存优化效果
# 执行计划 explain 详解
id | select_type | table      | type  | possible_keys | key     | key_len | ref  | rows | Extra
:-:| :-: | :-: | :-: | :-:| :-: | :-: | :-: | :-:| :-: 
1 | PRIMARY     | <derived2> | ALL   | NULL          | NULL    | NULL    | NULL |    9 | NULL
2 | DERIVED     | tb1        | range | PRIMARY       | PRIMARY | 8       | NULL |    9 | Using where 
```
    select_type
        查询类型
            SIMPLE          简单查询
            PRIMARY         最外层查询
            SUBQUERY        映射为子查询
            DERIVED         子查询
            UNION           联合
            UNION RESULT    使用联合的结果
            ...
    table
        正在访问的表名
    type
        查询时的访问方式，性能：all < index < range < index_merge < ref_or_null < ref < eq_ref < system/const
        ALL     全表扫描，对于数据表从头到尾找一遍: select * from tb1;
                       特别的：如果有limit限制，则找到之后就不在继续向下扫描
                            select * from tb1 where email = 'seven@live.com'
                            select * from tb1 where email = 'seven@live.com' limit 1;
                            虽然上述两个语句都会进行全表扫描，第二句使用了limit，则找到一个后就不再继续扫描。
        INDEX           全索引扫描，对索引从头到尾找一遍
                        select nid from tb1;
        RANGE          对索引列进行范围查找
                        select *  from tb1 where name < 'alex';
                        PS:
                            between and
                            in
                            >   >=  <   <=  操作
                            注意：!= 和 > 符号
        INDEX_MERGE     合并索引，使用多个单列索引搜索
                        select *  from tb1 where name = 'alex' or nid in (11,22,33);
        REF             根据索引查找一个或多个值
                        select *  from tb1 where name = 'seven';
        EQ_REF          连接时使用primary key 或 unique类型
                        select tb2.nid,tb1.name from tb2 left join tb1 on tb2.nid = tb1.nid;
        CONST           常量
                        表最多有一个匹配行,因为仅有一行,在这行的列值可被优化器剩余部分认为是常数,const表很快,因为它们只读取一次。
                        select nid from tb1 where nid = 2 ;
        SYSTEM          系统
                        表仅有一行(=系统表)。这是const联接类型的一个特例。
                        select * from (select nid from tb1 where nid = 1) as A;
    possible_keys:  可能使用的索引
    key:            真实使用的
    key_len:        MySQL中使用索引字节长度
    rows            mysql估计为了找到所需的行而要读取的行数 ------ 只是预估值
    extra
        该列包含MySQL解决查询的详细信息
        “Using index”
            此值表示mysql将使用覆盖索引，以避免访问表。不要把覆盖索引和index访问类型弄混了。
        “Using where”
            这意味着mysql服务器将在存储引擎检索行后再进行过滤，许多where条件里涉及索引中的列，当（并且如果）它读取索引时，就能被存储引擎检验，因此不是所有带where子句的查询都会显示“Using where”。有时“Using where”的出现就是一个暗示：查询可受益于不同的索引。
        “Using temporary”
            这意味着mysql在对查询结果排序时会使用一个临时表。
        “Using filesort”
            这意味着mysql会对结果使用一个外部索引排序，而不是按索引次序从表里读取行。mysql有两种文件排序算法，这两种排序方式都可以在内存或者磁盘上完成，explain不会告诉你mysql将使用哪一种文件排序，也不会告诉你排序会在内存里还是磁盘上完成。
        “Range checked for each record(index map: N)”
            这个意味着没有好用的索引，新的索引将在联接的每一行上重新估算，N是显示在possible_keys列中索引的位图，并且是冗余的。
```
# 索引分类
- InnoDB的索引实现说起，InnoDB有两大类索引：聚集索引(clustered index)、普通索引(secondary index)
- MySQL 无论如何都会建起来，并且存储有完整行数据的索引，就叫聚簇索引（clustered index）（指定主键或唯一键，默认用rowid字段）
- 不带行数据完整信息的索引，就叫二级索引（secondary index），也叫辅助索引
- 复合索引，同时给多个字段建立索引
## 聚簇索引
- 如果表设置了主键，则主键就是聚簇索引
- 如果表没有主键，则会默认第一个NOT NULL，且唯一（UNIQUE）的列作为聚簇索引
- 以上都没有，则会默认创建一个隐藏的row_id作为聚簇索引
- InnoDB的聚簇索引的叶子节点存储的是行记录（其实是页结构，一个页包含多行数据），InnoDB必须要有至少一个聚簇索引。
- 由此可见，使用聚簇索引查询会很快，因为可以直接定位到行记录。
## 普通索引
- 普通索引也叫二级索引，除聚簇索引外的索引，即非聚簇索引。
- InnoDB的普通索引叶子节点存储的是主键（聚簇索引）的值，而MyISAM的普通索引存储的是记录指针
## 索引查找过程
- 如果查询条件为普通索引（非聚簇索引），需要扫描两次B+树，第一次扫描通过普通索引定位到聚簇索引的值，然后第二次扫描通过聚簇索引的值定位到要查找的行记录数据。
## 回表查询
- 先通过普通索引的值定位聚簇索引值，再通过聚簇索引的值定位行记录数据，需要扫描两次索引B+树，它的性能较扫一遍索引树更低。
### 索引覆盖
- 只需要在一棵索引树上就能获取SQL所需的所有列数据，无需回表，速度更快
- 如何实现覆盖索引：常见的方法是，将被查询的字段，建立到联合索引里去
# redo log  和 bin log
- MySQL 是按页为单位来读取数据的，一个页里面有很多行记录，从内存刷数据到磁盘，也是以页为单位来刷。
- 这时候如果掉电了，数据就没了，于是 MySQL 把你对页修改了什么内容，记录了下来，保存到磁盘，也就是redo log。
- 写完 redo log，MySQL 就认为事务提交成功了，数据持久化了（ACID的D），然后在空闲的时候，再把内存的数据刷到磁盘。
- redo log 写到磁盘，比一般的写磁盘要快，原因有：
- 一般我们写磁盘，都是「随机写」，而 redo log，是「顺序写」;MySQL 在写 redo log 上做了优化，比如「组提交」
- redo log 记录了 sql 语句以及其他 api，记录的是数据的物理变化，binlog 记录的是数据的逻辑变化
- 另外还有两个参数：
- innodb_log_file_size：设置每个 redo log 文件的大小，默认是 50331648 byte，也就是 48 MB
- innodb_log_files_in_group：设置 redo log 文件的数量，默认是 2，最大值是 100
- redo log 只在使用了 innodb 作为存储引擎的 MySQL 上才有，而 binlog，只要你是 MySQL，就会有
- binlog 还有另一个作用：主从复制，主库把 binlog 发给从库，从库把 binlog 保存了下来，然后去执行它，这样就实现了主从同步。
- 当然，我们还能让自己的业务应用，去监听主库的 binlog，当数据库的数据发生变动时，去做特定的事情，比如进行数据实时统计
## 两阶段提交(redo log 要分两个阶段： prepare 和 commit)
- 写入：redo log（prepare）
- 写入：binlog
- 写入：redo log（commit）
### 1、先写 redo log，再写 binlog
这样会出现 redo log 写入到磁盘了，但是 binlog 还没写入磁盘，于是当发生 crash recovery 时，恢复后，主库会应用 redo log，恢复数据，但是由于没有 binlog，从库就不会同步这些数据，主库比从库“新”，造成主从不一致
### 2、先写 binlog，再写 redo log
跟上一种情况类似，很容易知道，这样会反过来，造成从库比主库“新”，也会造成主从不一致
### 而两阶段提交，就解决这个问题，crash recovery 时：
- 如果 redo log 已经 commit，那毫不犹豫的，把事务提交
- 如果 redo log 处于 prepare，则去判断事务对应的 binlog 是不是完整的
- 是，则把事务提交,否，则事务回滚; 两阶段提交，其实是为了保证 redo log 和 binlog 的逻辑一致性。
#  一个mysql的查询
一张学生成绩表，求出每个学生第一名的次数
![mysql-tips-example1.png](https://github.com/Django-27/workspace/blob/master/pic/mysql-tips-example1.png)

# mysql 在 select 中使用 case 语句(更好的形式效果)
![image](https://github.com/Django-27/workspace/raw/master/pic/mysql_case.png)
```
select o.id as '订单编号', o.add_time as '下单时间', ROUND(o.price_total / 100) as '订单金额(元)',
  (case p.bank_return_card_type
			when '0' then '零钱' 
			when '1' then '储蓄卡' 
			when '2' then '信用卡'
			when '4' then '花呗 '	end) as '支付类型'
		from order_order as o, pay_record as p
			where o.status = 120 and o.pay_platform = 20 and o.dealer_id = 111 
					and o.add_time > '2018-03-21 00:00:00' and o.id = p.order_id
					and p.bank_return_card_type <> -1;


select *,
    (CASE WHEN age>=60 THEN '老年' 
          WHEN age<60 AND age>=30 THEN '中年' 
          WHEN age<30 AND age>=18 THEN '青年' 
          ELSE '未成年' 
    END) as age_text
    from user
```
- 在高级语言中，CASE的可以用IF来替代，但是在SQL中不行
- CASE是SQL标准定义的，IF是数据库系统的扩展
- CASE可以用于SQL语句和SQL存储过程、触发器，IF只能用于存储过程和触发器
- 在SQL过程和触发器中，用IF替代CASE代价都相当的高，相当的麻烦，难以实现

#### (2)小记：
- 打包到rar，之后解压
- sudo tar -cvf new-dev-nginx-conf nginx.conf sites-enabled sites-available
- tar -xvf new-dev-nginx-conf #  对归档到这个文件夹下的进行解压（‘归档’）

# MySQL – How to get Top N rows for each group

- 通用的解法就是按组排序, 引入组内行, 号再取行号 < N 的记录即可
- MySQL 不支持 ROW_NUMER() 函数，它可以拿到分组内的行号，Oracle、PostgreSQL 或SQL Server支持，但有待考证
- mysql 中 show variables 等效于 show session variables（线程内），还有 show global variables（全局）
- session variables 可以不提前定义，可以直接使用 
- MySQL doesn't support ROW_NUMBER but you can use variables to emulate it
- "@"是:局部变量声明，如果没有"@"的字段代表是列名；
- @@error 等是全局变量，系统自定义的，我们只读，不能改！！

```
@qiyuan := barcode
# 上面的语句对每一行执行，并存储对应的 barcode 到 @qiyuan 变量中

@qiyuan_rank := if (@qiyaun = barcode, @qiyuan_rank + 1, 1)
# 上面的语句：如果 @qiyuan 与 barcode 的值相等，则增加 @qiyuan_rank 变量值；否则设置变量 @qiyuan_rank 为 1
# 之后 select 中要借用这个字段，所以 as 也是不可少的

# 测试通过的代码
select id, dealer_id, price_total
    from (
        select id, dealer_id, price_total
            ,@dealer_rank := if(@dealer_id = dealer_id, @dealer_rank + 1, 1) as dealer_rank
            ,@dealer_id := dealer_id
            from order_order, (select @dealer_id:=0, @dealer_rank:=0) as tmp  # 如果没有初始化是有问题的，必须
            order by dealer_id, price_total desc
        ) as ranked
    where dealer_rank <= 2


#  扩展了解, 使用 row_number()
select name, haircolor, score
    from 
        (select name, haircolor, score
                row_number() over (partition by haircolor order by score desc) as girl_rank
            from girls
        ) as ranked
    where girl_rank <= 2;
```
#### （2）tips： 是否能够进行优化（未完成）
![image 2018-03-30](https://github.com/Django-27/workspace/blob/master/pic/mysql-get-topn1.png)
#### （3）补充解释：
- ranked 内部进行内连接，记录总数没有变化

![image 2018-03-30 02](https://github.com/Django-27/workspace/blob/master/pic/mysql-get-topn2.png)

# 8 MySQL 查询
### 1 分组统计
先锋支付 | 原生支付 | 现金支付 | 合计
---|--- | --- | ---
883 | 0 | 2366 | 3249

```
方法一：太繁琐
-- select   count(*) from order_order where add_time > '2017-10-29' and deal_time < '2017-10-30' and pay_platform = 20;
-- select   count(*) from order_order where add_time > '2017-10-29' and deal_time < '2017-10-30' and pay_platform = 10;
-- select   count(*) from order_order where add_time > '2017-10-29' and deal_time < '2017-10-30' and pay_platform = 0;
-- select   count(*) from order_order where add_time > '2017-10-29' and deal_time < '2017-10-30' ;
方法二：group by 之前筛选掉了为 NULL 的数据
-- SELECT pay_platform , count(*) AS total from order_order where add_time > '2017-10-29' and deal_time < '2017-10-30'  GROUP BY pay_platform;
方法三：
SELECT 
	SUM( pay_platform = '20') as 先锋支付, 
	SUM( pay_platform = '10') as 原生支付, 
	SUM( pay_platform = '0') as 现金支付,
	COUNT(*) as 合计
FROM order_order
WHERE add_time > '2017-10-29' and deal_time < '2017-10-30';

```
### 2 格式化时间并统计
total | day
--- | --- 
8 |	2017-08-17 
3 |	2017-08-25
```
SELECT count(*) as total , DATE_FORMAT(add_time,'%Y-%m-%d') as day from order_order where pay_by = 20 and pay_platform = 10 GROUP BY DATE_FORMAT(add_time, '%Y-%m-%d');
如果是日期格式，直接用 year 可以取到年，eg: SELECT year(add_time), month(add_time), day(add_time), add_time from order_order limit 1;
```
建了一张order_order_copy表，导入数据有外键约束，可以删除这个临时表的外键约束，因为复制表结构的时候带上了了外键约束；亦可以 SET FOREIGN_KEY_CHECKS=0;  禁用外键约束。

```
SELECT count(*) , sum(amount) / 100 from pay_withdrawalsrecord where add_time > '2017-10-1' and end_time < '2017-11-1'  and status = 30 and bank_card_ownername != '*达' and  bank_card_ownername != '*博';
SELECT  * from pay_withdrawalsrecord where add_time > '2017-10-1' and end_time < '2017-11-1'  and status = 30   GROUP BY bank_card_bankname;
```
# mysql排序的一个小问题

### 问题
有一张表record，里面有一个字段是status表现记录的状态，现在根据status进行排序:
```
select * from record order by status desc limit 0,20;
select * from record order by status desc limit 20,20;
```
发现两个分页的集合中有重复数据出现。

### 解决办法
造成这个现象的原因是因为status本身是一个有很多重复值的字段，当使用该字段排序并使用了Limit时，同值的记录排序是不定的。所以在分页的时候就出现了问题。
mysql官方文档中有详情的一个例子: [mysql官方文档](https://dev.mysql.com/doc/refman/5.7/en/limit-optimization.html)

那么现在我暂时使用的解决办法就是:
```
select * from record order by status desc, id desc limit 0,20;
select * from record order by status desc, id desc limit 20,20;
```
在排序后面加上同时使用id排序，因为id是唯一的，就不会出现此问题。

# mysql 改密码的一次尝试
```
update user set plugin='mysql_native_password' where user='root'; #更改加密方式
alter user 'root'@'localhost' IDENTIFIED BY '123456';#设置密码
FLUSH PRIVILEGES;
```
# 基本操作
```
ALTER TABLE testalter_tbl DROP i;  #  删除字段
ALTER TABLE testalter_tbl ADD i INT;  # 增加字段
ALTER TABLE testalter_tbl ADD i INT FIRST;  #增加字段并制定位置
ALTER TABLE testalter_tbl ADD i INT AFTER c;

修改字段类型及名称
修改字段类型
例如，把字段 c 的类型从 CHAR(1) 改为 CHAR(10)，可以执行以下命令:
ALTER TABLE testalter_tbl MODIFY c CHAR(10); # 修改字段类型，字段名
ALTER TABLE testalter_tbl MODIFY j BIGINT NOT NULL DEFAULT 100;

ALTER TABLE testalter_tbl ALTER i SET DEFAULT 1000; # 删除默认值
ALTER TABLE testalter_tbl ALTER i DROP DEFAULT;
ALTER TABLE testalter_tbl RENAME TO alter_tbl; # 修改表名
```
## 修改默认时区
```javascript
select now(); 查看 MySql 系统时间。和当前时间做对比
set global time_zone = '+8:00';设置时区更改为东八区
flush privileges; 刷新权限
```
## 查看所有连接进程
```
show full processlist  看一下所有连接进程，注意查看进程等待时间以及所处状态 是否locked
```
## 创建一个只读权限的用户
```
CREATE USER 'select'@'%' IDENTIFIED BY '123456';# 创建查询用户、允许外网访问
GRANT SELECT ON * . * TO 'select'@'%';# 给新用户赋予查询权限
FLUSH PRIVILEGES;
```
## update 会锁表吗
### mysq由于默认是开启自动提交事务，所以首先得查看自己当前的数据库是否开启了自动提交事务。
```
select @@autocommit;
set autocommit = 0;设置为不开启自动提交
```
1.没有索引
运行命令：begin;开启事务，然后运行命令：update tb_user set phone=11 where name="c1";修改，先别commit事务。
再开一个窗口，直接运行命令：update tb_user set phone=22 where name="c2";会发现命令卡住了，但是当前面一个事务通过commit提交了，命令就会正常运行结束，说明是被锁表了。
2.给name字段加索引
create index index_name on tb_user(name);
然后继续如1里面的操作，也就是一个开启事务，运行update tb_user set phone=11 where name="c1"；先不提交
然后另一个运行update tb_user set phone=22 where name="c2";发现命令不会卡住，说明没有锁表
但是如果另一个也是update tb_user set phone=22 where name="c1";更新同一行，说明是锁行了
3.总结
如果没有索引，所以update会锁表，如果加了索引，就会锁行
### 合理建立索引
　　1.搜索的索引列 
　　最适合索引的列是出现在where子句种的列，或连接子句中指定的列。 而不是select关键字后的选择列表的列。
　　2.使用唯一索引
　　索引的列的基数越大，索引效果越好。比如id每行都不同，出生日期也不太相同，适合作索引。 而性别男或女只有两者情况，即使加索引，不管搜索哪个值都会得出大约一半的行。
　　3.使用短索引 
　　例如对字符串列进行索引，应该制定一个前缀长度。比如一个CHAR(200)的列，如果前10或20个字符内就大不相同，则可以只对前10个或20个字符进行索引。节省大量索引空间，也使查询更快，涉及的磁盘IO较少。较短的键值，索引高速缓存中的块能容纳更多的键值。
　　4.不要过度索引
　　每个额外的索引都要占用额外的磁盘空间，降低写操作的性能。因为在写修改表的内容时，索引必须进行更新，有时可能需要重构。 就比如一本字典，加入新的字必须要根据目录插入指定位置，后面的内容都要更新。
　　5. 选择最常作为访问条件的列作为主键
　　InnoDB有一个默认的保存顺序，按照主键或内部列进行访问的速度是最快的（比唯一索引还要快）。