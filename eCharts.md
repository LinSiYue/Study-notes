# eCharts学习笔记

## 一、项目用到的模板

### 1、富文本Nested Pies

显示效果：

![1570585796495](C:\Users\LINRE\AppData\Roaming\Typora\typora-user-images\1570585796495.png)

代码示例：

```javascript
app.title = '嵌套环形图';

option = {
    tooltip: {
        trigger: 'item',
        formatter: "{a} <br/>{b}: {c} ({d}%)"
    },
    legend: {
        orient: 'vertical',
        x: 'left',
        data:['直达','营销广告','搜索引擎','邮件营销','联盟广告','视频广告','百度','谷歌','必应','其他']
    },
    series: [
        {
            name:'访问来源',
            type:'pie',
            selectedMode: 'single',
            radius: [0, '30%'],

            label: {
                normal: {
                    position: 'inner'
                }
            },
            labelLine: {
                normal: {
                    show: false
                }
            },
            data:[
                {value:335, name:'直达', selected:true},
                {value:679, name:'营销广告'},
                {value:1548, name:'搜索引擎'}
            ]
        },
        {
            name:'访问来源',
            type:'pie',
            radius: ['40%', '55%'],
            label: {
                normal: {
                    // 此处|前的a，b，abg，hr是自定义的样式模版；
                    // |后的a表示系列名series的name，b表示数据名data的name,
                    // c表示数据值value，d表示所占百分比;|后的属性只能使用一次
                    formatter: '{a|{a}}{abg|}\n{hr|}\n  {b|{b}：}{c}  {per|{d}%}  ',
                    backgroundColor: '#eee',
                    borderColor: '#aaa',
                    borderWidth: 1,
                    borderRadius: 4,
                    // shadowBlur:3,
                    // shadowOffsetX: 2,
                    // shadowOffsetY: 2,
                    // shadowColor: '#999',
                    // padding: [0, 7],
                    rich: {
                        a: {
                            color: '#999',
                            lineHeight: 22,
                            align: 'center'
                        },
                        // abg: {
                        //     backgroundColor: '#333',
                        //     width: '100%',
                        //     align: 'right',
                        //     height: 22,
                        //     borderRadius: [4, 4, 0, 0]
                        // },
                        hr: {
                            borderColor: '#aaa',
                            width: '100%',
                            borderWidth: 0.5,
                            height: 0
                        },
                        b: {
                            fontSize: 16,
                            lineHeight: 33
                        },
                        per: {
                            color: '#eee',
                            backgroundColor: '#334455',
                            padding: [2, 4],
                            borderRadius: 2
                        }
                    }
                }
            },
            data:[
                {value:335, name:'直达'},
                {value:310, name:'邮件营销'},
                {value:234, name:'联盟广告'},
                {value:135, name:'视频广告'},
                {value:1048, name:'百度'},
                {value:251, name:'谷歌'},
                {value:147, name:'必应'},
                {value:102, name:'其他'}
            ]
        }
    ]
};
```

### 2、雷达图

显示效果：

![1570586089910](C:\Users\LINRE\AppData\Roaming\Typora\typora-user-images\1570586089910.png)

代码示例：

