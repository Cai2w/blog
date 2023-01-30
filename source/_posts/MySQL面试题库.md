---
title: MySQL面试题库
date: 2023-01-29 14:17:15
tags:
categories:
- 面试
description:
sticky:
cover:
---

## 正餐

### MySQL 索引使用有哪些注意事项呢

{% hideToggle  MySQL 索引使用有哪些注意事项 %}可以从三个维度回答这个问题：索引哪些情况会失效，索引不适合哪些场景，索引规则

索引哪些情况会失效

- 查询条件包含or，可能导致索引失效
- 如何字段类型是字符串，where时一定用引号括起来，否则索引失效
- like通配符可能导致索引失效。
- 联合索引，查询时的条件列不是联合索引中的第一个列，索引失效。
- 在索引列上使用mysql的内置函数，索引失效。
- 对索引列运算（如，+、-、*、/），索引失效。
- 索引字段上使用（！= 或者 < >，not in）时，可能会导致索引失效。
- 索引字段上使用is null， is not null，可能导致索引失效。
- 左连接查询或者右连接查询查询关联的字段编码格式不一样，可能导致索引失效。
- mysql估计使用全表扫描要比使用索引快,则不使用索引。

[后端程序员必备：索引失效的十大杂症](https://juejin.im/post/5de99dd2518825125e1ba49d)

索引不适合哪些场景

- 数据量少的不适合加索引
- 更新比较频繁的也不适合加索引
- 区分度低的字段不适合加索引（如性别）

索引的一些潜规则

- 覆盖索引
- 回表
- 索引数据结构（B+树）
- 最左前缀原则
- 索引下推

{% endhideToggle %}

### MySQL 遇到过死锁问题吗，你是如何解决的

{% hideToggle MySQL 遇到过死锁问题吗，你是如何解决的 %}我排查死锁的一般步骤是酱紫的：

- 查看死锁日志show engine innodb status;
- 找出死锁Sql
- 分析sql加锁情况
- 模拟死锁案发
- 分析死锁日志
- 分析死锁结果

可以看我这两篇文章哈：

- [手把手教你分析Mysql死锁问题](https://juejin.im/post/5e8b269f518825739379e82c)
- [Mysql死锁如何排查：insert on duplicate死锁一次排查分析过程](https://juejin.im/post/5d483e66518825052734b15a#heading-4)

{% endhideToggle %}





{% hideToggle display,bg,color %}
content
{% endhideToggle %}

## 结语

> 我是菜菜🥬，一个人菜瘾大的互联网从业者🎉
>
> 用功不求太猛，但求有恒🎈