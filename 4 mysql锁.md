MySQL 锁

[参考资料](https://blog.csdn.net/tr1912/article/details/81668423)

* MyISAM 只有表锁、InnoDB即支持行锁也支持表锁（主要是行锁使用），因为MyISAM只支持表锁，所以MySQL进程要么是在锁等待要么是在锁表中这样就不会出现死锁的情况。一般情况MyISAM锁都是隐式的，不需要开发者特别声明。但是因为MyISAM是表锁，所以在写入和更新时对并发的读性能有影响。所以尽量不要用MyISAM进行修改删除修改操作（插入不影响，默认情况下在读锁的时候允许插入操作）

* 悲观锁&乐观锁

  `乐观锁`：是开发者手动维护的一种形式，形式多种多样，其本质是在行内维护一个version字段，在事务提交前检查version是否有变化，如果有变化则rollback。

  `悲观锁`：是开发者通过加锁（for update）方式，确保数据一致性，具体参死锁相关知识。

* 死锁 Deadlock

  产生死锁一般是两个或者多个进程对若干资源加锁形成相互等待导致死锁。但具体过程情况比较复杂，如下过程依次执行：

  session-1:

  ~~~ mysql
  start transaction;
  select * from book where id = 2 for update;
  ~~~

  session-2:

  ~~~ mysql
  start transaction;
  select * from book where id = 2 for update;
  ~~~

  session-1:

  ~~~ mysql
  select * from book where id = 2 for update;   #此时执行该条语句开始阻塞，等待session-2释放锁
  ~~~

  session-2:

  ~~~ mysql
  select * from book where id = 1 for update;   
  #会出现死锁报错：ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
  ~~~

  解决办法：

  ​	其实session-1与session-2语义是一致的，是不同顺序加锁导致的问题。可以简单的通过如下方式解决：

  ~~~ mysql
  #两个session一样的写法
  start transaction;
  select * from book where id in (2,1) for update;  #* in的方式mysql会自动将括号内的数值按从小到大的顺序排序 这样保持加锁的顺序一致性
  ~~~

  

* 事务的隔离级别

  | 隔离界别\并发副作用              | 脏读 | 不可重复读 | 幻读 |
| -------------------------------- | ---- | ---------- | ---- |
  | 未提交读 read uncomited          | 是   | 是         | 是   |
| 已提交读 read commited           | 否   | 是         | 是   |
  | 可重复读 repeatable read（默认） | 否   | 否         | 是   |
  | 可序列化 serializable            | 否   | 否         | 否   |
  
  

* 锁相关注意点

  1. 行锁退化成表锁

     行锁是通过索引建立的，如果建立行锁的sql的条件语句没有用到索引（即使改行记录中有索引字段）那么也会把整个表加锁。但是如果where条件中用到了索引（无论是什么索引）便可实现行锁，那么其他session中执行该行加锁for update时都要阻塞。
     
     `特殊情况`：如果在sql中的条件语句因为隐式转换而无法使用索引则同样会表锁如下：
     
     ~~~ mysql
     select * from book where chapter = 1 for update;  #chapter的类型是varchar但是在该语句中会进行隐式转换无法用的chapter的字符串类型的索引
     ~~~
     
     尽量避免`gap lock(间隙锁)`  、`next-key lock` 详见如下范围查找
     
  2. 范围查找
  
     行锁的3种情形：
  
     `record lock` 对索引项加锁
  
     `gap lock` 对索引项前或者后加锁
  
     `next-key lock` 上面两种结合
  
     ~~~ mysql
     #对于范围查找会形成next-key lock，并且范围查找可以在其他的session中对同一范围重复加锁，容易导致死锁出现下面死锁例子
     select * from book where id > 100 for update;
     select * from book where id > 2 and id < 100 for update;
     #无论能否查到数据对会对未出现的数据加锁
     select * from book where id = 10000000000 for update;
     ~~~
  
     死锁示例(假设id目前只有5)
  
     session-1:
  
     ~~~ mysql
     select * from book where id > 5 for update;
     ~~~
  
     session-2:
  
     ~~~ mysql
     select * from book where id > 5 for update;
     ~~~
  
     session-1:
  
     ~~~ mysql
     insert into book (title,price) values ("java",22); #wait... 因为session-2占用锁，索引等待
     ~~~
  
     session-2:
  
     ~~~ mysql
     insert into book (title,price) values ("java",22); # Deadlock 同时释放锁，session-1则会写入成功
     ~~~
  
     
