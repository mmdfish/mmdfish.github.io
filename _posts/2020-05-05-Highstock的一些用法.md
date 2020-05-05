---
layout:     post
title:      第七篇 Highstock的一些用法
subtitle:   
date:       2020-05-05
author:     Mmdfish
header-img: 
catalog: true
tags:
    - HighCharts 
    - Highstock 
    - candlestick

---

# 关键词

HighCharts Highstock candlestick

# Github 

[highstock_demo](https://github.com/mmdfish/highstock_demo)
[CandleChart.vue](https://github.com/mmdfish/stockspec_vue/blob/master/stockspec/src/components/CandleChart.vue)

# HighStock介绍

[Highstock ](https://www.highcharts.com.cn/docs/highstock)
是基于 Highcharts 创建的专门用于股票图及大数据了时间轴图表，也就是意味着 Highstock 包含 Highcharts 所有功能，只是在 Highcharts 的基础上增加了新的功能，另外 Highstock 支持 K线图、蜡烛图等股票金融专用图表。我这里主要是实现K线图(蜡烛图)。

# HighStock在vue里应用

highcharts官方开发了Vue的扩展包，[highcharts-vue](https://www.highcharts.com.cn/docs/highcharts-vue)，Highcharts-Vue 扩展包默认使用 chart 的构造函数，使用 stockChart ，只需要导入 stock 模块，并在组件元素中使用 :constructor-type 参数。在data()中定义好chartOptions，传给 :options参数。

```
import Highcharts from 'highcharts'
import stockInit from 'highcharts/modules/stock'

stockInit(Highcharts) 
<highcharts :constructor-type="'stockChart'" :options="chartOptions"></highcharts>

data() {
        return {
            chartOptions: {
                series: [{
                    data: [1, 2, 3] // sample data
                }]
            }
        }
    }
```

# HighStock的汉化

HighStock 默认是英文的，如果想改成中文的话，需要用Highcharts.setOptions() 方法来定义一些将在所有图表上设置的全局参数，语言是 lang。最好在运用程序的主文件中(main.js)使用它，并且在这之前需要导入 Highcharts 包。
这里我们把月和星期几改成中文了。
修改前

![](https://upload-images.jianshu.io/upload_images/18184331-81c715ef774d36e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
import Highcahrts from 'highcharts'
Highcharts.setOptions({
  global: {
   useUTC: false
  },
  lang:{
   months:['一月', '二月', '三月', '四月', '五月', '六月', '七月', '八月','九月',  '十月','十一月', '十二月'],
   weekdays:['星期日',  '星期一', '星期二', '星期三', '星期四', '星期五', '星期六'],
   shortMonths: [ '01', '02', '03', '04', '05', '06', '07', '08', '09', '10', '11', '12'],
   }
});
```

修改后

![](https://upload-images.jianshu.io/upload_images/18184331-62f6fa74e4b1ad20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当然lang只能修改一部分内容。比如图中的open high low close只能通过tooltips来修改。这里我没有修改，可以参考[highstock#tooltip](https://api.highcharts.com.cn/highstock#tooltip)里的demo。

# K线图数据和成交量数据

K线图中的每个数据点包含五个数据，分别是 时间戳, 开盘价, 最高价, 最低价, 收盘价，如下：

```
data: [
    [1147651200000, 67.37, 68.38, 67.12, 67.79],
    [1147737600000, 68.1, 68.25, 64.75, 64.98]
]
```

成交量图数据每个数据点包含两个数据，即时间戳和成交量，如下：
```
data: [
    [1147651200000, 689567.44],
    [1147737600000, 1216478]
]
```

# 设置数据

在实际项目中我们的数据来源于Django的后端返回的json，这里为了方便展示，我把数据放在了stockData.js文件中。
六个数据分别为时间戳, 开盘价, 最高价, 最低价, 收盘价和成交量。

```
export const data = [
    [1416182400000, 7.204, 7.232, 7.037, 7.072, 889594.5],
    [1416268800000, 7.058, 7.107, 6.933, 6.968, 930296.44],
    ...
```

如果只是做简单展示，那直接把data写在series里，但实际应用中一般都需要在vue的created()中赋值。根据stockOptions的层级结构来获取想要赋值的对象。

```
import { data } from './stockData.js'
const stockData = data.map(d => d.slice(0.2))
export default {
	data() {
		return {
			chartOptions: {
			  series: [{
				type: 'candlestick',
				data: [],
			}]
			}	
		}
	}
	
	created() {
		let dataLength = stockData.length;
		let ohlc = []
		for (i = 0; i < dataLength; i += 1) {
			timeStamp = stockData[i][0];
			ohlc.push([
			  timeStamp, // the date
			  stockData[i][1], // open
			  stockData[i][2], // high
			  stockData[i][3], // low
			  stockData[i][4] // close
			]);
		}
                //根据层级结构赋值
		this.chartOptions.series[0].data = ohlc;
	}
}
```

# K线图颜色和成交量柱颜色

K线图的type为'candlestick'，K线图颜色可以通过通过 series.color 和 series.lineColor 来控制走势为跌的柱形颜色和线条颜色，series.upColor 和 series.upLineColor 来控制走势为涨的柱形颜色和线条颜色。

```
chartOptions: {
  series: [{
    type: 'candlestick',
    data: [],
    // 控制走势为跌的蜡烛颜色
    color: 'green',
    lineColor: 'green',
    // 控制走势为涨的蜡烛颜色
    upColor: 'red',
    upLineColor: 'red'
}]
}
```

成交量图的type为'column'，可以在data里动态设置颜色，

```
for (i = 0; i < dataLength; i += 1) {
        timeStamp = stockData[i][0];
		volume.push({
				  x: timeStamp,
				  y: stockData[i][5], // the date
				  color: stockData[i][1] > stockData[i][4] ? 'green' : 'red'
				});
}
this.chartOptions.series[1].data = volume;	

```

# x轴的最小范围 minRange和蜡烛柱的宽度maxPointWidth

k线图最下面的部分是highstock的scrollbar，当范围特别小的时候，比如只显示两三天的K线图，K柱会间隔得非常开，如果设置chartOptions.xAxis.minRange为30天，那这个scrollbar最小只能缩小到30天。不过即使是30天，K线柱还是会有点大，我们可以设置[series<candlestick>.maxPointWidth](https://api.highcharts.com.cn/highstock#series%3Ccandlestick%3E.maxPointWidth)，这样就算是数据很少时K线柱宽度最大只能到某个值。

![](https://upload-images.jianshu.io/upload_images/18184331-e4b64a43b6eb9289.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
chartOptions: {
        xAxis: {
          dateTimeLabelFormats: {
            millisecond: "%H:%M:%S.%L",
            second: "%H:%M:%S",
            minute: "%H:%M",
            hour: "%H:%M",
            day: "%m-%d",
            week: "%m-%d",
            month: "%y-%m",
            year: "%Y"
          }, 
          //3600000是一小时，所以minRange为30天
          minRange:30*24 *3600000
        },
        series: [
          {
            type: "candlestick",
            data: [],
            maxPointWidth: 5,
          },
          {
            type: "column",
            name: "成交量(亿)",
            data: [],
            maxPointWidth: 7,
          },
}
```
K线图和成交量柱状图都可以设置maxPointWidth，这里可能是因为k线图外框的关系，所以K线图的宽度需要设置比成交量柱状图小一些。

# 加入空数据

继续上一个问题，比如某只股票新上市刚一天或几天，就算我们设置minRange为30，实际数据并没有30天的数据，以我们stockdata.js里的data2为例，只有五天的数据，间隔还是很大。

![](https://upload-images.jianshu.io/upload_images/18184331-8ccdb64bf02a22b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

有没有办法左对齐，并让他们间隔不那么宽呢，这里我也找了很久，最后发现可以给数据里加入空的数据来解决这个问题。判断数据量是否不够20天，不够的话向后补齐，注意y的数据一定用null，如果给0的话鼠标悬浮上去还是会有显示。

```
if(dataLength < 20) {
        var latesttime = ohlc[dataLength - 1][0]
        for(i = dataLength; i < 20; i+=1) {
          latesttime += 24 *3600000
          volume.push({
            x: latesttime,
            y: null,
            color: 'white'
          });
        }
        this.chartOptions.series[1].data = volume;
        this.chartOptions.xAxis.min = ohlc[0][0];
        this.chartOptions.xAxis.max = ohlc[0][0] + 20*24 *3600000;
        this.chartOptions.xAxis.minRange = 30*24 *3600000;
      } else {
        this.chartOptions.xAxis.min = null;
        this.chartOptions.xAxis.max = null;
        this.chartOptions.xAxis.minRange = 30*24 *3600000;
      }
```
显示效果

![](https://upload-images.jianshu.io/upload_images/18184331-1798c389543694b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# y轴显示区间和刻度

highstock对于输入数据会有默认处理，就是默认计算刻度间隔和显示区间，比如下图中，highstock根据股价计算出最后显示区间为6-15，一共4个刻度，每个刻度间隔为3。但是看上去效果就是K线图集中在9-12这个区间，其他两个区间完全空出来，非常浪费。

![](https://upload-images.jianshu.io/upload_images/18184331-2ef10bd667e2f4a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

解决方法，设置[chartOptions.yAxis.tickPositioner](https://api.highcharts.com.cn/highstock#yAxis.tickPositioner)
函数，方法可以拿到dataMax和dataMin，然后根据自己的需求计算，这里我设置区间增长为五分之一的max-min，然后根据增长大于1还是小于1分别处理。

```
chartOptions: {
        yAxis: 
          {
           tickPositioner: function () {
              var positions = [],
                tick = 0,
                increment = (this.dataMax - this.dataMin) / 5,
                max = this.dataMax,
                min = this.dataMin;
                if(increment > 1) {
                  increment = Math.ceil(increment);
                  tick = Math.floor(this.dataMin);
                  for (tick; tick - increment <= this.dataMax; tick += increment) {
                    positions.push(tick);
                  }
                } else {
                  tick = Number(min.toFixed(1))
                  increment = Number(increment.toFixed(3))
                  for (tick; tick - increment <= this.dataMax; tick += increment) {
                    positions.push(Number(tick.toFixed(2)));
                  }
                }
              return positions;
	    }
      }
}
```

显示效果，利用率高了很多。

![](https://upload-images.jianshu.io/upload_images/18184331-557d2a78be25d083.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# ma日均线

highstock默认的是没有日均线，highstock默认带了很多[技术指标](https://www.highcharts.com.cn/docs/technical-indicator-series)，但是我没有在Vue中试出来，以后再尝试吧。均线这个比较好算，所以咱们自己来。

```
let ma = [];
let maset = [5,10,20,30];
for (i = 0; i < dataLength; i += 1) {
    for (let j = 0; j < maset.length; j++) {
          let value = maset[j];
          if(typeof ma['ma'+value] == "undefined"){
            ma['ma'+value]=[];
          }
          if(typeof ma[value+'total'] == "undefined"){
            ma[value+'total']=0;
          }
          if(i < value)
          {
            ma[value+'total'] += stockData[i][4];
            ma['ma'+value].push([timeStamp,null]);
          } else {
            ma[value+'total'] += (stockData[i][4] - stockData[i - value][4]);
            let kk = Number((ma[value+'total']/value).toFixed(2))
            ma['ma'+value].push([timeStamp, kk]);
          }   
        }
}
this.stockOptions.series[2].data = ma['ma5'];
this.stockOptions.series[3].data = ma['ma10'];
this.stockOptions.series[4].data = ma['ma20'];
this.stockOptions.series[5].data = ma['ma30'];
```

在chartOptions里series的定义，这里的yAxis和Candlestick的yAxis是同一个。
```
{
	type:'line',
	name:'ma5' ,
	data: [],
	color: "#000000",
	yAxis: 0,
	dataGrouping:[],
	lineWidth: 1,
}
```

# 禁用鼠标悬浮高亮

加入几条均线之后，当鼠标放在某个点的时候，所有的线都高亮了，看上去太花哨了，这里可以设置禁用高亮。
[plotOptions.series.states.hover.halo](https://api.highcharts.com.cn/highstock#plotOptions.series.states.hover.halo)

```
chartOptions: {
        plotOptions: {
            series: {
                states: {
                    hover: {
                        enabled: false
                    }
                },
            }
        },
}
```

# 获得highchart的对象

有时候我们需要在代码中用到highchart的对象，怎么去拿到它呢，我们可以用Vue的ref来做到。
首先我们需要在定义highchart组件时给它一个ref，这里我们设置 ref = "mystock"

```
<div><highcharts class="stock" :constructor-type="'stockChart'" :options="chartOptions" ref="mystock"></highcharts></div>
```
然后在js中用this.$refs.mystock.chart就可以获得实例化的chart对象，用console.log()可以看到里面的具体数据，然后获得相应的数据进行操作。

```
let chart = this.$refs.mystock.chart;
console.log(chart)
```

# 加入和大盘数据的对比

这个也是我自己想出来的需求，因为有贝塔系数和相关性的几个指标，都是跟大盘比较的，那是不是可以将大盘数据和该股票数据直观的显示出来。我们需要的效果是，每次scrollbar变化后，将显示区间中的第一天数据作为基准来对比大盘和股票的数据走势。
在仔细查看了HighStock的文档之后，发现了有[series<line>.compare](https://api.highcharts.com.cn/highstock#series%3Cline%3E.compare)
属性和Axis.setCompare()方法，但是不论是percent还是value，都会改变这个坐标轴的值，这个是不可以接受的。既然这样我们就新建一个坐标轴，这个坐标轴里只有股票的数据和大盘的数据。

```
yAxis: [
  {
	labels: {
	  align: "left",
	  x: -3
	},
	title: {
	  text: "股价(元)"
	},
	...
  },
  {
	labels: {
	  align: "right",
	  x: -3
	},
	title: {
	  text: "指数"
	},
	height: "65%",
	opposite: false,
        visible: false,
        ...
  },
  {
	labels: {
	  align: "left",
	  x: -3
	},
	title: {
	  text: "成交量(亿)"
	},
	top: "65%",
	height: "35%",
	offset: 0,
	lineWidth: 2
  }
],
```
现在图中就有3个y轴，第一个给K线和ma线用，第二个给股票和大盘对比用，第三个给成交量柱状图用。
第二个坐标轴我们设置成opposite: false就是这个坐标轴在图的左边，然后我们不让它显示，因为percent值也没有什么太大的意义。

然后在series里定义股票数据和大盘数据，对应第二个yAxis的index为1，设置大盘数据的compare为percent。因为我们这里不需要再画一条当前股票的数据，只是用来给大盘数据当基准的，所以股票数据的lineWidth设置为0，enableMouseTracking也设置为false，就是鼠标放上去也不会有tooltip。
```
      {
            type:'line',
            name:'大盘',
            data: [],
            color: "red",
            yAxis: 1,
            dataGrouping:[],
            lineWidth: 1,
            visible: true,
            compare: "percent"
          }
          ,{
            type:'line',
            name:'name',
            data: [],
            color: "white",
            yAxis: 1,
            dataGrouping:[],
            lineWidth: 0,
            visible: true,
            enableMouseTracking: false
          }
```

显示效果。这里我们大盘数据用的假数据，设置的是股票价格*200-random(100)。红线就是大盘的走势。拖动滚动轴可以看图里区间的第一天一直是和股票重合的。

![](https://upload-images.jianshu.io/upload_images/18184331-b417dfe493c823b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 动态设置显示某条线

有些人可能不想看和大盘的对比数据，那我们这里加一个checkbox，选中的话就显示，不选的话就不显示。
这里就需要用到我们之前说的获取chart实例对象了，我们设置checkbox的change方法为display。
在display方法里拿到chart的对象，然后设置大盘数据显示还是隐藏。

```
<el-checkbox @change="display" v-model="checked"><div style="color:red">对比大盘</div></el-checkbox>

<script>
export default {

  checked: false,

  data() {
    return {
      checked: false, //默认false
    },

methods: {
    display() {
      if(this.checked) {
        let chart = this.$refs.mystock.chart;
        chart.series[6].show();
      } else {
        let chart = this.$refs.mystock.chart;
        chart.series[6].hide();
      }
    },
```

以上就是我在使用highstock的过程中遇到的一些问题，highstock的文档和api还是蛮详细的，而且有很多的demo，是一个比较不错的开源画图库。
