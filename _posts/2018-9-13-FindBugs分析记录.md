---
layout: post
title: FindBugs分析记录
subtitle: FindBugs解决方案
date: 2018-9-13
author: lucienhsu
header-img: img/post-bg-2018-9-11-FindBugs.jpg
catalog: true
tags:
  - FindBugs
  - java
---

# 前言
FindBugs解决方案。

# 参考
1. [官网手册](http://findbugs.sourceforge.net/bugDescriptions.html)
1. [用FindBugs分析代码漏洞](https://www.cnblogs.com/hyddd/archive/2009/02/13/1390362.html)

# 正文    
 
### EI: May expose internal representation by returning reference to mutable object
- 官方描述  
> Returning a reference to a mutable object value stored in one of the object's fields exposes the internal representation of the object.  If instances are accessed by untrusted code, and unchecked changes to the mutable object would compromise security or other important properties, you will need to do something different. Returning a new copy of the object is better approach in many situations.
- [解决方案](https://stackoverflow.com/questions/8951107/malicious-code-vulnerability-may-expose-internal-representation-by-returning-r)

### EI2: May expose internal representation by incorporating reference to mutable object    
- 官方描述  
> This code stores a reference to an externally mutable object into the internal representation of the object.  If instances are accessed by untrusted code, and unchecked changes to the mutable object would compromise security or other important properties, you will need to do something different. Storing a copy of the object is better approach in many situations.
- [解决方案](https://www.cnblogs.com/hyddd/articles/1391118.html)
> ![mark](http://pa99q7scc.bkt.clouddn.com/blog/180913/9HajKbAdgK.png?imageslim)