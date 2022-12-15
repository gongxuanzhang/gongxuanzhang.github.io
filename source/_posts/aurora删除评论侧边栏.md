---
title: hexo Aurora 删除评论侧边栏
date: 2022-11-09 11:41:10
tags:
- 一些黑科技
categories: 开发相关

---


# hexo Aurora 去掉评论区

## 问题：评论侧边栏 

今天刚搭建了博客，用了hexo Aurora主题

整体主题挺满意的，但是有两个地方非常难受。

<img src="/images/image-20221109094356058.png" alt="image-20221109094356058" style="zoom: 50%;" />



在配置文件中设置了不启用之后变成了这样

<img src="/images/image-20221109094501762.png" alt="image-20221109094501762" style="zoom:50%;" />



仅仅是不请求接口了，但是侧边栏仍然存在，看了文档和github，好多人都有这个问题。下面来解决这个问题

## 解决

首先"最近评论"这几个字一定有几个地方存在，那我们搜索一下包含最近评论的文件，看看能不能找到端倪

```shell
直接控制台搜索
grep -i -r '最近评论' .
```

![image-20221109095002608](/images/image-20221109095002608.png)

好的 我们发现了一个js文件.编辑他。



这是个打包后的js文件，方法名都是没有意义的，都是编号。

没关系我们格式化之后直接定位具体内容

```js
"9abb": function (e) {
        e.exports = JSON.parse('{"menu":{"home":"首页","about":"关于","archives":"归档","categories":"分类","tags":"标签","post":"文章","message-board":"留言板","search":"搜索结果","not-found":"无法找到页面"},"home":{"recommended":"推荐文章"},"titles":{"articles":"文章列表","about":"关于我","category_list":"分类目录","tag_list":"标签目录","toc":"文章目录","comment":"评论区","recent_comment":"最近评论"},"settings":{"months":["一月","二月","三月","四月","五月","六月","七月","八月","九月","十月","十一月","十二月"],"articles":"文章","categories":"分类","tags":"标签","words":"文字","visitor_count":"总共访客数","visit_count":"总共访问数","button-all":"全部","paginator":{"newer":"新的","older":"以往","prev":"上一篇更回味","next":"下一篇更精彩"},"more-tags":"查看更多","admin-user":"博主","shared-on":"发布于","recently-search":"最近搜索","search-result":"一共找到 [total] 个结果","no-recent-search":"没有最近搜索记录。","no-search-result":"没有找到任何记录。","cmd-to-select":"查看","cmd-to-navigate":"选择","cmd-to-close":"关闭","searched-by":"搜索引擎","tips-back-to-top":"返回顶部","tips-open-menu":"打开菜单","tips-back-to-home":"返回首页","tips-open-search":"打开搜索","default-category":"文章","default-tag":"未分类","empty-tag":"目前没有标签","pinned":"置顶","featured":"推荐"}}')
    }
```

好消息是这个最近评论只有这一个地方出现过。

坏消息是这个json之后做了什么也不得而知。

先尝试第一步，修改一下文本内容看看生不生效

<img src="/images/image-20221109095605020.png" alt="image-20221109095605020" style="zoom:50%;margin-left:0px" />



重启一下看看效果

<img src="/images/image-20221109100320231.png" alt="image-20221109100320231" style="zoom:50%;margin-left:0px" />

> 很好，已经修改成功了。下一步就是把他干掉



这个评论内容在json中的key是recent_comment

全局搜索哪里用到了这个字段

![image-20221109102813197](/images/image-20221109102813197.png)

发现了这么一段代码，方法名是ie.继续找哪里用到了ie

<img src="/images/image-20221109102846918.png" alt="image-20221109102846918" style="zoom:50%;margin-left:0px" />

直接注掉这行代码。

<img src="/images/image-20221109102934094.png" alt="image-20221109102934094" style="zoom:50%;margin-left:0px" />

重启之后成功删除

<img src="/images/image-20221109103015680.png" alt="image-20221109103015680" style="zoom:50%;margin-left:0px" />


