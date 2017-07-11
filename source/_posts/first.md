---
title: Vue2.0组件——
date: 2017-06-26 09:30:53
tags: Vue2.0 Vux
---
# Vue2.0组件——
---
# 环形进度条(仪表盘)

***
## 实现
1. 最终的仪表盘，见下图水质部分(上面的是vux自带的环形进度条，功能略微简单)：
![仪表盘](http://osprt4338.bkt.clouddn.com/%E9%80%89%E5%8C%BA_051.png)<br>

2. 仪表盘主要有以下几个部分组成：
- 最小值:min(默认为0)
- 警戒最小值：alert-min
- 警戒最大值：alert-max
- 最大值：max(默认为100)

3. **Gauge组建**——调用方式：
```javascript
    <div class="water">
        <p>水质</p>
        <Flexbox wrap="wrap" :gutter="0" justify="center">
          <FlexboxItem :span="1/3" v-for='(water, i) in monitor.water' :key="water.peng_type">
            <Gauge :label="water.title"
                   :value="water.value | convert"
                   :min="water.min_value | convert"
                   :max="water.max_value | convert"
                   :alert-min="water.min_alert_value | convert"
                   :alert-max="water.max_alert_value | convert"
                   :peng-name="water.title"
                   field-name="一号水质监测点">
            </Gauge>
          </FlexboxItem>
        </Flexbox>
      </div>
```

4. **Gauge组建**——实现：<br>
* 用circle绘制值，由styleObject(返回stroke-dasharray和stroke-dashoffset)实现，其半径(radius)和圆心坐标(cx, cy)可以由调用的时候传入，组建内部都有默认值;
* 用path路径绘制lessNormal（小于正常值）断圆弧，属性d决定起始点和具体路径形状，通过describeArc计算d;
* 用path路径绘制normal（正常值）断圆弧，属性d决定起始点和具体路径形状，通过describeArc计算d;;
* 用path路径绘制moreNormal（大于正常值）断圆弧，属性d决定起始点和具体路径形状，通过describeArc计算d;;
```javascript
  <template>
    <div class="donut">
      <svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg" class="donut__svg">
        <circle id="donut-graph" v-bind:style="styleObject" :r="radius" :cy="cy" :cx="cx" stroke-width="5" stroke="url(#purple)"
                stroke-linejoin="round" stroke-linecap="round" stroke-opacity="0.7" fill="none"  transform="rotate(-90 50 50)"/>

        <path id="lessNormal" stroke="#FFD600" stroke-width="1.2" stroke-opacity="0.5" :d="offset.less"></path>
        <path id="normal" stroke="#04BE02" stroke-width="1.2" stroke-opacity="0.5" :d="offset.normal"></path>
        <path id="moreNormal" stroke="#C23531" stroke-width="1.2" stroke-opacity="0.5" :d="offset.more"></path>
        <defs>
          <linearGradient id="purple" x1="0%" y1="0%" x2="100%" y2="0%">
            <stop offset="0%" stop-color="#7a5bcf"/>
            <stop offset="100%" stop-color="#8A6FD5"/>
          </linearGradient>
        </defs>

      </svg>
      <div class="donut_content">
        <span>
          {{label}}
        </span><br>
        <span>
          {{value}}
        </span>
      </div>
    </div>
  </template>
```
```javascript
  methods: {
      computeOffsets: function () {
        this.offset = Object.assign({}, this.offset, {
          min2Normal: {
            offset: this.alertMin / (this.max - this.min),
            color: '#FFD600'
          },
          normal: {
            offset: this.alertMax / (this.max - this.min),
            color: '#04BE02'
          },
          normal2Max: {
            offset: 1,
            color: '#C23531'
          }
        });
        // 绘制小于正常值范围
        let lessEndAngleNum = this.alertMin / (this.max - this.min)
        let lessEndAngle = lessEndAngleNum * 360
        let less = describeArc(this.cx, this.cy, this.radius, 1, 0, lessEndAngle)
        this.$set(this.offset, 'less', less)

        // 绘制正常值范围
        let normalAngleNum = this.alertMax / (this.max - this.min)
        let normalAngle = normalAngleNum * 360
        let normal = describeArc(this.cx, this.cy, this.radius, 1, lessEndAngle, normalAngle)
        this.$set(this.offset, 'normal', normal)

        // 绘制大于正常值范围
        let moreEndAngleNum = this.max / (this.max - this.min)
        let moreEndAngle = moreEndAngleNum * 360
        let more = describeArc(this.cx, this.cy, this.radius, 1, normalAngle, moreEndAngle)
        this.$set(this.offset, 'more', more)
      }
    },
    computed: {
      circumference: function () {
        return 2 * Math.PI * this.radius
      },
      styleObject: function () {
        return {
          "stroke-dasharray": this.circumference,
          "stroke-dashoffset": this.circumference - ((this.value / (this.max - this.min)) * this.circumference)// 不是除以100
        }
      }
    }
```
## 详细代码
详细代码请见我的github，感谢关注:
- [VUE_ANT](https://github.com/zgh2hh/VUE-ANT)<br>
- [Gauge](https://github.com/zgh2hh/VUE-ANT/blob/master/src/components/gauge.vue)<br>
- [调用方式](https://github.com/zgh2hh/VUE-ANT/blob/master/src/features/fields/pages/fieldInfo.vue)

## 涉及技术、参考
- vue2.0
- svg
- [describeArc](http://stackoverflow.com/questions/5736398/how-to-calculate-the-svg-path-for-an-arc-of-a-circle)（用于svg中仪表盘各个点对应的坐标，角度）
- [纯SVG实现进度圈](http://www.w3cplus.com/svg/pure-svg-progress-circles.html)
