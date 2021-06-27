###  mysql

#### 唯一、外键约束

```mysql
# 约束
primary key、unique、not null、default、foreign key
添加主键：
		alter table 表名 add constraint 约束名 primary key (列名)
添加唯一约束：
		alter table 表名 add constraint 约束名 unique （列名）
添加外键约束：
		alter table 表1 add constraint 外键名 foreign key（列名） references 表2（列名）
删除外键：
		alter table 表名 drop foreign key 外键名
```

#### 远程服务器

```mysql
# 远程服务器
mysql -h 10.0.1.99 -u root -p
```

#### 数据表

```mysql
# 数据表
show tables
desc 表名
show columns from 表名
describe 表名
show create table 表名
```

#### 根据已有表创建新表

```mysql
create table tab_new like tab_old (使用旧表创建新表)
create table tab_new as select col1,col2… from tab_old definition only
create table 新表 select * from 旧表 where...  # 对旧表的记录进行快照，并存储为新表
```

#### 修改表名

```mysql
# 修改表名
alter table 表名 rename [to] 新表名;
rename table 表名 to 新表名;
```

#### 修改字段名

```mysql
# 修改字段名
alter table 表名 change 原名 新名 类型及约束;  # 重命名修改字段
alter table 表名 modify 列名 类型及约束;  # 不重命名修改字段
```

#### 增加字段

```mysql
# 知道插入的字段放在哪里
alter table student add score double(4,1) first;
alter table student add score double(4,1) after age; -- 可以使用after，不可以使用before
```

#### 插入

```mysql
# 插入
insert into 表名 values(值1,...), (值2,...)  # 主键默认值可使用三种方式：0、null、default，非主键使用默认值必须使用default
insert into 表1(列1,...) select 列1,... from 表2;  # 列名不需要相同
insert into 新表（列1，列2）select 列1，列2 from 旧表 where...  # 在新表中写入查询结果
```

#### 插入、替换、更新、忽略

```mysql
# 插入、替换
replace into 表名 (列1,...) values (值1,...)  # 若id记录不存在，插入，若存在，删除，再插入
# 插入、更新
insert into 表名 (列1,...) values (值1,...) on duplicate key update 列1=值1，列2=值2。# 若id记录不存在，插入，若存在，更新
# 插入、忽略
insert ignore into 表名 (列1,...) values (值1,...)  # 若id记录不存在，插入，若存在，忽略
```

#### 复制数据到新表

```mysql
# 复制数据到新表
create table 新表 select * from 旧表 where...  # 对旧表的记录进行快照，并存储为新表
```

#### 写入查询结果到新表

```mysql
# 写入查询结果到新表
insert into 新表（列1，列2）select 列1，列2 from 旧表 where...  # 在新表中写入查询结果
```

#### 删除

```mysql
# 删除
delete from 表名 where...
truncate table 表名
# 和select类似：删除t1表中，跟t2匹配不上的那些记录。
delete t1 from t1 left join t2 on t1.id=t2.id where t2.id is NULL;
```

#### +运算符

```mysql
-- 对于"+"运算符，仅用于数字类型的相加。若运算数为字符，尝试转化为数字，若转换失败，则认为是0;若运算数为null，结果为null
SELECT '123'+9;  # 结果为 132，'123'-->123
SELECT 'haha'+9;  # 结果为 9，’haha’-->0
SELECT null+9;  # 结果为 null
```

#### 模糊查询

```mysql
# 模糊查询
like 	%	 _  #  %表示任意多个任意字符，可以是0个    _表示一个任意字符，总是1个 
```

#### 精准查询大小写

```mysql
select * from emp where BINARY job='MANAGER'-- 想要精准的查大小写的话，要在前面加上关键词：binary
```

#### 范围查询

```mysql
# 范围查询
in		between and
```

#### exists查询

