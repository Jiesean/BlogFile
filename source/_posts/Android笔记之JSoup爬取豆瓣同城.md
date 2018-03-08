---
title: Android笔记之JSoup爬取豆瓣同城
date: 2017-05-19 22:20:40
tags: [Android]
---


### 前言
该实现异常简单，甚至都不能叫做爬虫，并不需要任何的技术，但是可以方便自己的生活，那就是有用的。

鉴于前面刚刚学习了JSoup来实现简单的爬虫，这次爬取了豆瓣同城的来为自己图个方便。
事情是这样的，每个周末我都习惯去豆瓣同城上去找一下有什么好看的摄影展或者画展，但是呢，这样的展览相对较少，在豆瓣同城上去查找需要翻很多页才能找到一个，还未必感兴趣，于是寻思写一个简单的爬虫为自己爬一下豆瓣同城的摄影和画展的相关内容，就方便多了。

<!-- more -->

### 实现
##### 网页分析
在简单的爬虫实现中，该步骤是最好费时间的，把每一个想要获取的信息，找到他多对应的元素，不多说。
因为我们是以找到同城的摄影展为主要目的的，因此我们可以直接从豆瓣同城-北京-展览这个首页开始爬取信息，这样减少了爬取信息的数量，减小了难度。

F12查看豆瓣网页源码的时候，控制台上打出这样一句话：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1806858-cc680bf7a5951ef5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
对程序员的招聘，无孔不入啊。
##### 爬取内容

利用Jsoup获取每一页面的document然后在选择出相应的元素，得到想要的内容，然后呈现在屏幕上就好了。
```
 document = Jsoup.connect("https://beijing.douban.com/events/week-exhibition?start=" + index + "0")
//                                .proxy("222.74.225.231",3128)
//                                .proxy("118.144.149.200",3128)
                                .timeout(10000)
                                .get();
                        Elements li = document.select("li.list-entry");
                        if (li.size() == 0) {
                            break;
                        }

                        for (Element element : li) {
                            Elements meta = element.select("ul.event-meta");
                            if (meta.toString().equals("")) {
                                continue;
                            }
                            LocalBean bean = new LocalBean();
                            bean.setImgURL(element.select("img").attr("data-lazy")); // 图片链接
                            bean.setTitle(element.select("div.title").select("a").attr("title"));//标题
                            bean.setURL(element.select("div.title").select("a").attr("href"));//链接

                            Elements tagElements = element.select("p.event-cate-tag hidden-xs").select("a");
                            if (!tagElements.toString().equals("")) {
                                for (int i = 0; i<tagElements.size() ; i++){
                                    bean.setTag(tagElements.get(i).text());
                                }
                            }
                            bean.setTime(meta.select("li.event-time").text());
                            bean.setLocation(meta.select("li").get(1).text());
                            bean.setCost(meta.select("li.fee").select("strong").text());

                            if (!bean.isWanted()) {
                                continue;
                            }

                            System.out.println("***************************");
                            System.out.println(bean.getTitle());
                            System.out.println(bean.getTime());
                            System.out.println(bean.getImgURL());
                            System.out.println(bean.getURL());
                            System.out.println(bean.getLocation());
                            System.out.println(bean.getCost());
                            System.out.println(bean.getTag());

                            mBeans.add(bean);
                        }
```
##### 筛选信息
我们目的主要是摄影，其次是画展，而且信息严谨性不是很强，可能存在漏选的情况。

所以我做的很简单，只是将标题和描述中的文字进行过滤，其中包含包含摄影相关的关键字的筛选出来。
##### 应对反爬
实际的爬虫实现中，应对反爬策略是很重要的一个环节，例如我们不间断的爬取豆瓣同城的话，连续几次之后就会被封IP, 然后我们就看到了403错误，这个时候我们就需要用代理IP进行访问了。
但因为本应用并没有频繁爬取数据的需求，数据量也是较小，因此没必要使用代理IP池的方式，打一枪换一炮，我们正常的每天只需要爬取一遍，我们只需要降低一下爬取的速度，每爬取一页的内容让线程休眠一会，然后继续爬取下一页，这样就降低了IP被封的几率。
```
05-09 18:52:03.669 20699-25842/com.jiesean.exhibitionspider W/System.err: org.jsoup.HttpStatusException: HTTP error fetching URL. Status=403, URL=https://beijing.douban.com/events/week-exhibition?start=1890
05-09 18:52:03.670 20699-25842/com.jiesean.exhibitionspider W/System.err:     at org.jsoup.helper.HttpConnection$Response.execute(HttpConnection.java:590)
05-09 18:52:03.671 20699-25842/com.jiesean.exhibitionspider W/System.err:     at org.jsoup.helper.HttpConnection$Response.execute(HttpConnection.java:540)
05-09 18:52:03.671 20699-25842/com.jiesean.exhibitionspider W/System.err:     at org.jsoup.helper.HttpConnection.execute(HttpConnection.java:227)
05-09 18:52:03.671 20699-25842/com.jiesean.exhibitionspider W/System.err:     at org.jsoup.helper.HttpConnection.get(HttpConnection.java:216)
05-09 18:52:03.671 20699-25842/com.jiesean.exhibitionspider W/System.err:     at com.jiesean.exhibitionspider.MainActivity$3.run(MainActivity.java:104)
05-09 18:52:03.671 20699-25842/com.jiesean.exhibitionspider W/System.err:     at java.lang.Thread.run(Thread.java:761)
```

### Demo
![](http://upload-images.jianshu.io/upload_images/1806858-3989d99191d47c9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/222)
![](http://upload-images.jianshu.io/upload_images/1806858-0c254484ce9f4d15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/222)

这个简单的小应用只是方便我个人的生活，花点小时间，倒是为自己节省了不少的时间。

DEMO地址：[Github传送门](https://github.com/Jiesean/ExhibitionSpider)
