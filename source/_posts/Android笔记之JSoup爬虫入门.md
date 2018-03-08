---
title: Android笔记之JSoup爬虫入门
date: 2017-04-13 22:26:09
tags: [Android]
---


## 前言
闲扯一些没用的，写这篇文章之前是有点私心的，因为之前评论某简书大v的文章是鸡汤，瞬间被拉黑，连个解释和说明的机会都没有，文章语言干涩，内容平平，于是就好奇到底是些什么样的人喜欢和吹捧这样的鸡汤作者。
所谓技术可以解惑答疑，所以我就爬来了该作者的所有的文章，每篇文章的阅读数，赞数，评论数，赞赏数，赞赏者，评论者，入选的专题。

1. 通过阅读数，赞数，评论数，赞赏数可以看出该作者的热度曲线，以及未来趋势。
2. 通过阅读数，赞数，评论数，赞赏数的一定权值得出查看人民群众最喜爱的文章排序。
3. 通过赞赏，评论者的次数和一定权值（赞赏权值更高），得出最忠实的粉丝。
4. 将题目和专题可以得出高频词汇。

<!-- more -->

这些数据不多，但是因为作者文章够多，所以足够用来为该作者和其分析进行简单的建模和画像。我看了其排名前十的文章，忠实粉丝，又看了看高频词汇，心口的那些疑问也就豁然开朗了。
但是转念一想，你看看简书首页推荐，也就差不多明白是什么当道，再者说了，作者写什么样的文章，什么水平的粉丝关注也本与我无关，腥的和素的谈不上好坏，更枉论高低。
所以尽管初衷与技术无关，最终的还是回归技术本身。

## JSoup使用
##### 简介
jsoup 是一款Java 的HTML解析器，可直接解析某个URL地址、HTML文本内容。它提供了一套非常省力的API，可通过DOM，CSS以及类似于jQuery的操作方法来取出和操作数据。
jsoup的主要功能如下：
1. 从一个URL，文件或字符串中解析HTML；
2. 使用DOM或CSS选择器来查找、取出数据；
3. 可操作HTML元素、属性、文本；

##### 依赖
```
compile 'org.jsoup:jsoup:1.9.2'
```
##### API
因为这里是JSoup的简单使用，并不能展现JSoup的强大功能，这里只用到了简单的JSoup的API和Html基础知识。

1. 获取Document
```
String baseURL = "http://www.jianshu.com/u/" + uid;
Document personalInfo= Jsoup.connect(baseURL).get();
```
2. 获取Elements
```
Elements noteList = document.select("ul.note-list");
Elements li = noteList.select("li");
```
3. 获得标签文本
```
String content = li.text().trim();
```

##### 文档
中文文档 : https://jsoup.org/cookbook/
英文文档 : http://www.open-open.com/jsoup/

## 文章信息爬取

##### 网页分析
这里需要简单的Html基础，简单的分析一下简书的作者文章信息页面的布局，找到相应的Elements,对于这个简单的入门的爬虫来说，这恐怕是最复杂的一个步骤了。

![2017-04-13_145642.png](http://upload-images.jianshu.io/upload_images/1806858-045086f87b975c38.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一目了然，所有关于这篇文章的所有内容都可以获取到。
下面我们简单的分析一下他的结构：

![](http://upload-images.jianshu.io/upload_images/1806858-e9bdd0064c436f97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![2017-04-13_151051.png](http://upload-images.jianshu.io/upload_images/1806858-12b87d1c6f1920f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 代码实现
代码很简单，将获取信息的代码放在子线程里面执行即可，当然这个爬虫并没有爬起来。
```
 new Thread() {
            @Override
            public void run() {
                int page = 1;

                try {
                    String uid = "1441f4ae075d";
                    String baseURL = "http://www.jianshu.com/u/" + uid;

                    Document personalInfo= Jsoup.connect(baseURL).get();
                    author.setName(personalInfo.select("a.name").text());
                    author.setArticleNum(Integer.parseInt(personalInfo.select("div.meta-block").select("p").get(2).text().trim()));

                    while (true){

                        Document doc = Jsoup.connect(baseURL +"?page=" + page).get();
                        Elements noteList = doc.select("ul.note-list");
                        Elements li = noteList.select("li");

                        if (li == null || articleList.size() == author.getArticleNum()){
                            break;
                        }
                        else{
                            for (Element element : li) {

                                Article article = new Article();
                                article.setTitle(element.select("a.title").text());
                                article.setAbstractStr(element.select("p.abstract").toString());
                                article.setReadNum(Integer.parseInt(element.select("div.meta").get(0).select("a").get(0).text().trim()));
                                article.setCommentNum(Integer.parseInt(element.select("div.meta").get(0).select("a").get(1).text().trim()));
                                article.setLikeNum(Integer.parseInt(element.select("div.meta").get(0).select("span").get(0).text().trim()));

                                if (element.select("div.meta").get(0).select("span").size() == 2) {
                                    article.setMoneyNum(Integer.parseInt(element.select("div.meta").get(0).select("span").get(1).text().trim()));
                                }

                                if(!articleList.contains(article)){

                                    articleList.add(article);

                                    System.out.println(article.getTitle());
                                    System.out.println(article.getReadNum());
                                    System.out.println(article.getCommentNum());
                                    System.out.println(article.getLikeNum());
                                    System.out.println(article.getMoneyNum());

                                }
                            }
                            page++;
                        }
                    }
                    System.out.println("总共获得文章篇数 ：" + articleList.size());
                }
                catch(Exception e){
                    e.printStackTrace();
                    System.out.println("Exception");
                }
            }
        }.start();
```