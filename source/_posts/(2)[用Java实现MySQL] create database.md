---
title: (2)[用Java实现MySQL] create database
date: 2022-12-13 11:41:10
tags:
- 手写mysql
categories: 系列文章

---


## 从创建一个数据库开始
表是属于数据库的，我们对数据的操作也是都是基于某个数据库。
所以要有一个 创建数据库的功能，基于这个最简单的功能把整个流程搭建起来
首先写两个接口，一个解析器  一个执行器
```java
/**
 * 执行器，一般是解析Sql得到的
 *
 * @author gxz gongxuanzhang@foxmail.com
 * @see org.gongxuanzhang.mysql.service.parser.SqlParser
 **/
public interface Executor {

    /**
     * 执行
     *
     * @return 返回执行结果
     **/
    Result doExecute() ;
}
```
```java
/**
 * Sql 语法分析器
 *
 * @author gxz gongxuanzhang@foxmail.com
 **/
public interface SqlParser {

    /**
     * 解析sql成可执行对象
     *
     * @param sql 被解析的sql
     * @return 返回可执行内容 或者直接报错
     * @throws SqlParseException 解析过程中出现问题
     **/
    Executor parse(String sql) throws SqlParseException;

}

```
这两个接口作为我们整个流程的总接口使用

流程简化用一张图表示

<img src="/images/mysql/flow.png" alt="framework"  />


剩下的就是开始编码。

有一个小细节
>  如果用户没有提供 dataDir 数据根目录，那我们需要给一个默认目录同时创建文件夹
>  我在Spring声明周期中加了一个初始化根目录的类 


```java
/**
 * 如果没配置data dir 给默认值
 * 同时创建文件夹
 *
 * @author gxz gongxuanzhang@foxmail.com
 **/
@Slf4j
public class MySQLInit implements EnvironmentPostProcessor {


    @Override
    public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
        String dataDir = environment.getProperty(PropertiesConstant.DATA_DIR);
        if (dataDir == null) {
            File db = new File("db");
            dataDir = db.getAbsolutePath();
            MutablePropertySources propertySources = environment.getPropertySources();
            Map<String, Object> map = new HashMap<>(4);
            map.put(PropertiesConstant.DATA_DIR, dataDir);
            propertySources.addLast(new MapPropertySource("mysql", map));
        }
        File db = new File(dataDir);
        if (!db.exists()) {
            db.mkdirs();
        }
    }
}
```

今天搭建了初版代码，大概900多行   整体流程已经搭建好了 
持续更新吧  
github: [https://github.com/gongxuanzhang/java-mysql](https://github.com/gongxuanzhang/java-mysql)

## 

## 



## 






