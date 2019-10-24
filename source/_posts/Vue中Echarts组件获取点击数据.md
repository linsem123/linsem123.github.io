---
title: Vue中Echarts组件获取点击数据
date: 2019-10-21 11:02:49
categories:
- 编程
tags:
- Echarts
---

# 《Vue中Echarts组件获取点击数据》

### 前言
大多数数据展示都会使用Echarts图表进行展示，但是如果遇到需要点击某一列的数据出发一个回调函数进行操作，列如放大点击列的数据，或拿到点击列的数据后执行某些操作。

### 一、实现过程
1. 配置图表属性 **tooltip**
   **trigger**,**axisPointer**,**triggerOn**这三个属性必须配置
   ```
   tooltip: {
          trigger: 'axis',
          showContent: false, //是否显示提示框浮层
          axisPointer: {
            type: 'line',
            lineStyle: {
              color: "#d19a37"
            },
            snap: true,
          },
          triggerOn: "mousemove|click",
   }
   ```
2. 点击回调接收
   - 引入组件 [vue-echarts-v3](https://github.com/xlsdg/vue-echarts-v3)
    ```
    import echarts from "vue-echarts-v3/src/lite.js";
    import 'echarts/lib/chart/line'; //折线图
    import 'echarts/lib/component/tooltip'; //引入提示框组件

    export default{
      components: {
        vChart: echarts
      }
    }
    ```
  - 使用组件vChart，注册回调 onReady
    ```
    <vChart
      :option="chart_option"
      ref="chart"
      @ready="onReady"
    />
    ```
  - onReady回调函数监听图表
    ```
    methods: {
      onReady(instance, echarts) {
        let me = this;
        let getData = params => {
          const pointInPixel = [params.offsetX, params.offsetY];
          if (instance.containPixel("grid", pointInPixel)) {
            let xIndex = instance.convertFromPixel({ seriesIndex: 0 }, [
              params.offsetX,
              params.offsetY
            ])[0];
            // console.log(xIndex)
            let my_data_obj = me.chart_option;
            let obj = {
              price: my_data_obj.series[0].data[xIndex],
              ave: my_data_obj.series[1].data[xIndex],
              date: my_data_obj.xAxis.data[xIndex]
            };
            me.chart_1_data_obj = obj
          }
        }
        instance.getZr().on("click", getData);
        instance.getZr().on("mousemove", getData);
        instance.getZr().on("mousedown", params => {
          me.is_touch_chart_1 = true;
          me.$refs.chart.mergeOption({
            tooltip: {
              show: true
            }
          });
        });
        instance.getZr().on("mouseup", params => {
          me.is_touch_chart_1 = false;
          me.$refs.chart.mergeOption({
            tooltip: {
              show: false
            }
          });
        });
      }
    }
    ```
### 二、总结
- onReady回调函数拿到当前点击的echarts实例对象，以便对整个canvas区域点击监听。
- getData函数接收绑定的原生事件返回的结果（当前用户点击的区域坐标）
- 通过echarts API convertFromPixel方法拿到用户点击的数据下标，通过下标获得用户点击的数据

### 参考
[完美解决echarts的柱状图和折线图的点击非图表图形元素不会触发事件](https://blog.csdn.net/smk108/article/details/78482154)
[echarts监听点击（空白处）事件，echarts监听滑动事件](https://blog.csdn.net/qq_24065713/article/details/84771570)