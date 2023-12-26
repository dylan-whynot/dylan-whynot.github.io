---
title : "SpringAop内部调用问题"
description: "SpringAop内部调用问题"
date:        "2023-12-26"
author:     "Dylan" 
tags:        ["Java", "Spring"]
categories:  ["编程" ]
---
使用Spring AOP功能时发现一个问题,简单描述一下:Spring 切面功能时,目标类方法内部调用时 增强功能没有生效,但是单个调用目录类的任何方法增强都是成功的.带着疑问Google,查阅Spring 官方文档.发现原来是自己对AOP原理理解不够深.
<!--more-->
![](/post_images/188-trunklog-stock.jpg)

# 问题描述

为了清楚的描述问题,我通过一个简单的例子来实现,首先定义一个TestService类内部包含两个方法分别为eatFoods() ,gotoWork(),在eatFood() 方法内部调用 gotoWork().并定义前置增强 具体代码如下:
```java
//接口类
public interface TestService {
    //eatfoods
    void eatFoods();

    //gotowork
    void gotoWork();
}

//具体实现类
@Component
public class TestServiceImpl implements TestService {
    public void eatFoods() {
        System.out.println("eat foods");
        gotoWork();
    }

    public void gotoWork() {
        System.out.println("go to work");
    }
}
//前置增强切面
@Aspect
public class TestAdvanced {

    @Before("this(com.smart.mytest.TestService)")
    public void beforeAdvanced(){
        System.out.println("have a rest");
    }
}
//测试方法
public class MyTest {
    public static void main(String[] args) {

        ApplicationContext context = new ClassPathXmlApplicationContext("com/smart/mytest/mytest.xml");
        TestService testService = (TestService) context.getBean("testService");
        testService.eatFoods();
    }
}

```
上述代码执行的结果为:
```java
have a rest
eat foods
go to work
```
这有点超乎意外, gotoWork()方法并没有执行前置增强.

# 分析原因

Spring AOP实现底层为JDK的动态代理或者CGLIB代理,就是为目标类生成新的子类,子类继承或者实现目标类的接口并覆盖其方法.
例如本例中会生成新的代理类实现.
TestService 接口,并覆写TestService的接口的两个方法,使其执行自己原有逻辑前先执行代理方法.大致内容如下:
1 .先执行	com.smart.mytest.TestAdvanced.TestAdvanced();
2 .执行 com.smart.mytest.TestServiceImpl.eatFoods();
而eatFoods() 内部调用的gotoWork() 隐藏了关键字 this
在上述代码输出时添加 this.getClass().getName()
```java
//testService 实际是代理对象的引用
testService.getClass().getName():com.sun.proxy.$Proxy7
//增强对象
com.smart.mytest.TestAdvanced     have a rest
//TestServiceImpl对象
com.smart.mytest.TestServiceImpl      eat foods
//TestServiceImpl
com.smart.mytest.TestServiceImpl     go to work
```
从上述代码可以看出 最终调用 gotoWork() 方法的并非代理对象而是this指向的本例对象所以没有织入增强.

# 解决方案

[参考Spring文档介绍](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-proxying) . 
1 .重构代码使代码中不出现内部调用.
2 .在调用内部方法时使用下面方法代替this
```java
((TestService)AopContext.currentProxy()).gotoWork();
```
并在配置文件中添加属性,expose-proxy = "true"
```java
<aop:aspectj-autoproxy  expose-proxy="true"/>
```
此时将正常获取当前的代理对象,这样就可以满足需求了.