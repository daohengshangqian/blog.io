---
layout: post
title:  PL/SQL中的包
date: 2008-04-26
categories: blog
tags: [PL/SQL]
description: 文章金句。
---
# 包

## 包的组成

包由两部分组成
分别为包的声明部分和包的实现部分

包的声明部分
```SQL
CREATE OR REPLACE PACKAGE  PKG_1 
IS

END ;
```
包的实现部分
```sql
CREATE OR REPLACE PACKAGE BODY PKG_1
IS

END ;

```

## 包的作用

- 使用包将程序分组归类
- 使用私有方法隐藏逻辑


```sql

CREATE OR REPLACE PACKAGE PKG_YUAN
IS
PI  CONSTANT  NUMBER :=3.14;

 FUNCTION GET_YUAN_ZHUI_TI_JI (IN_R NUMBER , IN_H NUMBER )
  RETURN NUMBER ;
  
END ;


```
包的实现部分

```sql

CREATE OR REPLACE PACKAGE BODY PKG_YUAN IS
  --获取圆面积
  FUNCTION GET_YUAN_MIAN_JI(IN_R NUMBER) RETURN NUMBER IS
  BEGIN
    RETURN IN_R * IN_R * PI;
  END;
  
  --获取圆柱体积
  FUNCTION GET_YUAN_ZHU_TI_JI(IN_R NUMBER, IN_H NUMBER) RETURN NUMBER IS
    V_MIAN_JI NUMBER;
  BEGIN
    V_MIAN_JI := GET_YUAN_MIAN_JI(IN_R);
  
    RETURN V_MIAN_JI * IN_H;
  END;
  
  --获取圆锥体积
  FUNCTION GET_YUAN_ZHUI_TI_JI(IN_R NUMBER, IN_H NUMBER) RETURN NUMBER IS
  BEGIN
    RETURN GET_YUAN_ZHU_TI_JI(IN_R, IN_H) / 3;
  END;

END;


```

在上面的例子中,我们将与圆相关的计算功能放入到一个包中.

我们还可以再将与正方形相关的计算功能放入另一个包中.

这样我们就通过包实现的程序的分类


## 包的私有方法
在包的实现部分中存在三个函数,而在包的声明部分只存在一个函数.

存放在包实现部分,但未在包声明部分出现的函数或过程我们称之为包的私有函数/过程

私有函数/过程只能被包内的其他函数过程调用.

并且被调用的私有过程/函数要写在调用者之前.

否则报错

包的声明部分

```sql
CREATE OR REPLACE PACKAGE PKG_YUAN_1
IS
PI  CONSTANT  NUMBER :=3.14;

 FUNCTION GET_YUAN_ZHUI_TI_JI (IN_R NUMBER , IN_H NUMBER )
  RETURN NUMBER ;
  
END ;


```


包的实现部分

```sql
CREATE OR REPLACE PACKAGE BODY PKG_YUAN_1 IS

  --获取圆锥体积
  FUNCTION GET_YUAN_ZHUI_TI_JI(IN_R NUMBER, IN_H NUMBER) RETURN NUMBER IS
  BEGIN
    RETURN GET_YUAN_ZHU_TI_JI(IN_R, IN_H) / 3;

  END;

    --获取圆面积
  FUNCTION GET_YUAN_MIAN_JI(IN_R NUMBER) RETURN NUMBER IS
  BEGIN
    RETURN IN_R * IN_R * PI;
  END;

  --获取圆柱体积
  FUNCTION GET_YUAN_ZHU_TI_JI(IN_R NUMBER, IN_H NUMBER) RETURN NUMBER IS
    V_MIAN_JI NUMBER;
  BEGIN
    V_MIAN_JI := GET_YUAN_MIAN_JI(IN_R);

    RETURN V_MIAN_JI * IN_H;
  END;

END;

```

编译错误

函数 GET_YUAN_ZHUI_TI_JI 引用了GET_YUAN_ZHU_TI_JI 

由于GET_YUAN_ZHUI_TI_JI在引用了GET_YUAN_ZHU_TI_JI之前,引发此报错.


```sql

PACKAGE BODY DAO.PKG_YUAN_1 编译错误

错误：PLS-00313: 'GET_YUAN_ZHU_TI_JI' not declared in this scope
行：6
文本：RETURN GET_YUAN_ZHU_TI_JI(IN_R, IN_H) / 3;

错误：PL/SQL: Statement ignored
行：6
文本：RETURN GET_YUAN_ZHU_TI_JI(IN_R, IN_H) / 3;

```
