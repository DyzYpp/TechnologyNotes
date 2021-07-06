#### 1. 索引的相关命令

show index from table_name;

创建普通索

alter table table_name add index index_name (column_list) ;

创建唯一索引

alter table table_name add unique index_name (column_list) ;

创建主键索引

alter table table_name add primary key index_name (column_list) ;

创建普通索引

create index index_name on table_name (column_list) ;

创建唯一索引

create unique index index_name on table_name (column_list) ;

删除索引

drop index index_name on table_name ; 

alter table table_name drop index index_name ;

 alter table table_name drop primary key ;

#### 2. 数据库表哪些情况需要建立索引

- 主键自动建立唯一索引

- 频繁作为查询条件的字段应创建索引

- 查询中与其它表关联的字段，外键关系建立索引

- 频繁更新的字段不适合建立索引

- where条件里用不到的字段不建立索引

- 单键/组合索引的选择问题，(高并发下倾向创建组合索引)

- 查询中排序的字段，排序字段若通过索引去访问将大大提高排序效率

- 查询中统计或分组的字段

  ------

  

#### 3. 数据库表哪些情况不需要建立索引

- 表数据少

- 频繁增删改的表

- 数据重复且分布平均的表字段

  ------

  

#### 4. Explain

##### 4.1 id  表的读取顺序

- id相同 从上到下顺序执行
- 如果是子查询，id不同，按id从大到小的顺序执行
- id相同又不同，id值越大，优先级越高，越先执行，id相同的，可以认为是一组，从上往下顺序执行。

##### 4.2 select_type 数据读取操作的操作类型

- simple       简单的select查询，查询中不包含子查询或union
- primary     查询中包含任何复杂的子部分，最外层查询被标记为
- subquery   在select或where列表中包含子查询
- derived       在from列表中包含的子查询被标记为deriver(衍生)，mysql会递归执行这些子查询把结果放在临时表里。
- union          若第二个select出现在union之后，则被标记为union;若union包含在from子句的子查询中，外层select被标记为: derived
- union result    从union表获取结果的select

##### 4.3 table 表

显示这一行数据是关于哪张表的

##### 4.4 type 性能级别

1. system	表只有一行记录(相当于系统表)，是const类型的特例，平时不会出现，可以忽略不计

2. const       表示通过索引，一次就找到了，const用于比较parmary_key和unique索引，因为只匹配一行数据，所以很快，如将主键置于where列表中，mysql就能将该查询转换为一个常量。

3. eq_ref      唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配，常见于主键或唯一索引扫描

4. ref            非唯一性索引扫描，返回匹配某个单独值的所有行

5. range       检索给定范围的行

6. index        全索引扫描

7. all              全表扫描，性能最差。

   **一般来说，sql查询效率至少到range级别。**

##### 4.5 possible_keys 哪些索引可以使用

查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用

##### 4.6 key  哪些索引实际被使用

实际使用的索引，如果为null，则没有使用到索引

查询中若使用了覆盖索引，则该索引只出现在key中

##### 4.7 key_length

表示索引使用的字节数，可通过该列计算查询中使用的索引的长度，在不损失精度的情况下，长度越短越好，key_length显示的值是索引字段的最大可能长度，并并非实际长度，即key_length是根据表定义而得，不是通过表内检索出的。

##### 4.8 ref 表之间的引用

表示索引的具体信息，哪个库的哪张表的哪个字段。

##### 4.9 rows 每张表都多少行被优化器查询

##### 4.10 Extra 包含不适合在其他列显示，但十分重要的信息

- Useing filesort  

  说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取，mysql无法利用表内的索引完成的排序操作成为 “文件排序”

- Useing temporary 

  使用了临时表保存中间结果，Mysql在对查询结果排序时使用临时表，常见于排序order by 和分组查询 group by 

- Useing index

  表示相应的select操作中使用了覆盖索引 (Covering index)，避免访问了表的数据行，效率较高。

  如果同时出现Useing where ，表明索引被用来执行索引键值的查找

  如果没有出现Useing where ， 表明索引用来读取数据而非执行查找动作。

#### 5. 覆盖索引 Covering index

