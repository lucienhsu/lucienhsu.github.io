---
layout: post
title: FindBugs分析记录
subtitle: FindBugs解决方案
date: 2018-9-13
author: lucienhsu
header-img: img/post-bg-2018-9-11-FindBugs.jpg
header-mask: 0.5
catalog: true
tags:
  - FindBugs
  - java
---

# 前言
FindBugs解决方案。

# 参考
1. [官网手册](http://findbugs.sourceforge.net/bugDescriptions.html)
2. [用FindBugs分析代码漏洞](https://www.cnblogs.com/hyddd/tag/hyddd%E7%9A%84FindBugs%E5%88%86%E6%9E%90%E8%AE%B0%E5%BD%95/default.html?page=1)

# 正文    
 
## EI: May expose internal representation by returning reference to mutable object
- 官方描述  
> Returning a reference to a mutable object value stored in one of the object's fields exposes the internal representation of the object.  If instances are accessed by untrusted code, and unchecked changes to the mutable object would compromise security or other important properties, you will need to do something different. Returning a new copy of the object is better approach in many situations.
- [解决方案](https://stackoverflow.com/questions/8951107/malicious-code-vulnerability-may-expose-internal-representation-by-returning-r)

## EI2: May expose internal representation by incorporating reference to mutable object    
- 官方描述  
> This code stores a reference to an externally mutable object into the internal representation of the object.  If instances are accessed by untrusted code, and unchecked changes to the mutable object would compromise security or other important properties, you will need to do something different. Storing a copy of the object is better approach in many situations.
- [解决方案](https://www.cnblogs.com/hyddd/articles/1391118.html)
> ![mark](http://pa99q7scc.bkt.clouddn.com/blog/180913/9HajKbAdgK.png?imageslim)

## Field is a mutable collection    
- 官方描述  
> A mutable collection instance is assigned to a final static field, thus can be changed by malicious code or by accident from another package. Consider wrapping this field into Collections.unmodifiableSet/List/Map/etc. to avoid this vulnerability.
- [解决方案](https://blog.csdn.net/ai_xiangjuan/article/details/78248694)       
```java
    import java.util.Collections;
    import java.util.HashMap;
    import java.util.Map;
    
    ...
    
    public static final Map<String, Object> ENGIN_DATATYPE;
    static {
        Map<String, Object> aMap = new HashMap();
        aMap.put("Number", "NUMBER,FLOAT,DOUBLE,DECIMAL,BINARY_FLOAT,BINARY_DOUBLE");
        aMap.put("String", "STRING,VARCHAR,CHAR,NCHAR,VARCHAR2,NVARCHAR2");
        aMap.put("Date", "DATE");
        aMap.put("Boolean", "BOOLEAN");
        aMap.put("Integer", "INTEGER,TINYINT,SMALLINT,MEDIUMINT,INT");
        aMap.put("BigNumber", "BIGNUMBER,BIGINT");
        aMap.put("Binary", "BINARY");
        aMap.put("Timestamp", "TIMESTAMP,DATETIME");
        aMap.put("Internet Address", "INTERNET ADDRESS");
        ENGIN_DATATYPE = Collections.unmodifiableMap(aMap); // 创建不可变集合
    }
```