```vue
<template>
  <div :class="className" :style="{height:height,width:width}" />
</template>

<script>
import echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import resize from './mixins/resize'

const animationDuration = 3000

export default {
  mixins: [resize],
  props: {
    className: {
      type: String,
      default: 'chart'
    },
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '300px'
    }
  },
  data() {
    return {
      chart: null
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.initChart()
    })
  },
  beforeDestroy() {
    if (!this.chart) {
      return
    }
    this.chart.dispose()
    this.chart = null
  },
  methods: {
    initChart() {
      this.chart = echarts.init(this.$el, 'macarons')

      this.chart.setOption({
        tooltip: {
          trigger: 'axis',
          axisPointer: { // 坐标轴指示器，坐标轴触发有效
            type: 'shadow' // 默认为直线，可选为：'line' | 'shadow'
          }
        },
        radar: {
          radius: '66%',
          center: ['50%', '42%'],
          splitNumber: 8,
          splitArea: {
            areaStyle: {
              color: 'rgba(127,95,132,.3)',
              opacity: 1,
              shadowBlur: 45,
              shadowColor: 'rgba(0,0,0,.5)',
              shadowOffsetX: 0,
              shadowOffsetY: 15
            }
          },
          indicator: [
            { name: 'Sales', max: 10000 },
            { name: 'Administration', max: 20000 },
            { name: 'Information Techology', max: 20000 },
            { name: 'Customer Support', max: 20000 },
            { name: 'Development', max: 20000 },
            { name: 'Marketing', max: 20000 }
          ]
        },
        legend: {
          left: 'center',
          bottom: '10',
          data: ['Allocated Budget', 'Expected Spending', 'Actual Spending']
        },
        series: [{
          type: 'radar',
          symbolSize: 0,
          areaStyle: {
            normal: {
              shadowBlur: 13,
              shadowColor: 'rgba(0,0,0,.2)',
              shadowOffsetX: 0,
              shadowOffsetY: 10,
              opacity: 1
            }
          },
          data: [
            {
              value: [5000, 7000, 12000, 11000, 15000, 14000],
              name: 'Allocated Budget'
            },
            {
              value: [4000, 9000, 15000, 15000, 13000, 11000],
              name: 'Expected Spending'
            },
            {
              value: [5500, 11000, 12000, 15000, 12000, 12000],
              name: 'Actual Spending'
            }
          ],
          animationDuration: animationDuration
        }]
      })
    }
  }
}
</script>
```

### 3、地图散点图

显示效果：

![1570588391675](C:\Users\LINRE\AppData\Roaming\Typora\typora-user-images\1570588391675.png)

代码示例：

```javascript
var data = [
     {name: '海门', value: 9},
     {name: '鄂尔多斯', value: 12},
     {name: '招远', value: 12},
     {name: '舟山', value: 12},
     {name: '锦州', value: 54},
     {name: '南昌', value: 54},
     {name: '柳州', value: 54},
     {name: '三亚', value: 54},
     {name: '自贡', value: 56},
     {name: '吉林', value: 56},
     {name: '阳江', value: 57},
     {name: '泸州', value: 57},
     {name: '西宁', value: 57},
     {name: '宜宾', value: 58},
     {name: '洛阳', value: 134},
     {name: '秦皇岛', value: 136},
     {name: '株洲', value: 143},
     {name: '石家庄', value: 147},
     {name: '莱芜', value: 148},
     {name: '常德', value: 152},
     {name: '保定', value: 153},
     {name: '湘潭', value: 154},
     {name: '金华', value: 157},
     {name: '岳阳', value: 169},
     {name: '长沙', value: 175},
     {name: '衢州', value: 177},
     {name: '廊坊', value: 193},
     {name: '菏泽', value: 194},
     {name: '合肥', value: 229},
     {name: '武汉', value: 273},
     {name: '大庆', value: 279}
];
var geoCoordMap = {
    '海门':[121.15,31.89],
    '鄂尔多斯':[109.781327,39.608266],
    '招远':[120.38,37.35],
    '抚顺':[123.97,41.97],
    '玉溪':[102.52,24.35],
    '张家口':[114.87,40.82],
    '阳泉':[113.57,37.85],
    '莱州':[119.942327,37.177017],
    '湖州':[120.1,30.86],
    '汕头':[116.69,23.39],
    '昆山':[120.95,31.39],
    '宁波':[121.56,29.86],
    '湛江':[110.359377,21.270708],
    '揭阳':[116.35,23.55],
    '荣成':[122.41,37.16],
    '重庆':[106.54,29.59],
    '台州':[121.420757,28.656386],
    '九江':[115.97,29.71],
    '金华':[119.64,29.12],
    '岳阳':[113.09,29.37],
    '长沙':[113,28.21],
    '衢州':[118.88,28.97],
    '廊坊':[116.7,39.53],
    '菏泽':[115.480656,35.23375],
    '合肥':[117.27,31.86],
    '武汉':[114.31,30.52],
    '大庆':[125.03,46.58]
};

var convertData = function (data) {
    var res = [];
    for (var i = 0; i < data.length; i++) {
        var geoCoord = geoCoordMap[data[i].name];
        if (geoCoord) {
            res.push({
                name: data[i].name,
                value: geoCoord.concat(data[i].value)
            });
        }
    }
    return res;
};

option = {
    title: {
        text: '散点图城市分布',
        subtext: 'data from PM25.in',
        sublink: 'http://www.pm25.in',
        left: 'center'
    },
    tooltip : {
        trigger: 'item'
    },
    //地图区域样式
		    geo: {
		        map: 'china',
		        label: {
		            emphasis: {
		                show: false
		            }
		        },
		        itemStyle: {
		        	//未激活样式
		            normal: {
		                areaColor: 'rgb(10,63,107)',
		                borderColor: '#111'
		            },
		            //激活样式
		            emphasis: {
		                areaColor: '#2a333d'
		            }
		        }
		    },
    series : [
        {
            name: 'pm2.5',
            type: 'scatter',
            coordinateSystem: 'geo',
            data: convertData(data),
            symbolSize: function (val) {
                return val[2] / 10;
            },
            label: {
                normal: {
                    formatter: '{b}',
                    position: 'right',
                    show: false
                },
                emphasis: {
                    show: true
                }
            },
            itemStyle: {
                normal: {
                    color: 'rgb(217,222,42)'
                }
            }
        },
        {
            name: 'Top 5',
            type: 'effectScatter',
            coordinateSystem: 'geo',
            data: convertData(data.sort(function (a, b) {
                return b.value - a.value;
            }).slice(0, 6)),
            symbolSize: function (val) {
                return val[2] / 10;
            },
            showEffectOn: 'render',
            rippleEffect: {
                brushType: 'stroke'
            },
            hoverAnimation: true,
            label: {
                normal: {
                    formatter: '{b}',
                    position: 'right',
                    show: true
                }
            },
            itemStyle: {
                normal: {
                    color: 'rgb(217,222,42)',
                    shadowBlur: 10,
                    shadowColor: '#333'
                }
            },
            zlevel: 1
        }
    ]
};
```

