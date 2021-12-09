<template>
  <div><act-bargraph :graph-data="graphData"></act-bargraph></div>
</template>

<script>
import ActBargraph from "./components/act-bargraph";
export default {
  name: "App",
  components: {
    ActBargraph
  },
  data() {
    //父组件定义全局数据，方便传给需要的子组件们
    //return一定要有，因为data在这里是一个函数方法，不然函数返回的就是undefined
    return {
      //垂直拆分：柱状图组件参数（采用）
        graphData: {
          series: [
            {
              //科目名称
              subjectName: '地理',
              //满分
              fullMark: 100,
              //标记线
              markLine: [
                {
                  //标记线名称（图例名称）
                  markLineName: '考生分数',
                  //标记线分数
                  markLineScore: 98,
                  //类型（0为考生分数；1为其他，如：省平均、市平均）
                  type: 0,
                  //标记线颜色
                  markLineColor: '#B8741A',
                },
              ],
            },
            {
              subjectName: '生物',
              fullMark: 100,
              markLine: [
                {
                  markLineName: '考生分数',
                  markLineScore: 94,
                  type: 0,
                  markLineColor: '#B8741A',
                },
              ],
            },
            {
              subjectName: '历史',
              fullMark: 100,
              markLine: [
                {
                  markLineName: '考生分数',
                  markLineScore: 74,
                  type: 0,
                  markLineColor: '#B8741A',
                },
              ],
            },
          ],
        },
    };
  }
};
</script>