理解方式一：select的数据列只用从索引中就能获取，不必读取数据行，Mysql可以利用索引返回表中的字段，而不必根据索引再次读取数据文件，换句话说**查询列要被所建的索引覆盖。**

理解方式二：索引是高效找到行的一个方法，但是一般数据库也能使用索引找到一个列的数据，因此它不必读取整个行，毕竟索引的叶子节点存储了他们索引的数据，当能通过读取索引就可以得到想要的数据，那就不需要读取行了，一个索引包含了或者说覆盖了满足查询结果的数据就叫覆盖索引。

**注意:** 	如果要使用覆盖索引，一定要注意select列表中只取需要的列，切记不可以select * ，因为如果将所有字段一起做索引会导致索引文件过大，查询性能下降

#### 6. JOIN 语句的优化

多表联查，尽可能减少 JOIN 语句中的 NestedLoop 的循环总次数： "**永远用小结果集驱动大的结果集**"

优先优化 NestedLoop 的内层循环

保证 JOIN 语句中的被驱动表上的 JOIN 条件字段已经被索引

当无法保证被驱动表的 JOIN 条件字段被索引且内存资源充足的前提下，不要太吝惜 JOINBUFFER 的设置

#### 7. 如何避免索引失效

1. 尽量全值匹配，避免模糊查询
2. 最左原则
3. 不在索引列上做任何操作(计算，函数，自动或手动的类型转换)，会导致索引失效，进而全表扫描。
4. 存储引擎不能使用索引中范围条件右边的列
5. 尽量使用覆盖索引，只访问索引列的查询， 减少select * 
6. 使用 != <> 的时候无法使用索引会导致全表扫描
7. is null, is not null 也无法使用索引列
8. like以通配符开头 % 会导致索引失效，变成全表扫描
9. 字符串不加单引号索引失效
10. 少用or，用or连接会导致索引失效

#### 7. Order By 的两种排序

Mysql支持两种方式的排序，FileSort 和 Index 其中，Index效率高，是Mysql扫描索引本身完成的排序，FileSort方式效率低，

Order By 满足两种情况，排序方式会使用 Index : 

- Order By 语句使用索引最左前列
- 使用 Where 子句与 Order By子句条件列组合满足最左前列

FileSort 又有两种排序方式：

- 单路排序 从磁盘读取查询需要的所有列，按照order by 列在Buffer对他们进行排序，然后扫描排序后的列表进行输出。它的效率更快一些，避免了第二次数据读取，并且把随机IO变成了顺序IO，但是 它会使用更多的空间，因为它把每一行都保存在内存中了。

  单路排序存在的问题：

  在mysql的 ini 配置文件中，sort_buffer数据所代表的容量。单路排序要比多路排序多占用很多空间，所以有可能取出的数据总大小超过了sort_buffer的数值，导致每次只能取sort_buffer容量大小的数据，进行排序，排序完在去sort_buffer容量大小的数据，进行多次IO。效率反而不如双路排序

- 双路排序   Mysql4.1之前使用的是双路排序，扫描磁盘两次，最终得到数据。读取行指针和OrderBy的列，对他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取对应的数据输出

##### 提高Order By排序速度

1. Order By 时，select * 是大忌，只查询需要的字段，这点非常重要，为什么呢

   1.1  当查询的字段大小综合小于 max_length_for_sort_data ， 而且排序字段不是 text|blob 类型时，会用改进后的算法--单路排序，否则用老算法---多路排序。

   1.2  两种算法的数据都有可能超出sort_buffer的容量，超出之后，会创建tmp文件进行合并排序，导致多次IO，但是用单路排序风险更大一些，所以要提高sort_buffer_size;

2. 尝试提高 sort_buffer_size

   不管用那种算法，提高这个参数都会提高效率，当然，要根据系统的能力去提高，因为这个参数是针对每个进程的。

3. 尝试提高 max_length_for_sort_data

   提高这个参数，会增加用改进算法的概率，但是如果设的太高，数据总容量超出sort_buffer_size的概率就会增大，明显症状是高的磁盘IO活动和低的处理器使用率

   ![](../TechnologyNotes\img\imgFile\orderby索引使用.png)

