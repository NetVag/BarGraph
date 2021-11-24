<template>
  <bargraph :series="series" :font="font"></bargraph>
</template>

<script>
import bargraph from "./components/bargraph";
export default {
  name: "App",
  components: {
    bargraph,
  },
  data() {
    //父组件定义全局数据，方便传给需要的子组件们
    //return一定要有，因为data在这里是一个函数方法，不然函数返回的就是undefined
    return {
      font: 20,
      series: [
        {
          //学科名
          name: "subject",
          data: ["平均分", "物理", "历史", "化学", "生物", "政治", "地理"],
        },
        {
          //总分
          name: "maxScore",
          data: [150, 120, 100, 100, 100, 100, 100],
        },
        {
          //学生分数
          name: "stuScore",
          data: [92.5, 82, 93, 98, 93, 94, 95],
        },
        {
          //省平均
          name: "provAvg",
          data: [61.12, 66.5, 56.14, 62.51, 44.1, 68.32, 69.2],
        },
        {
          //市平均
          name: "cityAvg",
          data: [74.52, 76.5, 66.25, 54.1, 56.2, 75.32, 75.32],
        },
      ],
    };
  },
};
</script>