```mysql
# exists查询
EXISTS()查询是将主查询的结果集放到子查询中做验证，根据验证结果是true或false来决定主查询数据结果是否得以保存
select * from A where exists(select 1 from b where B.id  = A.id);
以上查询等价于：
1、SELECT * FROM A;
2、SELECT 1 FROM B WHERE B.id = A.id;

例如：查找未分配具体部门的员工的所有信息
select e.* from employees as e left join dept_emp as de on e.emp_no=de.emp_no 
where de.emp_no is null
等价于
select e.* from employees as e where not exists 
(select 1 from dept_emp as de where de.emp_no = e.emp_no)
```

#### 聚合函数

```mysql
# 聚合函数
# 1、count(*)对表中行的数目计数，不管是否为null
# 2、count(column)对column列中具有值的行进行计数，忽略null
聚合函数不能跟普通字段和单行函数一起使用
where子句不能使用聚合函数（分组函数）
select deptno, avg(sal)
    from  emp
		where avg(sal) >2000   -- where子句不能使用多行函数（分组函数）
    group by  deptno
```

#### 日期、时间

```mysql
# 日期、时间
timestampdiff(时间类型, 日期1, 日期2)：日期1大于日期2，结果为负，日期1小于日期2，结果为正。
timestampdiff(day,'2019-05-20', '2019-05-21')  # 1
timestampdiff(hour, '2015-03-22 07:00:00', '2015-03-22 18:00:00')
timestampdiff(second, '2015-03-22 07:00:00', '2015-03-22 7:01:01')

datediff(日期1, 日期2)：得到的结果是日期1与日期2相差的天数。
datediff('2007-12-31','2007-12-30');   # 1
datediff('2010-12-30','2010-12-31');   # -1
```

#### 拼接

```mysql
# 拼接
select concat(trim(列1), '-', trim(列2)) as 别名 from 表
```

#### 分组

```mysql
# 分组
group by单独使用时，只显示出每组的第一条记录
# group by + group_concat()：输出分组集合
select gender,group_concat(name) from students group by gender
# group by + having  having作用和where一样，但having只能用于group by
# group by + with rollup  with rollup的作用是：在最后新增一行，来记录当前列里所有记录的总和
```

#### 分页查询（部分行）

```mysql
select * from 表名 limit 开始, 条目数  # 从start开始，获取count条数据
select * from 表名 limit 条目数 offset 开始

例如：每5行一页，返回第2页的数据
SELECT * FROM employees LIMIT 5, 5  # 第一条记录序号为0
SELECT * FROM employees LIMIT 5 OFFSET 5
```

#### 连接查询

```mysql
A、内连接
# 1、等值连接
select e.last_name,e.job_id,j.job_title from employees e,jobs j where e.job_id = j.job_id;  # sql92标准
select e.last_name,e.job_id,j.job_title from employees e inner join jobs j on e.job_id = j.job_id;  # sql99标准

# 2、非等值连接
select e.last_name,j.grade_level from employees e,job_grades j where e.salary between j.lowest_sal and j.highest_sal;  # sql92标准
select e.last_name,j.grade_level from employees e inner join job_grades j on e.salary between j.lowest_sal and j.highest_sal;  # sql99标准

# 3、自连接
select e1.last_name 'employee',e2.last_name 'manager' from employees e1,employees e2 where e1.manager_id = e2.employee_id;  # sql92标准
select e1.last_name 'employees',e2.last_name 'manager' from employees e1 inner join employees e2 
on e1.manager_id = e2.employee_id;  # sql99标准

B、外连接
# sql99标准
left join、right join

C、交叉连接
# select ... from tab1, tab2 ——> 返回的是笛卡尔积，第一个表的每一行与第二个表的每一行配对
# select ... from tab1 cross join tab2 ——> 返回的是笛卡尔积
select empno,ename,job,dname,loc,e.deptno from emp e,dept d where e.deptno=d.deptno; -- 公共字段一定要加上所属的表名
```

#### 子查询