### 4、折线图

显示效果：

![1570589438517](C:\Users\LINRE\AppData\Roaming\Typora\typora-user-images\1570589438517.png)

代码示例：

```javascript
option = {
    title: {
        text: '堆叠区域图'
    },
    tooltip : {
        trigger: 'axis',
        axisPointer: {
            type: 'cross',
            label: {
                backgroundColor: '#6a7985'
            }
        }
    },
    legend: {
        data:['邮件营销','联盟广告','视频广告','直接访问','搜索引擎']
    },
    toolbox: {
        feature: {
            saveAsImage: {}
        }
    },
    grid: {
        left: '3%',
        right: '4%',
        bottom: '3%',
        containLabel: true
    },
    xAxis : [
        {
            type : 'category',
            boundaryGap : false,
            data : ['周一','周二','周三','周四','周五','周六','周日']
        }
    ],
    yAxis : [
        {
            type : 'value'
        }
    ],
    series : [
        {
            name:'邮件营销',
            type:'line',
            //smooth:true, // 使折现变为光滑曲线
            stack: '总量',
            areaStyle: {},
            data:[120, 132, 101, 134, 90, 230, 210]
        },
        {
            name:'联盟广告',
            type:'line',
            //smooth:true, // 使折现变为光滑曲线
            stack: '总量',
            areaStyle: {},
            data:[220, 182, 191, 234, 290, 330, 310]
        },
        {
            name:'视频广告',
            type:'line',
            //smooth:true,
            stack: '总量',
            areaStyle: {},
            data:[150, 232, 201, 154, 190, 330, 410]
        },
        {
            name:'直接访问',
            type:'line',
            smooth:true,
            stack: '总量',
            areaStyle: {normal: {}},
            data:[320, 332, 301, 334, 390, 330, 320]
        },
        {
            name:'搜索引擎',
            type:'line',
            smooth:true,
            stack: '总量',
            label: {
                normal: {
                    show: true,
                    position: 'top'
                }
            },
            areaStyle: {normal: {}},
            data:[820, 932, 901, 934, 1290, 1330, 1320]
        }
    ]
};
```

### 5、堆叠条形图

显示效果：

![1570590055184](C:\Users\LINRE\AppData\Roaming\Typora\typora-user-images\1570590055184.png)

代码示例：

