---
layout:     post
title:      用Vue+Django实现一个简单的股票指标网站
subtitle:   
date:       2020-04-28
author:     Mmdfish
header-img: 
catalog: true
tags:
    - Python
    - Django
    - BaoStock
    - Vue
    - Highcharts

---

#关键词：

Python Django BaoStock Numpy Pandas
Vue.js highcharts

#动机：

刚刚开始接触股票，其实基本各个股票网站都有各种数据，但是想找一些关于个股对应大盘的波动或者是和大盘的相关性的指标却不太容易。突发奇想自己做一个吧。做完之后想着怎么展示，正好之前做直播导航只做完后端没做前端，趁着这个机会逼自己把前端做出来。

#思路：

做数据分析，最合适的当然是Python，毕竟好用的库太多了。

第一步当然是要拿到股票的交易数据，最开始的时候想着是做一个爬虫去抓股票网站的数据，后来想着Python的第三方库这么多，会不会有直接可以拿股票数据的库。搜索了一下发现真的有个tushare，用了一会发现很多功能都需要积分才能使用，继续找，发现还有一个库叫[BaoStock](http://baostock.com/baostock/index.php/)
，试用了一会发现完全可以满足自己的需求，而且无需注册不用积分，速度上感觉比tushare还要快一些。简单对比后果断选择了BaoStock来做。关于Tushare和BaoStock的对比大家可以自己试试。

拿到数据后就是计算指标，计算了贝塔系数，相关性系数，振幅，之后可能会加一些新的指标。
然后用Django做后端，Vue做前端，自己在学习Vue的过程中，使用了[ElementUI](https://element.eleme.cn/#/zh-CN/)和[Highcharts](https://www.highcharts.com.cn/)，其中遇到很多问题也都记录下来了。

先展示一下效果，前端很简单：
![](https://upload-images.jianshu.io/upload_images/18184331-882bbf5b94a82f5b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/18184331-66067b8238802e1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

