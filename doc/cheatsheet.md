[toc]

# 常用命令

## 基本用法

```bash
echo "select title from sakila.film" | ./soar -log-output=soar.log
```

## 指定配置文件

```bash
vi soar.yaml
# yaml format config file
online-dsn:
    addr:     127.0.0.1:3306
    schema:   sakila
    user:     root
    password: "1t'sB1g3rt"
    disable:  false

test-dsn:
    addr:     127.0.0.1:3306
    schema:   sakila
    user:     root
    password: "1t'sB1g3rt"
    disable:  false
```

```bash
echo "select title from sakila.film" | ./soar -test-dsn="root:1t'sB1g3rt@127.0.0.1:3306/sakila" -allow-online-as-test -log-output=soar.log
```

## 打印所有的启发式规则

```bash
$ soar -list-heuristic-rules
```

## 忽略某些规则

```bash
$ soar -ignore-rules "ALI.001,IDX.*"
```

## 打印支持的报告格式

```bash
$ soar -list-report-types
```

## 以指定格式输出报告

```bash
$ soar -report-type json
```

## 语法检查工具

```bash
$ echo "select * from tb" | soar -only-syntax-check
$ echo $?
0

$ echo "select * fromtb" | soar -only-syntax-check
At SQL 0 : syntax error at position 16 near 'fromtb'
$ echo $?
1

```

## 慢日志进行分析示例

```bash
$ pt-query-digest slow.log > slow.log.digest
# parse pt-query-digest's output which example script
$ python2.7 doc/example/digest_pt.py slow.log.digest > slow.md
```


## SQL指纹

```bash
$ echo "select * from film where col='abc'" | soar -report-type=fingerprint
```

输出

```sql
select * from film where col=?
```

## 将UPDATE/DELETE/INSERT语法转为SELECT

```bash
$ echo "update film set title = 'abc'" | soar -rewrite-rules dml2select,delimiter  -report-type rewrite
```

输出

```sql
select * from film;
```


## 合并多条ALTER语句

```bash
$ echo "alter table tb add column a int; alter table tb add column b int;" | soar -report-type rewrite -rewrite-rules mergealter
```

输出

```sql
ALTER TABLE `tb` add column a int, add column b int ;
```

## SQL美化

```bash
$ echo "select * from tbl where col = 'val'" | ./soar -report-type=pretty
```

输出

```sql
SELECT
  *
FROM
  tbl
WHERE
  col  = 'val';
```

## EXPLAIN信息分析报告

```bash
$ soar -report-type explain-digest << EOF
+----+-------------+-------+------+---------------+------+---------+------+------+-------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra |
+----+-------------+-------+------+---------------+------+---------+------+------+-------+
|  1 | SIMPLE      | film  | ALL  | NULL          | NULL | NULL    | NULL | 1131 |       |
+----+-------------+-------+------+---------------+------+---------+------+------+-------+
EOF
```

```text
##  Explain信息

| id | select\_type | table | partitions | type | possible_keys | key | key\_len | ref | rows | filtered | scalability | Extra |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1  | SIMPLE | *film* | NULL | ALL | NULL | NULL | NULL | NULL | 0 | 0.00% | ☠️ **O(n)** |  |


### Explain信息解读

#### SelectType信息解读

* **SIMPLE**: 简单SELECT(不使用UNION或子查询等).

#### Type信息解读

* ☠️ **ALL**: 最坏的情况, 从头到尾全表扫描.
```

## markdown转HTML

通过指定-report-css, -report-javascript, -markdown-extensions, -markdown-html-flags这些参数，你还可以控制HTML的显示格式。

```bash
$ cat test.md | soar -report-type md2html > test.html
```

