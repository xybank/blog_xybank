---
title: Mybatis
date: 2018-05-08 12:00:00
tags: [Mybatis]
categories: 杨新才
---

settings是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。

<!--more-->

### 1. settings标签讲解

  该配置影响的所有映射器中配置的缓存的全局开关。默认值true  如：

``` setting:
<setting name="cacheEnabled" value="true"/>
```

  延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置fetchType属性来覆盖该项的开关状态。默认值false  如：

``` setting:
<setting name="lazyLoadingEnabled" value="true"/>
```

  是否允许单一语句返回多结果集（需要兼容驱动）。 默认值true  如：

``` setting:
<setting name="multipleResultSetsEnabled" value="true"/>
```

  使用列标签代替列名。不同的驱动在这方面会有不同的表现， 具体可参考相关驱动文档或通过测试这两种不同的模式来观察所用驱动的结果。默认值true  如：

``` setting:
<setting name="useColumnLabel" value="true"/>
```

  允许 JDBC 支持自动生成主键，需要驱动兼容。 如果设置为 true 则这个设置强制使用自动生成主键，尽管一些驱动不能兼容但仍可正常工作（比如 Derby）。 默认值false  如：

``` setting:
<setting name="useGeneratedKeys" value="false"/>
```

  指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示取消自动映射；PARTIAL 只会自动映射没有定义嵌套结果集映射的结果集。 FULL 会自动映射任意复杂的结果集（无论是否嵌套）。  如：

``` setting:
<setting name="autoMappingBehavior" value="PARTIAL"/>
```

  配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（prepared statements）； BATCH 执行器将重用语句并执行批量更新。默认SIMPLE  如：

``` setting:
<setting name="defaultExecutorType" value="SIMPLE"/>
```
