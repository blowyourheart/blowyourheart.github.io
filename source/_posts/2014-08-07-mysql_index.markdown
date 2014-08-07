---
layout: post
title: "mysql 索引"
date: 2014-08-07 18:42:06 +0800
comments: true
categories: mysql 架构
---

## 索引的作用
如果没有索引，当在一个表上进行select操作时会扫描全表，随着网站运营的时间越来越久，表数据越来越大，网站访问性能也会显得越来越慢。

如果访问最频繁的操作在表上建立了索引，就不需要扫描表来找到数据，能极大的提高网站速度。

## 索引域的选择
 1. 越小的数据类型越好。
 1. 简单的数据类型更好，例如int，enum。
 1. 尽量避免使用NULL，可以使用0，""等占位符，NULL会使得索引要处理异常的情况。
 1. 最好使用整形，一些经常访问的字段也最好能转换成整形处理，比如日期时间，可以使用整形表示的秒数来表示。

## 单列索引与多列索引
 单列索引是只在一个域上建立索引，比如主键，外键约束，以及对单个字段的唯一性约束，都是单列索引。

 多列索引是对多个字段建立索引，包括多个字段的唯一性约束等。

 mysql可以在一张表上建立多个索引，但是一次查询只会使用一个索引来搜索。

### 最左前缀原则
  对于一个多列索引（A，B，C），如果where子句中有【A，B，C】，【A，B】，【A】这三个中的任何一个约束，则可以使用该索引，而以B或者C开头的查询则无法使用该索引。

 因为索引一般都以B-Tree存储，树形结构都是从一个节点开始比较，然后过度到下一个节点，所以先天的会有最左前缀这个特性。**记住这点非常重要**，要结合自身的业务逻辑设计出最简洁最高效的索引。

 举例： 如果你有一张表存储一个用户买过的商品，表结构为 order(id, productId, userId, createdTime)。假定你有一下常用三种业务逻辑：

 1. 查询某件商品是否被某个用户购买。
 2. 查询一件商品被哪些用户购买了。
 3. 查询某个用户购买了哪些商品。
 
 
 在productId和userId上建立索引如下。

    alter table `order` add index productId_userId(productId, userId)
    alter table `order` add index userId(userId)

 这样需求1，2可以通过第一个索引查询，根据最左前缀原则，需求3无法使用多列索引 productId_userId, 所以对userId单独建立了一个索引。


## 索引引擎
 1. B-Tree， 默认值。
 1. Hash索引  数据会存储在内存中，而且engine必须是 MEMORY , 慎用。
 1. R-Tree 空间索引，用于空间数据库。
 1. Full-Text 全文索引， R-Tree的增强版本。

## 索引的创建，删除，查看
### 创建索引
	ALTER TABLE table_name ADD INDEX index_name (column_list)
	ALTER TABLE table_name ADD UNIQUE (column_list)
	ALTER TABLE table_name ADD PRIMARY KEY (column_list)

### 索引的删除 
	DROP INDEX index_name ON talbe_name
	ALTER TABLE table_name DROP INDEX index_name
	ALTER TABLE table_name DROP PRIMARY KEY

### 索引查看
	show index from tblname;
