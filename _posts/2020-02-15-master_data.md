---
layout: post
title: 漫谈主数据
date: 2020-02-15
categories: blog
tags: [数据治理]
description: 主数据
---


## 主数据的概念
    
 主数据(Master Data) 是指具有高度业务价值的,可以在企业内跨越各个业务部门被重复使用的数据,是单一、准确、权威的数据来源。
 

## 主数据的特性与识别

  根据主数据的定义,可以看出主数据有以下几个特征:
  
  - 高价值性:主数据具有高度的业务价值,描述了企业最核心的数据,是企业最具有价值的核心数据资产.
  - 
  - 特征一致性:主数据可以跨越多个系统,主数据的特征常被用于业务流程判断与数据分析的维度,因此需要保证主数据的特征在不同应用之间的一致性.
  

  - 识别唯一性:在一个企业范围内,同一个主数据实体应有唯一的数据编码,可根据唯一数据编码确定唯一的数据.
  

  - 长期有效性:主数据通常贯穿该业务对象的整个生命周期甚至更长,这就要求主数据在系统中保持其有效性.
  
  - 业务稳定性:主数据用来描述业务对象的关键信息,在业务系统中其识别信息与关键特征会被业务过程中产生的数据继承,引用和复制.但无论业务过程如何复杂和持久,其识别信息与关键特征都不会被业务改变.
  
  业界多采用多特性综合判断的模式进行主数据识别，在严格模式下任一属性实行一票否决，在非严格模式下，可使用权重评分模式

- 严格模式：

  数据项名称 | 特征一致性 | 识别唯一性 | 长期有效性 | 交易稳定性 | 高价值性 | 判定结果
  --- | --- |  --- |  --- |  --- |  --- |  --- |  ---
  银行卡号 | 是 | 是| 是 | 是| 是 | 是
  身份证号| 是 | 是| 是 | 是| 是 | 是
  卡内余额 | 是  | 是 | 否 |否 | 是 | 否
  
  
- 非严格模式：

  数据项名称 | 特征一致性（10%） | 识别唯一性 （30%）| 长期有效性（20%） | 交易稳定性（10%） | 高价值性 （20%）| 分数 | 判定结果
  --- | --- |  --- |  --- |  --- |  --- |  --- |  --- |  ---
  银行卡号 | 是 | 是| 是 | 是| 是 | 100 | 是
  身份证号| 是 | 是| 是 | 是| 是 | 100  |是
  卡内余额 | 是  | 是 | 否 |否 | 是| 70  | 是
  

## 主数据的分类

根据不同的维度，存在多种分类方法：

按照层次分类：

- 引用数据 : 用于将其他数据进行分类或目录整编的数据，对定义的数据域值进行控制，包括对标准数据、代码值的控制，典型例子为：国家与地区代码、电话区号
- 企业结构数据：描述企业数据之间的关系、反映了现实世界中的实体间的关系或流程。例如：企业组织、产品线
- 业务结构数据：业务结构数据描述了业务了业务的直接参与者，产品数据和客户数据都是典型的业务结构数据。

按照数据来源分类：

原始主数据：即核心业务实体自带的属性，如客户的姓名，住址，证件号，联系方式，等。
衍生主数据：即通过计算客户的行为数据，得到的一些衍生的数据，这些数据也是不易改变的，如偏好类数据



按照数据范围分类

根据企业所处行业不同,主数据内容也不同,但绝大部分包含以下类型的主数据:

1. 基础数据
2. 组织与人员
3. 财务
4. 项目
5. 物理
6. 供应商与客户
7. 知识
8. 办公类

