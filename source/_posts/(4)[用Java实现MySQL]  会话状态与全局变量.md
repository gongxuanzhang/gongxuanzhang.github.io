---
title: (3) [用Java实现MySQL] 会话状态与全局变量
date: 2022-12-18 10:41:10
tags:
- 手写mysql

categories: 系列文章

---

### 全局变量的维护
初创项目的时候，我利用Spring的Environment接口实现了全局维护和默认配置的基本模型。

本章根据Request的特性封装成我们自己的MySqlSession
用SessionId 模拟真正开启的session窗口
要保证session数据和局部变量的隔离性
Session其实就是一个Map 关键是如何**管理**


### 如何获取Session?
建立Session管理器，因为整个系统的连接处使用了http，借由Servlet的request我们可以直接把session区分开来，用servlet的SessionId来作为唯一标识
不考虑Session过期
 一个非常简单的会话管理器就有了
```java
/**
 * 会话管理器
 *
 * @author gxz gongxuanzhang@foxmail.com
 **/
public class SessionManager {

    private final static Map<String, MySqlSession> SESSION_BOX = new ConcurrentHashMap<>();


    public static MySqlSession currentSession() {
        RequestAttributes requestAttributes = RequestContextHolder.currentRequestAttributes();
        return SESSION_BOX.computeIfAbsent(requestAttributes.getSessionId(), MySqlSession::new);
    }

}
```

### 何时清理Session?

因为Session不过期的性质，如果我们开启一个窗口将会一直获取使用Session没有问题.
但是如果Session超过了最大数量，是需要拒绝新开启的Session的
所以直接从连接层提供。
但是这里Http的无状态的弊端就展现了，当退出会话的时候就 服务器是可能不知道的。
不过期 --> 和无状态  两个特性产生了冲突
最终我还是决定把Session变成有过期时间的  默认十分钟 
然后在SpringMVC的拦截器中加入了会话续期的内容

一张图表示会话续期的过程
<img src="/images/mysql/session.png" alt="session"  />

SessionManager 就变成了
```java
public class SessionManager {

    private static TimedCache<String, MySqlSession> SESSION_BOX;


    public static void init() {
        GlobalProperties global = GlobalProperties.getInstance();
        String sessionCount = global.get(PropertiesConstant.MAX_SESSION_COUNT);
        SESSION_BOX = new TimedCache<>(Integer.parseInt(sessionCount));
    }


    public static MySqlSession currentSession() throws SessionException {
        RequestAttributes requestAttributes = RequestContextHolder.currentRequestAttributes();
        MySqlSession mySqlSession = SESSION_BOX.get(requestAttributes.getSessionId());
        if (mySqlSession != null) {
            return mySqlSession;
        }
        String sessionId = requestAttributes.getSessionId();
        GlobalProperties global = GlobalProperties.getInstance();
        mySqlSession = new MySqlSession(sessionId);
        if (!SESSION_BOX.put(sessionId, mySqlSession, Integer.parseInt(global.get(SESSION_DURATION)))) {
            throw new SessionException("会话创建失败");
        }
        return mySqlSession;
    }


}
```
