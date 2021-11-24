<template>
  <div id="bargraph">
    <h1>canvas test</h1>
    <canvas
      id="canvas"
      ref="canvas"
      v-bind:width="canvasWidth"
      v-bind:height="canvasHeight"
    >
    </canvas>
  </div>
</template>

<script>
export default {
  name: "bargraph",
  //props是子组件访问父组件数据的唯一接口
  props: {
    font: {
      type: Number,
      default: "",
    },
    series: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      canvasWidth: 800,
      canvasHeight: 450,
    };
  },
  mounted() {
    const canvas = document.querySelector("#canvas");
    if (canvas) {
      console.log("Variable already exists");
      window.ctx = canvas.getContext("2d");
    }
    this.initCanvas();
  },
  methods: {
    initCanvas() {
      console.log("初始化canvas");
      this.draw();
    },
    //封装画直线的操作以复用
    strokeLine(ctx, moveTo, lineTo, color, width) {
      ctx.beginPath();
      ctx.lineWidth = width;
      ctx.strokeStyle = color;
      ctx.moveTo(...moveTo);
      ctx.lineTo(...lineTo);
      ctx.stroke();
      ctx.closePath();
    },
    draw() {
      //将画布宽高赋值给临时变量 width height 方便操作
      let height = this.canvasHeight;
      let width = this.canvasWidth;
      //矩形的长,用以划线
      let maxScore;
      //100分时的矩形比例系数
      let factor = 0.56;
      //科目信息文本开始绘画的y轴位置 & 矩形粗线结束绘画的位置
      let beginY;
      //各科总分用于绘画柱形
      for (let i = 0; i < this.series[1].data.length; i++) {
        maxScore = this.series[1].data[i];
        //矩形实现方法：粗直线，便于同底不同高的绘画
        //比较每一科的Y轴值，保留最小值（离上边缘最近的那个）
        let temp = height * (1 - factor * (maxScore / 100));
        beginY = beginY < temp ? beginY : temp;
        //画出粗直线作为柱形
        this.strokeLine(
          ctx,
          [width * 0.13 * (i + 1), height * 0.86],
          [width * 0.13 * (i + 1), temp],
          "#F7EBDA",
          width * 0.1
        );
        let recHeight = (height * 0.86)-(temp)
        console.log(recHeight)
        // for(let j = )
        this.strokeLine(
          ctx,
          [width * 0.13 * (i + 1) - width * 0.05, height * 0.86],
          [width * 0.13 * (i + 1) + width * 0.05, height * 0.86],
          "#797979",
          1
        );
      }
      //各科总分用于绘画x轴参数
      //文本的对齐方式的属性(水平属性):
      //ctx.textAlign = "left" || "right" || "center" || "start" || "end";
      //当前文本基线的属性(垂直属性):
      //ctx.textBaseline = "top" || "hanging" || "middle" || "alphabetic" || "ideographic" || "bottom";
      ctx.textAlign = "center";
      ctx.textBaseline = "middle";
      ctx.font = this.font + "px 微软雅黑";
      ctx.fillStyle = "black";
      for (let i = 0; i < this.series[0].data.length; i++) {
        ctx.fillText(
          this.series[0].data[i],
          width * 0.13 * (i + 1),
          beginY - 1.1 * this.font
        );
        // ctx.fillText(
        //   this.series[2].data[i],
        //   width * 0.13 * (i + 1),
        //   beginY - 2.2 * this.font
        // );
      }
      this.strokeLine(
        ctx,
        [width * 0.07, beginY - 0.75 * this.font],
        [
          width * 0.13 * this.series[0].data.length + width * 0.07,
          beginY - 0.75 * this.font,
        ],
        "red",
        1
      );
    },
  },
};
</script>

<style>
canvas {
  display: block;
  margin: 0 auto;
  border: 1px solid;
}
</style>