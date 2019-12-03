```mysql

#查询联合  作用是查询索引和每个查询语句的类型
EXPLAIN SELECT * FROM employees WHERE emp_no = '10001'
UNION
select * from employees where emp_no = '10002'


#如果union是作为子查询为存在的话那么查询的结果类型就会变成 2: DEPENDENT SUBQUERY / 3: DEPENDENT UNION
EXPLAIN select * from employees where emp_no in 
(
	select emp_no from employees where emp_no = '10001'
  union 
	select emp_no from employees where emp_no = '10002'
)

#如果主查询条件中查询的结果是别的表结果 那么类型就是SUBQUERY
explain select * from employees where emp_no = 
	(select emp_no from employees where emp_no='10001')
	
	
#如果子查询中from的是另一个表的结果的话那么类型就会变成 DERIVED 并且第一个查询的表还会变成第二个查询出来的结果
explain select * from (select emp_no from employees where emp_no = '10001') a

#--------------------------------------------------------------------------------------------------------
#type 查询的速度

#eq_ref  速度比较好  
# e表为all b表为eq_ref 
# 执行原理是先根据from表的顺序 , 先查询全部 , 当查询第二个表的时候 直接去树中查找到与第一个表设定条件相等的那些数据就可以了
EXPLAIN select * from employees b,employees e where e.emp_no = b.emp_no

#ref
#这种情况通常出现在两个表不是一个表但是又因为主键索引相关联的表上面 执行原理同上
EXPLAIN select * from employees e,titles t where e.emp_no = t.emp_no
 

#ref_or_null  表示mysql中含有null的字段
CREATE TABLE `t_order` ( `id` bigint(20) NOT NULL, `user_id` bigint(20) DEFAULT NULL, PRIMARY KEY (`id`), KEY `user_id` (`user_id`) USING BTREE ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

#这表示数据库的表设计中 , 被当做索引的字段是允许空值的, 一般在开发中不建议这样做 , 因为null值并不会被保存到数据树中, 而是单独保存在一片区域 , 如果数据很多那么将会耗费额外的性能来进行null值的查找
explain select * from t_order where user_id=100 or user_id is null

#index_merge  通常出现在一个表含有两个索引并查询这两个索引的时候 , 这个时候mysql就会将两个索引合并并且做出优化
explain select * from dept_manager where emp_no = 110022 or dept_no = 'd001'
  
#range  用于查询范围的索引 , 这种索引很常见
explain select * from employees where emp_no > 10001

#index 这个连接类型会扫描索引树 , 仅仅比all快一点
explain select count(*) from employees 

#all  扫描整个表 , 最慢的连接类型, 尽量避免
EXPLAIN select * from employees


#--------------------------------------------------------------------------------------------------------
#extra  
#Not exists 
explain select count(1) from employees a left JOIN dept_emp b on a.emp_no = b.emp_no where b.emp_no is null;
explain select count(1) from employees a left JOIN dept_emp b on a.emp_no = b.emp_no ;show PROFILES

#Range checked for each record (index map: 0x1)   mysql并没有找到合适的索引来为我们进行优化 , 因为索引根本不成立
EXPLAIN select * from employees e,employees b where e.emp_no >=b.emp_no and e.emp_no<=b.emp_no and e.first_name='Kazuhide'

#一般这种情况都需要优化 , 因为生成了一个临时表 , temporary 访问临时表是比较消耗性能的
explain select * from  employees a INNER JOIN dept_emp b on a.emp_no = b.emp_no group by b.emp_no;
explain select * from  employees a INNER JOIN dept_emp b on a.emp_no = b.emp_no group by a.emp_no;show PROFILES

#Using union(PRIMARY,dept_no); Using where  一般这种时候都是使用了联合索引查询导致的 , 这并不是一件坏事
explain select * from dept_manager where emp_no = 110022 or dept_no = 'd001'


#?????
explain select dept_no from dept_emp GROUP BY dept_no;



select @@profiling




#--------------------------------------------------------------------------------------------------------
#索引的策略以及优化
ALTER TABLE employees.titles DROP INDEX emp_no;


explain select * from employees.titles where emp_no = '10001' and title='Senior Engineer' and from_date = '1986-06-26';show PROFILES
#如果没有关联到主键那么就没有使用mysql的优化就是说没有根据索引树去查找
explain select * from employees.titles where title='Senior Engineer' and from_date = '1986-06-26';
#但是如果在where的条件中查找到了索引那么mysql就会自动将我们的顺序排序并使用索引进行查询
explain select * from employees.titles where title='Senior Engineer' and from_date = '1986-06-26' and emp_no = '10001';


#显示mysql决定要使用的键的长度
explain select * from employees.titles where emp_no = '10001';show PROFILES

#由于from_date不是索引所以mysql在使用查询的时候并没有奖索引一同加入进去   优化的办法可以给from_date增加一个辅助索引 , 或者使用in 来填坑
explain select 8 from employees.titles where emp_no = '10001' and from_date='1986-06-26'


#使用模糊查询   由于title是辅助索引并且通配符在后缀所以可以使用
explain select * from employees.titles where emp_no = '10001' and title LIKE 'Senior%'
#由于通配符在前缀 所以不能够使用
explain select * from employees.titles where emp_no = '10001' and title LIKE '%Senior%'

#范围列可以使用到索引 , 不过必须为列的左前缀但是范围后面的列无法应用到索引 , 并且索引同时应用于一个范围列如果多个范围列则不能应用
EXPLAIN select * from employees.titles where emp_no < '10001' and title = 'SeniorEngineer'

#如果使用between就不算是范围查询 作用在emp_no上的between
EXPLAIN select * from employees.titles where emp_no BETWEEN '10001' and '10010' and title ='Senior Engineer'


#用联合索引将数据的性能提升
select count(DISTINCT(concat(first_name,last_name)))/count(*) as selectivity from employees.employees


select (select count(*) from employees) as employees_count, (select count(*) from dept_emp) as dept_emp_count

#小结果集驱动大结果集
explain select * from dept_emp a INNER JOIN employees b on a.emp_no = b.emp_no 

#如果写的是left join就是左边驱动右边 , 是强制性的驱动右边
explain select * from dept_emp a LEFT JOIN employees b on a.emp_no = b.emp_no


select (select count(*) from employees) as employees_count, (select count(*) from dept_emp where to_date='9999-01-01') as dept_emp_count

#使用这个连接让mysql可以正确的用小结果集驱动大结果集
explain select * from dept_emp a STRAIGHT_JOIN  employees b on a.emp_no = b.emp_no where a.to_date='9999-01-01'
```

