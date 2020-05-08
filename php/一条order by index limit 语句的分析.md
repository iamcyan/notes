在开发过程中，遇到了一条 order by index limit 的语句，执行时间慢，下面记录一下分析过程和原因
## 问题 SQL
```mysql
select * from t1 where call_type = 2 and sys_type = 1 order by third_id desc limit 1;
```

说明：数据库为 MySQL，引擎为 Innodb，版本为 5.6，其中third_id为普通索引。

对于一个慢 SQL，想到的第一部是通过 explain 查看其执行计划，上面这条 SQL 的执行计划如下：

| **id** | **select_type** | **table**       | **type** | **possible_keys** | **key**      | **key_len** | **ref** | **rows** | **Extra**   |
| ------ | --------------- | --------------- | -------- | ----------------- | ------------ | ----------- | ------- | -------- | ----------- |
| 1      | SIMPLE          | call_record_log | index    |                   | idx_third_id | 4           |         | 1        | Using where |

可以看到，启用了索引 id_third_id，下面就需要搞明白为什么 SQL 在启用了索引的情况下，还是很慢，有下面3个猜测：

1. 因为在 SQL 中使用了 desc，猜测索引在被倒序排序的过程中耗时较长。
2. 在我们使用 limit 的情况下引发了慢查询
3. order by index 的原因，根据 explain 的解释信息可以，type 类型为 index，而 index 类型通常来说是比较慢的一种情况。

下面我们针对这两种情况进行验证。

## 猜测1：索引倒序排序的问题

1、通过查询资料得到，在 MySQL 中，使用 Innodb 引擎的时候，索引的排序时有序的，通过正序使用和倒序使用不应该影响 MySQL 的查询效率（）。

2、通过将 desc 改为 asc，在实际测试过程中发现，对返回数据影响最大的是第一条符合条件的记录出现的位置，即在找到第一条记录前，需要扫描多少条语句。

根据上面两点，可知：order by index desc/asc 并不是引发这条 SQL 产生慢查询的原因。



## 猜测2：limit  引发了慢查询

针对 limit 的使用情况，查询了 [MySQL 使用手册](https://dev.mysql.com/doc/refman/5.6/en/limit-optimization.html)，证实了猜测。摘录关键信息如下：

> If you select only a few rows with LIMIT, MySQL uses indexes in some cases when normally it would prefer to do a full table scan.
>
> 大意：如果只是查询几条数据，MySQL 在某些情况下会使用索引，通常情况下回权标扫描。

这一点和我们上面看到的 explain 是一致的，使用了索引。但是并没有告诉我们会什么在使用了索引的情况下会慢，继而引发了猜测，是不是 limit 也是导致这个问题的原因呢。将 limit 条件去掉看一下 explain 的信息：

| **id** | **select_type** | **table**       | **type** | **possible_keys** | **key** | **key_len** | **ref** | **rows** | **Extra**                   |
| ------ | --------------- | --------------- | -------- | ----------------- | ------- | ----------- | ------- | -------- | --------------------------- |
| 1      | SIMPLE          | call_record_log | ALL      |                   |         |             |         | 2133780  | Using where; Using filesort |

发现，此时执行的是全表扫描，并没有继续使用索引。可以得出的推断：

1. 没有 limit 的情况下，没有像希望的那样使用索引，说明 MySQL 认为使用索引代价更大。
2. 在有 limit 的情况下，会相当于强制使用了索引。

继续阅读手册中 order by 相关的东西，发现如下：

>If you combine LIMIT row_count with ORDER BY, MySQL stops sorting as soon as it has found the first row_count rows of the sorted result, rather than sorting the entire result. If ordering is done by using an index, this is very fast. If a filesort must be done, all rows that match the query without the LIMIT clause are selected, and most or all of them are sorted, before the first row_count are found. After the initial rows have been found, MySQL does not sort any remainder of the result set.
>
>大意：如果 order by 和 limit 组合使用，MySQL 在查询到对应的数据之后就会立即停止而不是排序全部的结果。如果 order by 使用了索引那么通常来说这个过程是很快的。如果 filesort 的操作必须被执行，在匹配到程序要查询的结果之前，这个 SQL 在没有 limit 的情况下查出来的绝大部分或者全部结果都会被排序。在最原始的记录被找到之后，MySQL 不再排序后面的任何结果集。

手册说在 order by 后面有索引的情况会 very fast，但是和我们的结果完全不同，没有在文档中找到解释慢查询的原因，先去看看 手册中关于 order by 的说明。

## 猜测3：order by index 引发了慢查询

查看对应的[文档](<https://dev.mysql.com/doc/refman/5.6/en/order-by-optimization.html>)

> In some cases, MySQL may use an index to satisfy an `ORDER BY` clause and avoid the extra sorting involved in performing a `filesort` operation.
>
> The index may also be used even if the `ORDER BY` does not match the index exactly, as long as all unused portions of the index and all extra `ORDER BY` columns are constants in the `WHERE` clause. If the index does not contain all columns accessed by the query, the index is used only if index access is cheaper than other access methods.
>
> 大意：
>
> 在某些情况下，MySQL可能会使用索引来满足ORDER BY子句，并避免执行filesort操作时涉及的额外排序。
>
> 即使ORDER BY与索引不完全匹配，也可以使用索引，只要索引的所有未使用部分和所有额外的ORDER BY列都是WHERE子句中的常量即可。 如果索引不包含查询访问的所有列，则仅在索引访问比其他访问方法更便宜时才使用索引。

通过上面的描述，我们看到了解决问题的希望，即并不是在 order by 后面指定了索引索引就会被启用，只有当使用索引更为合适的情况下才会被使用，这和我们上面的 SQL 在没有 limit 的情况下执行的是全表扫面是一致的，即此种情况下 MySQL 执行全表扫描都比按照索引执行的顺序更高效。继续看文档。

```mysql
SELECT * FROM t1ORDER BY key_part1, key_part2;
```

> However, the query uses `SELECT *`, which may select more columns than *key_part1* and *key_part2*. In that case, scanning an entire index and looking up table rows to find columns not in the index may be more expensive than scanning the table and sorting the results. If so, the optimizer probably will not use the index. If `SELECT *` selects only the index columns, the index will be used and sorting avoided.
>
> 大意：
> 但是，查询使用SELECT *，它可以选择比key_part1和key_part2更多的列。 在这种情况下，扫描整个索引并查找表行以查找不在索引中的列可能比扫描表并对结果进行排序更昂贵。 如果是这样，优化器可能不会使用索引。 如果SELECT *仅选择索引列，则将使用索引并避免排序。

我们发现如果 order by 的索引没有覆盖掉全部的查询字段的时候，优化器认为使用索引是一种更昂贵的操作，就不在使用索引。终于在文档中发现了能够解释我们上面的 SQL 是查询耗时较长的原因了。

后续的文档给出了哪几种情况下 order by 的索引会生效，哪几种情况下是无效的，在此就不一一列举了。

## 到底如何优化上面的 SQL 呢

我们私下讨论的存在三个方向：

1. 将 where 条件中涉及到的几个字段和 thrid 字段建立联合索引，同时select 字段要覆盖索引
2. 将 order by third 替换成 primary key，可以在一定程度上解决这个问题。
3. 根据具体的业务逻辑，重构 SQL。