```mysql
# 子查询
A、标量子查询: 子查询返回的结果是一个数据(一行一列)
				select * from students where age > (select avg(age) from students);
B、列子查询: 返回的结果是一列(一列多行)
				in、not in：等于/不等于列表中的任意一个
　　		 any/some：子查询中某一个值满足就行
　　		 all：子查询中所以值都满足
　　		 
				select name from classes where id in (select cls_id from students);
				-- 子查询的结果是多条记录，此时不能之间使用=，<  >  需要借助 any  all  in来实现
				-- 查询工资低于任何一个“CLERK”的工资的雇员信息。 
        -- select sal from emp where job ='CLERK'  -- 1300  1100  950  800
        select * from emp where sal< (select max(sal) from emp where job ='CLERK');
        select * from emp where sal< any(select sal from emp where job ='CLERK');

        -- 查询[工资比所有的“SALESMAN”都高的]雇员的编号、名字和工资。
        select max(sal) from emp where job ='SALESMAN';
        -- ALL  >ALL
        select empno,ename,sal from emp  where sal >(select max(sal) from emp where job ='SALESMAN');
        select empno,ename,sal from emp  where sal > ALL(select sal from emp where job ='SALESMAN');
        
C、行子查询: 返回的结果是一行(一行多列)
		# 将多个字段合成一个行元素
		select * from students where (height,age) = (select max(height),max(age) from students);
D、表子查询(结果集有多行多列)

E、相关子查询：外部查询的子查询
    select cust_name, cust_state, (select count(*) from orders where orders.cust_id = customers.cust_id) as orders from customers order by cust_name;

几种情况：
a、子查询在SELECT后面
		只支持标量子查询
		SELECT * , (SELECT COUNT(1) FROM employees 
                WHERE employees.department_id = departments.department_id) FROM departments;
b、子查询在FROM后面
　　支持表子查询，在一个SELECT查询后的表中查询新的内容    
  #查询每个部门的平均工资的工资等级
		SELECT department_id,department_name,a , j.grade_level 
		FROM (
        SELECT e.department_id ,d.department_name, AVG(salary) a 
        FROM employees e 
        LEFT JOIN departments d 
        ON  e.department_id=d.department_id 
        GROUP BY e.department_id) tmp 
		INNER JOIN job_grades j ON tmp.a BETWEEN j.lowest_sal AND j.highest_sal;
c、子查询在WHERE/HAVING后面
		支持标量子查询、列子查询、行子查询
		#标量子查询：查询工资最少的员工信息
		SELECT * FROM employees WHERE salary = (SELECT MIN(salary) FROM employees);
		#列子查询：查询location_id为1400或1500或2700的部门中的所有员工姓名
		SELECT last_name FROM employees WHERE department_id IN 
		(SELECT department_id FROM departments WHERE location_id IN(1400,1500,2700));
		#行子查询：查询编号最小并且工资最高的员工信息
    SELECT * FROM employees WHERE 
    (employee_id,salary) = (SELECT MIN(employee_id),MAX(salary) FROM employees);
d、子查询在EXISTS后面
		支持相关子查询
		相关子查询是先执行外查询，再由EXISTS过滤; 
		能用IN代替
		exists(select 语句)：有记录，则为1，无记录，则为0
    #查询有员工的部门名
    SELECT department_name FROM departments d WHERE 
    EXISTS(SELECT * FROM employees e WHERE e.department_id = d.department_id);
		#用IN代替
    SELECT department_name FROM departments d WHERE d.department_id IN (
       SELECT DISTINCT department_id FROM employees);
```

#### any、some、all

```mysql
-- not in 是 “<>all”的别名，用法相同。
-- 语句in 与“=any”是相同的。
-- 语句some是any的别名，用法相同。
```

#### 集合查询union

```mysql
# 集合查询
# UNION 运算符通过组合其他两个结果表（例如 TABLE1 和 TABLE2）并消去表中任何重复行而派生出一个结果表。当 ALL 随 UNION 一起使用时（即 UNION ALL），不消除重复行。两种情况下，派生表的每一行不是来自 TABLE1 就是来自 TABLE2。 
将多个查询的结果组合到一个结果集中
1、select的列名一致，顺序无所谓
2、n条select语句中间有n-1个*union*
3、多条select语句中，重复的行默认取消，可使用*union all*包含重复行
4、对结果集排序只能使用一个*order by*，位于最后一个select后面

select vend_id, prod_id, prod_price
from products
where prod_price <= 5
union
select vend_id, prod_id, prod_price
from products
where vend_id in (1001, 1002)
order by vend_id, prod_price;
```

