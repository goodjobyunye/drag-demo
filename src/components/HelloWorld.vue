<template>
  <div>
    <div class="imgBox">
      <img id="drag1" src="../assets/bar.jpg" width="100" height="100" draggable="true" @dragstart="drag($event)" />
    </div>
    <hr />
    <div class="dragSpace" @drop="drop($event)" @dragover="allowDrop($event)">

    </div>
    <hr />
    
  </div>
</template>
<script>
  // 引入基本模板
  let echarts = require('echarts/lib/echarts');
  // 引入柱状图组件
  require('echarts/lib/chart/bar');
  // 引入提示框和title组件
  require('echarts/lib/component/tooltip');
  require('echarts/lib/component/title');
  const $ = require('jquery');
  export default {
    name: 'drag-echarts',
    data: function() {
      return {
        params: [],
        boxIndex: -1,
      };
    },
    methods: {
      getCss: function (o, key) {
        return o.currentStyle ? o.currentStyle[key] : document.defaultView.getComputedStyle(o, false)[key];
      },

      assignParams: function(target, boxIndex) {
        if (this.getCss(target, "left") !== "auto") {
          this.params[boxIndex].left = this.getCss(target, "left");
        }
        if (this.getCss(target, "top") !== "auto") {
          this.params[boxIndex].top = this.getCss(target, "top");
        }
      },

      startDrag: function (bar, target) {
        const _this = this;
        const boxIndex = _this.boxIndex;
        this.params.push({
          left: 0,
          top: 0,
          currentX: 0,
          currentY: 0,
          flag: false
        });
        this.assignParams(target, boxIndex);

        bar.onmousedown = function (event) {
          _this.params[boxIndex].flag = true;
          if (!event) {
            event = window.event;

            bar.onselectstart = function () {
              return false;
            }
          }
          _this.params[boxIndex].currentX = event.clientX;
          _this.params[boxIndex].currentY = event.clientY;
        };

        document.onmouseup = function () {
          const boxIndex = _this.params.findIndex((item) => item.flag);
          _this.params[boxIndex].flag = false;
          _this.assignParams(document.querySelector(`.box${boxIndex}`), boxIndex);
        };

        document.onmousemove = function (event) {
          const e = event ? event : window.event;
          if (_this.params.some((item) => item.flag)) {
            const boxIndex = _this.params.findIndex((item) => item.flag);
            const target = document.querySelector(`.box${boxIndex}`);
            const nowX = e.clientX, nowY = e.clientY;
            const disX = nowX - _this.params[boxIndex].currentX, disY = nowY - _this.params[boxIndex].currentY;
            target.style.left = parseInt(_this.params[boxIndex].left) + disX + "px";
            target.style.top = parseInt(_this.params[boxIndex].top) + disY + "px";
            if (event.preventDefault) {
              event.preventDefault();
            }
            return false;
          }
        }
      },

      allowDrop: (ev)=>{
       ev.preventDefault();
      },

      drag: (ev)=> {
        ev.dataTransfer.setData("Text",ev.target.id);
      },

      drop: function(ev) {
        this.boxIndex++;
        ev.preventDefault();
        const oBox = $(`<div class="box box${this.boxIndex}" style="left: 119px; top: 172px;">
          <div class="main">
            <div class="bar">拖拽</div>
            <div class="content">
            </div>
          </div>
        </div>`);
        oBox.css('left', ev.clientX + 'px');
        oBox.css('top', ev.clientY + 'px');
        $(ev.target).append(oBox);
        this.creatBar(document.querySelector(`.box${this.boxIndex} .content`));
        this.startDrag(document.querySelector(`.box${this.boxIndex} .bar`), document.querySelector(`.box${this.boxIndex}`));
      },

      creatBar: ($box) => {
        const option = {
          color: ['#3398DB'],
          tooltip: {
              trigger: 'axis',
              axisPointer: {            // 坐标轴指示器，坐标轴触发有效
                  type: 'shadow'        // 默认为直线，可选为：'line' | 'shadow'
              }
          },
          grid: {
              left: '3%',
              right: '4%',
              bottom: '3%',
              containLabel: true
          },
          xAxis: [
              {
                  type: 'category',
                  data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'],
                  axisTick: {
                      alignWithLabel: true
                  }
              }
          ],
          yAxis: [
              {
                  type: 'value'
              }
          ],
          series: [
              {
                  name: '直接访问',
                  type: 'bar',
                  barWidth: '60%',
                  data: [10, 52, 200, 334, 390, 330, 220]
              }
          ]
        };

        let myChart = echarts.init($box);
        // 绘制图表
        myChart.setOption(option);
      },
    },

    mounted: function() {
      
      
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
* {
  margin: 0;
  padding: 0;
}

.box {
  position: absolute;
  left: 100px;
  top: 100px;
  padding: 5px;
  background: #f0f3f9;
  font-size: 12px;
  -moz-box-shadow: 2px 2px 4px #666666;
  -webkit-box-shadow: 2px 2px 4px #666666;
}

.main {
  border: 1px solid #a0b3d6;
  background: white;
}

.bar {
  line-height: 24px;
  background: #beceeb;
  border-bottom: 1px solid #a0b3d6;
  padding-left: 5px;
  cursor: move;
}

.content {
  width: 420px;
  height: 250px;
  padding: 10px 5px;
}

.imgBox {
  height: 100px;
}

.dragSpace {
  height: 1000px;
}
</style>