```javascript
app.title = '堆叠条形图';

option = {
    tooltip : {
        trigger: 'axis',
        axisPointer : {            // 坐标轴指示器，坐标轴触发有效
            type : 'shadow'        // 默认为直线，可选为：'line' | 'shadow'
        }
    },
    legend: {
        data: ['直接访问', '邮件营销','联盟广告','视频广告','搜索引擎']
    },
    grid: {
        left: '3%',
        right: '4%',
        bottom: '25%',
        containLabel: true,
        height:'55%'
    },
    xAxis:  {
        type: 'value'
    },
    yAxis: {
        type: 'category',
        data: ['周一','周二','周三','周四','周五','周六','周日']
    },
    series: [
        {
            name: '直接访问',
            type: 'bar',
            stack: '总量',
            label: {
                normal: {
                    show: true,
                    position: 'insideRight'
                }
            },
            barWidth:'20px',
            data: [320, 302, 301, 334, 390, 330, 320]
        },
        {
            name: '邮件营销',
            type: 'bar',
            stack: '总量',
            label: {
                normal: {
                    show: true,
                    position: 'insideRight'
                }
            },
            data: [120, 132, 101, 134, 90, 230, 210]
        },
        {
            name: '联盟广告',
            type: 'bar',
            stack: '总量',
            label: {
                normal: {
                    show: true,
                    position: 'insideRight'
                }
            },
            data: [220, 182, 191, 234, 290, 330, 310]
        },
        {
            name: '视频广告',
            type: 'bar',
            stack: '总量',
            label: {
                normal: {
                    show: true,
                    position: 'insideRight'
                }
            },
            data: [150, 212, 201, 154, 190, 330, 410]
        },
        {
            name: '搜索引擎',
            type: 'bar',
            stack: '总量',
            label: {
                normal: {
                    show: true,
                    position: 'insideRight'
                }
            },
            data: [820, 832, 901, 934, 1290, 1330, 1320]
        }
    ]
};
```

### 6、Multiple Y Axes

显示效果：

![1570590426566](C:\Users\LINRE\AppData\Roaming\Typora\typora-user-images\1570590426566.png)

代码示例：

```javascript
app.title = '多 Y 轴示例';

var colors = ['#5793f3', '#d14a61', '#675bba'];

option = {
    color: colors,
	title: {
        text: '多 Y 轴柱状图'
    },
    tooltip: {
        trigger: 'axis',
        axisPointer: {
            type: 'cross'
        }
    },
    grid: {
        right: '20%'
    },
    toolbox: {
        feature: {
            dataView: {show: true, readOnly: false},
            restore: {show: true},
            saveAsImage: {show: true}
        }
    },
    legend: {
        data:['蒸发量','降水量','平均温度']
    },
    xAxis: [
        {
            type: 'category',
            axisTick: {
                alignWithLabel: true
            },
            data: ['1月','2月','3月','4月','5月','6月','7月','8月','9月','10月','11月','12月']
        }
    ],
    yAxis: [
        {
            type: 'value',
            name: '蒸发量',
            min: 0,
            max: 250,
            position: 'right',
            axisLine: {
                lineStyle: {
                    color: colors[0]
                }
            },
            axisLabel: {
                formatter: '{value} ml'
            }
        },
        {
            type: 'value',
            name: '降水量',
            min: 0,
            max: 250,
            position: 'right',
            offset: 80,
            axisLine: {
                lineStyle: {
                    color: colors[1]
                }
            },
            axisLabel: {
                formatter: '{value} ml'
            }
        },
        {
            type: 'value',
            name: '温度',
            min: 0,
            max: 25,
            position: 'left',
            axisLine: {
                lineStyle: {
                    color: colors[2]
                }
            },
            axisLabel: {
                formatter: '{value} °C'
            }
        }
    ],
    series: [
        {
            name:'蒸发量',
            type:'bar',
            data:[2.0, 4.9, 7.0, 23.2, 25.6, 76.7, 135.6, 162.2, 32.6, 20.0, 6.4, 3.3]
        },
        {
            name:'降水量',
            type:'bar',
            yAxisIndex: 1,
            data:[2.6, 5.9, 9.0, 26.4, 28.7, 70.7, 175.6, 182.2, 48.7, 18.8, 6.0, 2.3]
        },
        {
            name:'平均温度',
            type:'line',
            yAxisIndex: 2,
            data:[2.0, 2.2, 3.3, 4.5, 6.3, 10.2, 20.3, 23.4, 23.0, 16.5, 12.0, 6.2]
        }
    ]
};
```