#### 完整的查询

```mysql
# 完整的select语句
select distinct *
from 表名
join on 表2
where ....
group by ... having ...
order by ...
limit start,count

# 执行顺序
from 表名
join on 表2
where ....
group by ...
select distinct *
having ...
order by ...
limit start,count

# having 与 where
where是从表里进行字段的删选，某些计算出来的列数据，不能用where进行筛选
having是从select里进行字段的筛选，某些表里有但是没有select出来的字段，不能使用having

select product_id,"2018" report_year,(datediff(if(period_end<"2019-01-01",period_end,date("2018-12-31")),if(period_start>"2018-01-01",period_start,date("2018-01-01")))+1)*average_daily_sales as total_amount from Sales having total_amount>0  -- 不能使用where

```

#### 存储过程

```mysql
# 1、使用一个数据库
use 数据库名
# 2、改变默认的分隔符
delimiter $$;  -- 默认分隔符';' ——> '$$'
# 3、创建
create procedure 存储过程名(参数)
begin
select语句;
end;
# 4、调用
call 存储过程名(参数);
```

#### 视图

```mysql
# 视图
定义： create view 视图名 as select ...
查看： show tables
使用： select * from 视图名
删除： drop view 视图名
```

#### 事务

```mysql
# 事务
开启： begin / start transaction  # 开启事务后执行修改命令，变更会维护到本地缓存中，而不维护到物理表中
提交： commit  # 将缓存中的数据变更维护到物理表中，如果`COMMIT`语句执行失败了，整个事务也会失败
回滚： rollback  # 放弃缓存中变更的数据
```

#### 索引

```mysql
# 索引
创建： create index 索引名 on 表名（列名（长度））  # 字段是字符串，需要指定长度
			create index 索引名 on 表名（列1，列2）
			create unique index on 表名（列名）
查看： show index from 表名
删除： drop index 索引名 on 表名
```

#### 强制索引

```mysql
新建：
		create index 索引名称 on 表名(字段名称(长度))
		alter table students add index idx_class_id (class_id);
强制使用索引：
		select * from students force index (idx_class_id) where class_id = 1
```

#### 触发器

```mysql
# 触发器
创建： create trigger 触发器名 
			触发时机 触发事件  				# 触发时机：before、after  触发事件：insert、update、delete
			on 表名 for each row
			begin
					执行语句
			end
查看： show triggers\G
删除： drop trigger [if exists] [schema_name.]触发名
```

####  replace into

```mysql
# replace
replace into 表（列...） values （值...）
update 表 set 列=replace(列，旧值，新值)
```

#### 统计字符串中子字符串出现的次数

```mysql
select (length("10,A,B")-length(replace("10,A,B",",","")))/length(",") as cnt
```

#### 查看数据表名、字段名

```mysql
# 查看数据库中的数据表
select table_name from information_schema.tables where table_schema='数据库名'
# 查看数据表的字段
select column_name from information_schema.columns where table_schema='数据库名' and table_name='数据表名'

例如：为数据库中的表生成一列指令
select concat('select count(*) from ', table_name, ';') from information_schema.tables where table_schema='nowcoder'
```

#### case when then end

```mysql
语法：
case  参考的数
   when 参考的数的值1 then 得到的值1
   when 参考的数的值2 then 得到的值2
   ...
   else commands
end

例如：计算满意度
sum(case when 是否满意='是' then 1 else 0 end) / count(是否满意) as 满意度
sum(if(是否满意='是',1,0)) / count(是否满意) as 满意度
```

#### if()、ifnull()、isnull()、nullif()

```mysql
ifnull(expression_1,expression_2);
# 如果expression_1不为NULL，则函数返回expression_1; 否则返回expression_2的结果

if(expr1,expr2,expr3)
# 如果expr1的值为true，则返回expr2的值，否则返回expr3的值

isnull(expr)
# expr 为null，那么isnull() 的返回值为 1，否则返回值为 0

nullif(expr1,expr2)
# 如果expr1= expr2 成立，那么返回值为NULL，否则返回值为 expr1
```

