/* eslint-disable prettier/prettier */
<template>
  <!-- <canvas id="canvas" :width="canvas.width" :height="canvas.height"></canvas> -->
  <canvas id="canvas"></canvas>
</template>
<script>
export default {
  name: "ActBargraph",
  //props是子组件访问父组件数据的唯一接口
  props: {
    graphData: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      //canvas对象
      canvas: {},
      //在父组件所占比例
      canvasProportion: 0.55,
      //splitLine分割线设置
      //小分割线多少分画一条,每25分为粗线
      thinLine: 5,
      thinLineColor: "#D7D7D7",
      thinLineWidth: 1,
      borderLineColor: "#797979",
      borderLineWidth: 2,
      //考生分数线透明粗直线颜色
      stuBLineColor: "black",
      //考生分数线透明粗直线宽度
      stuBLineWidth: 25,
      //考生分数线直线宽度
      stuLineWidth: 2,
      //几分标一次分数
      yAxisScore: 50,
      //文本占比设置（小数）
      textFontFactor: 0.032,
      //y轴文本区域占宽比（小数）
      yAxisFactor: 0.075,
      //x轴文本区域占高比（小数）
      xAxisFactor: 0.17,
      //图例区域占高比（小数）
      legendFactor: 0.17,
      //柱形设置
      barColor: "#F7EBDA",
      barWidthFactor: 0.8
    };
  },
  //动态监听所传对象
  watch: {
    graphData: {
      handler(newData) {
        if (this.$parent.$el) {
          this.draw();
        }
      },
      immediate: true,
      deep: true
    }
  },
  //实例创建后执行
  created() {},
  //挂载完成后调用
  mounted() {
    this.canvasResize();
    this.draw();
    window.onresize = () => {
      //调用methods中的事件
      this.canvasResize();
      let that = this;
      that.draw();
      //设置延时，不然可能没内容
      setTimeout(function() {
        that.draw();
      }, 100);
    };
  },
  methods: {
    canvasResize() {
      this.canvas = document.getElementById("canvas");
      let elObj = this.$parent.$el;
      this.canvas.width = elObj.clientWidth * this.canvasProportion;
      this.canvas.height = this.canvas.width * 0.5;
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
    //封装设置文本样式操作以复用
    //后续有需求可提供自定义
    textType(ctx, textAlign, textBaseline, font, fillStyle) {
      //文本的对齐方式的属性(水平属性):
      //ctx.textAlign = "left" || "right" || "center" || "start" || "end";
      //当前文本基线的属性(垂直属性):
      //ctx.textBaseline = "top" || "hanging" || "middle" || "alphabetic" || "ideographic" || "bottom";
      ctx.textAlign = textAlign;
      ctx.textBaseline = textBaseline;
      ctx.font = font;
      ctx.fillStyle = fillStyle;
    },
    //画图
    draw() {
      var ctx = canvas.getContext("2d");
      ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
      //绘制柱形以及柱形对应学科
      //柱形占高比
      var barFactor = 1 - (this.legendFactor + this.xAxisFactor);
      //标准柱形高度
      var stdBarHeight = this.canvas.height * barFactor;
      //柱形占比最大区间
      var maxBarWidth =
        (this.canvas.width * (1 - this.yAxisFactor * 2)) /
        this.graphData.series.length;
      //标准柱形宽度
      var stdBarWidth = maxBarWidth * this.barWidthFactor;
      //最大满分值
      var maxFullMark;
      var underlineStartX;
      var underlineEndX;
      //求满分最大值,标准柱形占高就是满分最大值柱形的高
      this.graphData.series.forEach(item => {
        maxFullMark = maxFullMark > item.fullMark ? maxFullMark : item.fullMark;
      });
      //矩形实现方法：粗直线，便于同底不同高的绘画
      for (let i = 0; i < this.graphData.series.length; i++) {
        let fullMark = this.graphData.series[i].fullMark;
        //当前柱形高度占比，用该科满分/最大满分
        let currentFactor = fullMark / maxFullMark;
        //计算起点和终点
        let startY = this.canvas.height * (1 - this.legendFactor);
        let endY = startY - stdBarHeight * currentFactor;
        //粗直线x轴位置
        let locationX =
          this.canvas.width * this.yAxisFactor +
          i * maxBarWidth +
          maxBarWidth / 2;
        //画出粗直线作为柱形
        this.strokeLine(
          ctx,
          [locationX, startY],
          [locationX, endY],
          this.barColor,
          stdBarWidth
        );
        //设置字体样式
        let font = this.canvas.width * this.textFontFactor + "px 微软雅黑";
        this.textType(ctx, "center", "middle", font, "black");
        //写x轴科目信息
        let subjectName = this.graphData.series[i].subjectName;
        //微调科目名位置
        let locationText =
          this.canvas.height * this.textFontFactor +
          this.canvas.height * 0.17 * 0.3;
        ctx.fillText(
          subjectName,
          locationX,
          startY - stdBarHeight - locationText
        );
        //绘制柱形刻度线（一般为细线，整除5或到顶端时为粗线）
        //刻度线总数，若不整除则记录topScale为1，画到顶端时标粗线
        let scale = this.thinLine / maxFullMark;
        let scaleNum = fullMark / this.thinLine;
        let topScale = 0;
        let borderLine = this.yAxisScore / this.thinLine;
        let startX = locationX - stdBarWidth * 0.5;
        let endX = locationX + stdBarWidth * 0.5;
        //y轴标注刻度
        let yText = 0;
        if (fullMark % this.borderLine != 0) {
          topScale = 1;
        }
        for (let j = 0; j <= scaleNum; j++) {
          //标线Y轴位置
          let locationY = startY - stdBarHeight * scale * j;
          //粗线
          if (j % borderLine == 0) {
            this.strokeLine(
              ctx,
              [startX, locationY],
              [endX, locationY],
              this.borderLineColor,
              this.borderLineWidth
            );
            if (i == 0) {
              //标刻度
              ctx.fillText(yText, startX - locationText, locationY);
              yText += this.yAxisScore;
            }
          } else {
            this.strokeLine(
              ctx,
              [startX, locationY],
              [endX, locationY],
              this.thinLineColor,
              this.thinLineWidth
            );
          }
        }
        //如果topScale不为0，给该特殊矩形封顶
        if (topScale != 0) {
          this.strokeLine(
            ctx,
            [startX, endY],
            [endX, endY],
            this.borderLineColor,
            this.borderLineWidth
          );
        }
        //绘制图例、考生分数线
        //图例线开始结束位置
        let legendStartX;
        let legendEndX;
        let legendLineWidth = stdBarWidth * 0.7;
        for (let k = 0; k < this.graphData.series[i].markLine.length; k++) {
          let markLine = this.graphData.series[i].markLine[k];
          //可以在第一个矩形那里先把图例画了
          if (i == 0) {
            //图例开始位置
            let legendY = this.canvas.height * (1 - this.legendFactor * 0.5);
            let legendNameWidth =
              markLine.markLineName.length *
              this.canvas.width *
              this.textFontFactor;
            if (k == 0) {
              legendStartX = locationX + legendNameWidth * 0.25;
              legendEndX = legendStartX + legendLineWidth;
            } else {
              legendStartX = legendStartX + legendLineWidth * 2;
              legendEndX = legendStartX + legendLineWidth;
            }
            if (markLine.type == 0) {
              //图例名字
              ctx.fillText(
                markLine.markLineName,
                legendStartX - legendNameWidth * 0.55,
                legendY
              );
              //考生分数图例
              ctx.globalAlpha = 0.1;
              this.strokeLine(
                ctx,
                [legendStartX, legendY],
                [legendEndX, legendY],
                this.stuBLineColor,
                this.stuBLineWidth
              );
              ctx.globalAlpha = 1;
              this.strokeLine(
                ctx,
                [legendStartX, legendY],
                [legendEndX, legendY],
                markLine.markLineColor,
                this.stuLineWidth
              );
            } else {
              //图例名字
              ctx.fillText(
                markLine.markLineName,
                legendStartX - legendNameWidth * 0.55,
                legendY
              );
              this.strokeLine(
                ctx,
                [legendStartX, legendY],
                [legendEndX, legendY],
                markLine.markLineColor,
                this.stuLineWidth
              );
            }
          }
          let markLineY =
            stdBarHeight *
            currentFactor *
            (this.graphData.series[i].markLine[k].markLineScore / fullMark);
          //考生分数线
          if (markLine.type == 0) {
            ctx.globalAlpha = 0.1;
            this.strokeLine(
              ctx,
              [startX, startY - markLineY],
              [endX, startY - markLineY],
              this.stuBLineColor,
              this.stuBLineWidth
            );
            ctx.globalAlpha = 1;
            this.strokeLine(
              ctx,
              [startX, startY - markLineY],
              [endX, startY - markLineY],
              markLine.markLineColor,
              this.stuLineWidth
            );
          } else {
            //其他分数线
            this.strokeLine(
              ctx,
              [startX, startY - markLineY],
              [endX, startY - markLineY],
              markLine.markLineColor,
              this.stuLineWidth
            );
          }
        }
        //如果是第一个矩形或最后一个矩形，画竖线将图封起来
        if (i == 0) {
          this.strokeLine(
            ctx,
            [startX, startY],
            [startX, endY],
            this.borderLineColor,
            this.borderLineWidth
          );
          //记录将要绘制的红线的x轴开始坐标
          underlineStartX = startX;
        }
        if (i == this.graphData.series.length - 1) {
          this.strokeLine(
            ctx,
            [endX, startY],
            [endX, endY],
            this.borderLineColor,
            this.borderLineWidth
          );
          //记录将要绘制的红线的x轴开始坐标
          underlineEndX = endX;
        }
      }
      //绘制科目下划红线
      let underlineY =
        this.canvas.height * this.textFontFactor +
        this.canvas.height * 0.17 * 0.5;
      this.strokeLine(
        ctx,
        [underlineStartX, underlineY],
        [underlineEndX, underlineY],
        "red",
        this.borderLineWidth
      );
      //console.log('canvas', this.canvas.toDataURL())
    }
  }
};
</script>
<style>
canvas {
  border: 1px solid;
}
</style>