### 7、仪表盘

显示效果：

![1570593038621](C:\Users\LINRE\AppData\Roaming\Typora\typora-user-images\1570593038621.png)

代码示例：

```javascript
option = {
    tooltip : {
        formatter: "{a} <br/>{c} {b}"
    },
    toolbox: {
        show: true,
        feature: {
            restore: {show: true},
            saveAsImage: {show: true}
        }
    },
    // 连续型视觉映射
    visualMap: {
        orient: 'horizontal',
        left: 'center',
        min: 10,
        max: 100,
        type:'continuous',
        text: ['低','高'],
        // Map the score column to color
        dimension: 0,
        inRange: {
            color: ["rgba(194,53,49,1)", "rgba(99,134,158,1)", "rgba(145,199,174,1)"]
        }
    },
    series : [
        {
            name: '速度',
            type: 'gauge',
            z: 3,
            min: 0,
            max: 220,
            splitNumber: 11,
            radius: '50%',
            axisLine: {            // 坐标轴线
                lineStyle: {       // 属性lineStyle控制线条样式
                    width: 10
                }
            },
            axisTick: {            // 坐标轴小标记
                length: 15,        // 属性length控制线长
                lineStyle: {       // 属性lineStyle控制线条样式
                    color: 'auto'
                }
            },
            splitLine: {           // 分隔线
                length: 20,         // 属性length控制线长
                lineStyle: {       // 属性lineStyle（详见lineStyle）控制线条样式
                    color: 'auto'
                }
            },
            axisLabel: {
                backgroundColor: 'auto',
                borderRadius: 2,
                color: '#eee',
                padding: 3,
                textShadowBlur: 2,
                textShadowOffsetX: 1,
                textShadowOffsetY: 1,
                textShadowColor: '#222'
            },
            title : {
                // 其余属性默认使用全局文本样式，详见TEXTSTYLE
                fontWeight: 'bolder',
                fontSize: 20,
                fontStyle: 'italic'
            },
            detail : {
                // 其余属性默认使用全局文本样式，详见TEXTSTYLE
                formatter: function (value) {
                    value = (value + '').split('.');
                    value.length < 2 && (value.push('00'));
                    return ('00' + value[0]).slice(-2)
                        + '.' + (value[1] + '00').slice(0, 2);
                },
                fontWeight: 'bolder',
                borderRadius: 3,
                backgroundColor: '#444',
                borderColor: '#aaa',
                shadowBlur: 5,
                shadowColor: '#333',
                shadowOffsetX: 0,
                shadowOffsetY: 3,
                borderWidth: 2,
                textBorderColor: '#000',
                textBorderWidth: 2,
                textShadowBlur: 2,
                textShadowColor: '#fff',
                textShadowOffsetX: 0,
                textShadowOffsetY: 0,
                fontFamily: 'Arial',
                width: 100,
                color: '#eee',
                rich: {}
            },
            data:[{value: 40, name: 'km/h'}]
        },
        {
            name: '转速',
            type: 'gauge',
            center: ['20%', '55%'],    // 默认全局居中
            radius: '35%',
            min:0,
            max:7,
            endAngle:45,
            splitNumber:7,
            axisLine: {            // 坐标轴线
                lineStyle: {       // 属性lineStyle控制线条样式
                    width: 8
                }
            },
            axisTick: {            // 坐标轴小标记
                length:12,        // 属性length控制线长
                lineStyle: {       // 属性lineStyle控制线条样式
                    color: 'auto'
                }
            },
            splitLine: {           // 分隔线
                length:20,         // 属性length控制线长
                lineStyle: {       // 属性lineStyle（详见lineStyle）控制线条样式
                    color: 'auto'
                }
            },
            pointer: {
                width:5
            },
            title: {
                offsetCenter: [0, '-30%'],       // x, y，单位px
            },
            detail: {
                // 其余属性默认使用全局文本样式，详见TEXTSTYLE
                fontWeight: 'bolder'
            },
            data:[{value: 1.5, name: 'x1000 r/min'}]
        },
        {
            name: '油表',
            type: 'gauge',
            center: ['77%', '50%'],    // 默认全局居中
            radius: '25%',
            min: 0,
            max: 2,
            startAngle: 135,
            endAngle: 45,
            splitNumber: 2,
            axisLine: {            // 坐标轴线
                lineStyle: {       // 属性lineStyle控制线条样式
                    width: 8
                }
            },
            axisTick: {            // 坐标轴小标记
                splitNumber: 5,
                length: 10,        // 属性length控制线长
                lineStyle: {        // 属性lineStyle控制线条样式
                    color: 'auto'
                }
            },
            axisLabel: {
                formatter:function(v){
                    switch (v + '') {
                        case '0' : return 'E';
                        case '1' : return 'Gas';
                        case '2' : return 'F';
                    }
                }
            },
            splitLine: {           // 分隔线
                length: 15,         // 属性length控制线长
                lineStyle: {       // 属性lineStyle（详见lineStyle）控制线条样式
                    color: 'auto'
                }
            },
            pointer: {
                width:2
            },
            title : {
                show: false
            },
            detail : {
                show: false
            },
            data:[{value: 0.5, name: 'gas'}]
        },
        {
            name: '水表',
            type: 'gauge',
            center : ['77%', '50%'],    // 默认全局居中
            radius : '25%',
            min: 0,
            max: 2,
            startAngle: 315,
            endAngle: 225,
            splitNumber: 2,
            axisLine: {            // 坐标轴线
                lineStyle: {       // 属性lineStyle控制线条样式
                    width: 8
                }
            },
            axisTick: {            // 坐标轴小标记
                show: false
            },
            axisLabel: {
                formatter:function(v){
                    switch (v + '') {
                        case '0' : return 'H';
                        case '1' : return 'Water';
                        case '2' : return 'C';
                    }
                }
            },
            splitLine: {           // 分隔线
                length: 15,         // 属性length控制线长
                lineStyle: {       // 属性lineStyle（详见lineStyle）控制线条样式
                    color: 'auto'
                }
            },
            pointer: {
                width:2
            },
            title: {
                show: false
            },
            detail: {
                show: false
            },
            data:[{value: 0.5, name: 'gas'}]
        }
    ]
};

setInterval(function (){
    option.series[0].data[0].value = (Math.random()*100).toFixed(2) - 0;
    option.series[1].data[0].value = (Math.random()*7).toFixed(2) - 0;
    option.series[2].data[0].value = (Math.random()*2).toFixed(2) - 0;
    option.series[3].data[0].value = (Math.random()*2).toFixed(2) - 0;
    myChart.setOption(option,true);
},2000);
```