#### null与''

```mysql
1、空值不占空间，NULL值占空间。当字段不为NULL时，也可以插入空值。

2、当使用 IS NOT NULL 或者 IS NULL 时，只能查出字段中没有不为NULL的或者为 NULL 的，不能查出空值。

3、判断NULL 用IS NULL 或者 is not null,SQL 语句函数中可以使用IFNULL()函数来进行处理，判断空字符用 =''或者<>''来进行处理。

4、在进行count()统计某列的记录数的时候，如果采用的NULL值，会别系统自动忽略掉，但是空值是会进行统计到其中的。
```

#### 窗口函数

```mysql
select *, dense_rank() over(partition by 课程号
                       order by 成绩 desc rows/range子句) as排名
from 成绩表;

rank()：如果有并列名次的行，会占用下一名次的位置。
				比如正常排名是1，2，3，4，但是现在前3名是并列的名次，结果是：1，1，1，4
dense_rank()：如果有并列名次的行，不占用下一名次的位置。
							比如正常排名是1，2，3，4，但是现在前3名是并列的名次，结果是：1，1，1，2。
row_number()：不考虑并列名次的情况，排名是正常的1，2，3，4
```

```mysql
unbounded preceding: 区间的第一行
N preceding: 当前行之前的N行
current row: 当前行
unbounded following：区间的最后一行
N following：当前行之后的N行

select `id`, 
sum(`id`) over(order by `id`) as `default_sum`,
sum(`id`) over(order by `id` rows between unbounded preceding and current row) as `rows_unbounded_sum`,
sum(`id`) over(order by `id` range between unbounded preceding and current row) as `range_unbounded_sum`,
sum(`id`) over(order by `id` rows between 1 preceding and 1 following) as `rows_sum`,
sum(`id`) over(order by `id` range between 1 preceding and 1 following) as `range_sum`
from window_func_test
```

