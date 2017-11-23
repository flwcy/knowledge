---
title: PreparedStatement IN语句
date: 2016-07-21 20:53:55
categories:
- 技术札记
tags: [jdbc]
---

为了防止SQL注入攻击，PreparedStatement不允许一个占位符（？）有多个值，在执行有**IN**子句查询的时候这个问题变得棘手起来。本文旨在提供对应的解决方案。

<!--more-->

## 问题描述

由于存在SQL注入攻击的安全问题，PreparedStatement不允许一个占位符（？）有多个值，一个`?`占位符仅代表了一个值，而不是值的列表，那么，当我们使用PreparedStatement来执行包含IN子句的SQL语句的最佳实现是怎样的呢？

我们考虑下如下的SQL语句：

```sql

SELECT email FROM db_user WHERE user_name IN (?)

```

## 解决方案

使用preparedStatement.setString(1,"‘flwcy’,'rooike','test'")基本上是一个非有效的解决方案。为了解决这个问题，我们有很多可用的方案，具体解决方案如下：

准备好`SELECT email FROM db_user WHERE user_name = ?`语句，为每个值都执行这条语句，最后合并结果集，我们只需为这些执行语句准备一个PreparedStatement对象即可，该方式执行缓慢而且费力。

```java

            String[] args = {"jjr","test","js123"};

            List<String> results = new ArrayList<String>();

            preparedStatement = connection.prepareStatement(sql);

            for(String str : args){

                preparedStatement.setString(1,str);

                resultSet = preparedStatement.executeQuery();

                while(resultSet.next()){

                    results.add(resultSet.getString("email"));

                }

            }

```

准备好`SELEC email FROM db_user where user_name = ?`语句，并执行该语句，需要为该语句准备与`IN`语句相同长度的参数集合。

```java

            // Build the SQL statement

            String[] args = {"jjr","test","js123"};

            StringBuilder stringBuilder = new StringBuilder();

            for(String str : args){

                stringBuilder.append("?,");

            }

            String sql = "SELECT email FROM db_user WHERE user_name IN(" + stringBuilder.deleteCharAt(stringBuilder.length() - 1) +")";

            List<String> results = new ArrayList<String>();

            preparedStatement = connection.prepareStatement(sql);

            // Injection parameters

            int index = 1;

            for(String str : args){

                preparedStatement.setString(index++,str);

            }

            resultSet = preparedStatement.executeQuery();

            while(resultSet.next()){

                results.add(resultSet.getString("email"));

            }

```

## More Read

[PreparedStatement IN clause alternatives?](http://stackoverflow.com/a/189399)