### 8、GL 3D地球

显示效果：

![1571283545662](C:\Users\LINRE\AppData\Roaming\Typora\typora-user-images\1571283545662.png)

代码示例：

```vue
<template>
    <div id="myChart"></div>
</template>

<script>
    import SockJS from 'sockjs-client';
    import Stomp from "stompjs";

    export default {
        name: "Geo",
        mounted() {
            this.initMap()
            this.createSockClient()
        },
        data() {
            return {
                myChart: {},
                mapChart: {},
                num: 0,
                img: require('../static/timg.png'),
                userId: '1',
                sockClient: '',
                stompClient: '',
                backendData: [],
                routeData: [],
                option: {
                    backgroundColor: '#000',
                    globe: {
                        globeRadius: 60,
                        baseTexture: '',
                        silent: true,
                        environment: '',
                        shading: 'realistic',
                        light: {
                            main: {
                                color: '#fff',
                                intensity: 0.8,
                                shadow: false,
                                shadowQuality: 'high',
                                alpha: 55,
                                beta: 10
                            },
                            ambient: {
                                color: '#fff',
                                intensity: 0.5
                            }
                        },
                        postEffect: {
                            enable: false,
                            SSAO: {
                                enable: true,
                                radius: 10
                            }
                        },
                        viewControl: {
                            autoRotate: false,
                            autoRotateDirection: 'cw',
                            alpha: 40,
                            beta: 15,
                            autoRotateAfterStill: 1,
                            animationDurationUpdate: 2000
                        }
                    },
                    series: [
                        {
                            type: 'lines3D',
                            // 尾迹线条
                            effect: {
                                show: true,
                                period: 3,
                                trailWidth: 3,
                                trailLength: 1,
                                trailOpacity: 1,
                                trailColor: 'rgb(231,14,17)'
                            },
                            lineStyle: {
                                opacity: 0
                            },
                            data: []
                        }
                    ]
                },
                mapOption:{
                    backgroundColor: 'rgb(255,255,255)',
                    visualMap: {
                        show: false,
                        min: 0,
                        max: 100000
                    },
                    series: [
                        {
                            type: 'map',
                            map: 'world',
                            left: 0,
                            top: 0,
                            right: 0,
                            bottom: 0,
                            environment: '#000',
                            boundingCoords: [
                                [-180, 90],
                                [180, -90]
                            ],
                            itemStyle: {
                                normal: {
                                    borderWidth: 1,
                                    borderColor: 'rgb(237,177,1)',
                                    areaColor: 'rgb(71,137,191)',
                                    color: 'rgb(255,255,255)'
                                }
                            }
                        }
                    ]
                }
            }
        },
        methods: {
            initMap() {
                this.mapChart = echarts.init(document.createElement('canvas'), null, {
                    width: 3086,
                    height: 3048
                });
                this.myChart = echarts.init(document.getElementById("myChart"));
                this.mapChart.setOption(this.mapOption);
                this.option.globe.baseTexture = this.mapChart
                // 设置背景图片需要用到require访问到静态资源
                this.option.globe.environment = this.img
                this.myChart.setOption(this.option);
            },
            // 从API中获取数据
            createSockClient() {
                this.sockClient = new SockJS("http://ZHAProJB28-W10:8889/searchMap-endPoint");
                this.stompClient = Stomp.over(this.sockClient);
                // 由于内部方法中的this和外面的this不同，所以需要赋值变量拿到外部的this
                let self = this;
                this.stompClient.connect({}, function () {
                    self.stompClient.subscribe("/user/" + self.userId + "/dataServe", function (response) {
                        let result = JSON.parse(response.body);
                        if (result.map) {
                            self.backendData = result.map;
                            self.num = 0;
                        }

                    })
                });
                this.drawLine()
            },
            drawLine() {
                let self = this;
                // 每次输出一条数据，自动定位
                setInterval(function () {
                    let route = [];
                    if (self.num < self.backendData.length) {
                        route.push(self.backendData[self.num].coords)
                        self.num++
                    }
                    self.routeData = route
                    console.log(self.routeData[0][0])
                    setTimeout(function () {
                        self.option.series[0].data = self.routeData
                        self.myChart.setOption(self.option)
                    }, 1000)
                    setTimeout(function () {
                        self.option.globe.viewControl.targetCoord = self.routeData[0][0]
                        self.myChart.setOption(self.option)
                    }, 2000)
                    self.option.globe.viewControl.targetCoord = self.routeData[0][1]
                    self.myChart.setOption(self.option)
                }, 5000)
            },
            disconnectSockClient() {
                if (this.stompClient) {
                    this.stompClient.disconnect();
                    this.sockClient.close();
                }
            },
        },
        beforeDestroy() {
            this.disconnectSockClient();
            clearInterval(this.reconnectTimer);
        }
    }
</script>

<style scoped>
    #myChart {
        width: 100vw;
        height: 100vh;
    }
</style>
```

## 二、点击事件

```vue
<template>
    <div>
        <div id="vessleMap"></div>
        <SearchRank :style="showIf" :routeData="routeData"/>
    </div>
</template>
<script>
data () {
    return {
        showIf: '',
        ...
        // option或者series中的silent属性为false时图形响应触发鼠标事件
    } 
},
methods: {
    ...
    initMap() {
        this.myChart = echarts.init(document.getElementById("vessleMap"));
        this.setHotPortsScatterData();
        this.myChart.setOption(this.option);
        // !!!此处点击事件要拿到data里面的值，需要用箭头函数
        this.myChart.on('click', () => {
            alert(this.showIf);
            this.showIf = "display:none";// 此处动态改变style最好还是利用class来改变
        });//点击事件，此事件还可以用到柱状图等其他地图
    }
},
...
</script>
```