![image-20200518145409442](https://gitee.com/louisyanglu/images/raw/master/20210627234241.png)

#### 求累计值cumsum()

```mysql
SET @csum := 0;
SELECT 日期, 净利润, (@csum := @csum + 净利润) AS 累计利润
FROM daily_pnl_view;

select 
    player_id,
    event_date,
    sum(games_played) over(partition by player_id order by event_date) games_played_so_far
from Activity;
```

```mysql
SELECT 
    Id, Company, Salary
FROM
    (SELECT 
        e.Id,
        e.Salary,
        e.Company,
        IF(@prev = e.Company, @Rank:=@Rank + 1, @Rank:=1) AS `rank`,@prev:=e.Company
    FROM
        Employee e, (SELECT @Rank:=0, @prev:=0) AS temp
    ORDER BY e.Company , e.Salary , e.Id) Ranking
        INNER JOIN
    (SELECT 
        COUNT(*) AS totalcount, Company AS name
    FROM
        Employee e2
    GROUP BY e2.Company) companycount ON companycount.name = Ranking.Company
WHERE 
    `rank` = FLOOR((totalcount + 1) / 2) OR `rank` = FLOOR((totalcount + 2) / 2)
```

#### 组内排名

```mysql
# 先按照公司、工资进行排序，然后设置变量，组内rank值依次+1
SELECT 
        e.Id,
        e.Salary,
        e.Company, 
        IF(@prev = e.Company, @Rank:=@Rank + 1, @Rank:=1) AS `rank`,
        @prev:=e.Company
    FROM
        Employee e, (SELECT @Rank:=0, @prev:=0) AS temp order by e.Company, e.Salary, e.Id
```

#### 单行函数

##### 字符函数

```mysql
length(str)、concat(str1,str2...)、upper(str)、lower(str)、replace(str,old,new)、left(str, n)、right(str, n)
concat(str...)  -- 如果连接的str中包含null，结果为null
concat_ws(x, str1, str2)  -- 指定分隔符连接字符串，如果连接的str中包含null，结果 不为null
substr(str,pos,len)  -- 截取子串，索引从1开始
instr(str,substr)  -- 返回子串第一次出现的索引，字符串索引从1开始
trim(str,substr)  -- 首尾去除规定字符，默认去空格
trim(substr from str)
								select trim('7' from '77my  name 777 is  xiaoming  777');
lpad(str,len,padstr)、rpad(str,len,padstr)  -- 用规定字符左(右)填充至指定长度
repeat(str, n)  -- 字符串重复指定次数
space(n)  -- 返回n个空格
strcmp(str1, str2)  -- 比较字符串，大于返回1，小于返回-1，等于返回0
reverse(str)  -- 反转字符串
elt(n, str1, str2...)  -- 从一堆str中取第n个字符串，从1开始
```

##### 数学函数

```mysql
round(x,d)、ceil(x)、floor(x)、abs()、pi()、rand()
sign(x)  -- 返回x的符号，x为负数、0、正数返回-1、0、1
truncate(x,d)  -- 截取保留指定小数位
mod(x1,x2)  -- 取模，负号跟随x1
						mod(-10,3);#-1
						mod(10,-3);#1
求最大、最小值
least(10, 20)
greatest(10, 20)
```

##### 日期函数

```mysql
now()、curdate()、curtime()
year(str)、month(str)、day(str)、hour(str)、minute(str)、second(str)
monthname(str)、dayname(str)  -- 返回日期中 月份、日期 的名称，如January、Monday
dayofweek(str)  -- 一周的第几天，1代表周日
weekday(str)  -- 星期几，0代表周一
week(str)  -- 一年的第几个星期
datediff(date1, date2)
timediff(time1, time2)
str_to_date(str,format)  -- 字符串按格式转为日期
date_format(date,format)  -- 日期按格式转为字符串
				SELECT str_to_date('10-1 2019','%c-%d %Y');#2019-10-01
				SELECT date_format(now(),'%Y.%m.%d');#2019.10.19
```

![image-20200517134915056](https://gitee.com/louisyanglu/images/raw/master/20210627234250.png)

##### 流程控制函数

```mysql
if()：二选一
case when：多选一
```

#### 自定义函数

```mysql
创建：
CREATE FUNCTION 函数名(参数列表) RETURNS 返回类型
BEGIN
	函数体
END

调用：
SELECT 函数名(参数列表)
```

```mysql
# 将时间转化成指定格式的字符串
CREATE FUNCTION my_date_format(p_date date) RETURNS CHAR(10)
 BEGIN
   IF p_date is null THEN
    RETURN NULL;
   END IF;
    RETURN date_format(now(), '%Y-%m-%d');
 END;
 
 # 第N高的工资
 CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  RETURN (     
  SELECT  IF(count<N,NULL,min) 
  FROM
    (SELECT MIN(Salary) AS min, COUNT(1) AS count
    FROM
      (SELECT DISTINCT Salary
      FROM Employee ORDER BY Salary DESC LIMIT N) AS a
    ) as b
  );
END
```

#### 一行转多列

```mysql
-- substring_index的作用:取得目标字符串左侧第n个分割符左侧的部分，n为负时返回右侧第n个的右部分
-- help_topic 是数据库mysql的一个表，该表提供查询帮助主题给定关键字的详细内容
select substring_index('a,b,c,d', ',', topic.help_topic_id+1) as new_col, topic.help_topic_id from mysql.help_topic as topic 
where topic.help_topic_id<=length('a,b,c,d')-length(replace('a,b,c,d', ',', ''))
```

![image-20200521223633996](https://gitee.com/louisyanglu/images/raw/master/20210627234256.png)

```mysql
-- substring_index()嵌套，获取最右边的值
select substring_index(substring_index('a,b,c,d', ',', topic.help_topic_id+1), ',', -1) as new_col, topic.help_topic_id from mysql.help_topic as topic 
where topic.help_topic_id<=length('a,b,c,d')-length(replace('a,b,c,d', ',', ''))
```

![image-20200521223806990](https://gitee.com/louisyanglu/images/raw/master/20210627234301.png)