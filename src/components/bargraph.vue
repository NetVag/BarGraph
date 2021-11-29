<template>
  <div id="bargraph">
    <canvas
      id="canvas"
      ref="canvas"
      :width="canvasWidth"
      :height="canvasHeight"
    ></canvas>
  </div>
</template>

<script>
  export default {
    name: 'bargraph',
    //props是子组件访问父组件数据的唯一接口
    props: {
      font: {
        type: Number,
        default: 12,
      },
      series: {
        type: Array,
        required: true,
      },
    },
    data() {
      return {
        canvasWidth: 550,
        canvasHeight: 300,
      }
    },
    watch: {
      series: {
        handler(newData) {
          this.setOption()
        },
        immediate: true,
        deep: true,
      },
    },
    mounted() {
      const canvas = document.querySelector('#canvas')
      if (canvas) {
        window.ctx = canvas.getContext('2d')
      }
      this.initCanvas()
    },
    methods: {
      setOption() {
        const canvas = document.querySelector('#canvas')
        if (canvas) {
          window.ctx = canvas.getContext('2d')
        }
        this.initCanvas()
      },
      initCanvas() {
        this.draw()
      },
      //封装画直线的操作以复用
      strokeLine(ctx, moveTo, lineTo, color, width) {
        ctx.beginPath()
        ctx.lineWidth = width
        ctx.strokeStyle = color
        ctx.moveTo(...moveTo)
        ctx.lineTo(...lineTo)
        ctx.stroke()
        ctx.closePath()
      },
      draw() {
        //文本的对齐方式的属性(水平属性):
        //ctx.textAlign = "left" || "right" || "center" || "start" || "end";
        //当前文本基线的属性(垂直属性):
        //ctx.textBaseline = "top" || "hanging" || "middle" || "alphabetic" || "ideographic" || "bottom";
        //后面需封装复用
        ctx.textAlign = 'center'
        ctx.textBaseline = 'middle'
        ctx.font = this.font + 'px 微软雅黑'
        ctx.fillStyle = 'black'
        //将画布宽高赋值给临时变量 width height 方便操作
        let height = this.canvasHeight
        let width = this.canvasWidth
        //各科满分
        let maxScore
        //满分中的最大值，作为y轴顶端数据
        let Max
        //100分时的矩形比例系数
        let factor = 0.56
        //科目信息文本开始绘画的y轴位置 & 矩形粗线结束绘画的位置
        let beginY
        //100分时的矩形高度
        let stdRecHeight = height * 0.86 - height * (1 - factor)
        //记录各矩形高度
        let recHeight
        //记录各种类型的分数的名字，如：学生分数、省平均分
        let enumName
        ////记录各种类型的分数的值
        let enumValue
        //各科总分用于绘画柱形
        for (let i = 0; i < this.series[1].data.length; i++) {
          maxScore = this.series[1].data[i]
          //矩形实现方法：粗直线，便于同底不同高的绘画
          //暂时记录该粗直线终点Y轴值
          let temp = height * 0.86 - stdRecHeight * (maxScore / 100)
          //console.log(height * 0.86 - temp)
          //比较每一科的Y轴值，保留最小值（离上边缘最近的那个）
          beginY = beginY < temp ? beginY : temp
          //记录最大满分
          Max = Max > maxScore ? Max : maxScore
          //画出粗直线作为柱形
          this.strokeLine(
            ctx,
            [width * 0.13 * (i + 1), height * 0.86],
            [width * 0.13 * (i + 1), temp],
            '#F7EBDA',
            width * 0.1
          )
          let scaleNum = maxScore / 5
          let topScale = 0
          if (maxScore % 25 != 0) {
            topScale = 1
          }
          //console.log(scaleNum);
          for (let j = 0; j <= scaleNum; j++) {
            //console.log(j * stdRecHeight * 0.05)
            if (j % 5 == 0) {
              //console.log(j + " " + height * 0.86 - j * stdRecHeight * 0.05);
              this.strokeLine(
                ctx,
                [
                  width * 0.13 * (i + 1) - width * 0.05,
                  height * 0.86 - j * stdRecHeight * 0.05,
                ],
                [
                  width * 0.13 * (i + 1) + width * 0.05,
                  height * 0.86 - j * stdRecHeight * 0.05,
                ],
                '#797979',
                2
              )
            } else {
              this.strokeLine(
                ctx,
                [
                  width * 0.13 * (i + 1) - width * 0.05,
                  height * 0.86 - j * stdRecHeight * 0.05,
                ],
                [
                  width * 0.13 * (i + 1) + width * 0.05,
                  height * 0.86 - j * stdRecHeight * 0.05,
                ],
                '#D7D7D7',
                1
              )
            }
          }
          //console.log(topScale + " " + temp);
          if (topScale != 0) {
            this.strokeLine(
              ctx,
              [width * 0.13 * (i + 1) - width * 0.05, temp],
              [width * 0.13 * (i + 1) + width * 0.05, temp],
              '#797979',
              2
            )
          }
          //取每科的各种值，2号下标默认存的都是学生分数
          for (let k = 2; k < this.series.length; k++) {
            enumName = this.series[k].name
            enumValue = this.series[k].data[i]
            //画分数线
            this.strokeLine(
              ctx,
              [
                width * 0.13 * (i + 1) - width * 0.05,
                temp + (height * 0.86 - temp) * (1 - enumValue / 100),
              ],
              [
                width * 0.13 * (i + 1) + width * 0.05,
                temp + (height * 0.86 - temp) * (1 - enumValue / 100),
              ],
              this.series[k].color,
              2
            )
            //图例
            //ctx.fillText(enumName, width * 0.13 * (i + 1), height * 0.95);
          }
        }
        //图例
        for (let i = 2; i < this.series.length; i++) {
          enumName = this.series[i].name
          ctx.fillText(
            enumName,
            ((width * 0.8) / (this.series.length - 2)) * (i - 1) - width * 0.1,
            height * 0.95
          )
          this.strokeLine(
            ctx,
            [
              ((width * 0.8) / (this.series.length - 2)) * (i - 1) -
                width * 0.05,
              height * 0.95,
            ],
            [
              ((width * 0.8) / (this.series.length - 2)) * (i - 1) +
                width * 0.05,
              height * 0.95,
            ],
            this.series[i].color,
            2
          )
        }
        //各科总分用于绘画x轴参数
        for (let i = 0; i < this.series[0].data.length; i++) {
          ctx.fillText(
            this.series[0].data[i],
            width * 0.13 * (i + 1),
            beginY - 1.1 * this.font
          )
          // ctx.fillText(
          //   this.series[2].data[i],
          //   width * 0.13 * (i + 1),
          //   beginY - 2.2 * this.font
          // );
        }
        //绘制y轴信息
        for (let i = 0, j = 0; i < Max; i += 25) {
          ctx.fillText(i, width * 0.04, height * 0.86 - (stdRecHeight / 4) * j)
          j++
        }
        ctx.fillText(Max, width * 0.04, beginY)
        //绘制红线
        this.strokeLine(
          ctx,
          [width * 0.07, beginY - 0.75 * this.font],
          [
            width * 0.13 * this.series[0].data.length + width * 0.07,
            beginY - 0.75 * this.font,
          ],
          'red',
          1
        )
      },
    },
  }
</script>

<style>
  canvas {
    display: block;
    margin: 0 auto;
    border: 1px solid;
  }
</style>
