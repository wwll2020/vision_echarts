<template>
  <div class="com-container">
    <div class="title" :style="comStyle">
      <span>{{ "｜" + showTitle }}</span>
      <span
        class="iconfont title-icon"
        @click="showChoice = !showChoice"
        :style="comStyle"
        >&#xe6eb;</span
      >
      <div class="select-con" v-show="showChoice" :style="marginStyle">
        <div
          class="select-item"
          v-for="item in selectTypes"
          :key="item.key"
          @click="handleSelect(item.key)"
        >
          {{ item.text }}
        </div>
      </div>
    </div>
    <div class="com-chart" ref="trend_ref"></div>
  </div>
</template>
<script>
import { mapState } from "vuex";
import { getThemeValue } from "@/utils/theme_utils";

export default {
  data() {
    return {
      chartInstance: null,
      allData: null,
      showChoice: false, //是否显示可选项
      choiceType: "map", //显示数据类型
      titleFontSize: 0 //指明标题的字体大小
    };
  },
  // 改造1
  // 在组件创建完成之后 进行回调函数的注册
  created() {
    // this.$socket.registerCallBack("trendData", this.getData);
  },
  mounted() {
    this.initChart();
    this.getData();
    // 发送数据给服务器，告诉服务器 我现在需要数据
    // this.$socket.send({
    //   action: "getData",
    //   socketType: "trendData",
    //   chartName: "trend",
    //   value: ""
    // });
    window.addEventListener("resize", this.screenAdapter);
    this.screenAdapter();
  },
  destroyed() {
    window.removeEventListener("resize", this.screenAdapter);
    // 在组件销毁的时候，进行回调函数的取消
    // this.$socket.unRegisterCallBack("trendData");
  },
  computed: {
    selectTypes() {
      if (!this.allData) {
        return [];
      } else {
        return this.allData.type.filter(item => {
          return item.key !== this.choiceType;
        });
      }
    },
    showTitle() {
      if (!this.allData) {
        return "";
      } else {
        return this.allData[this.choiceType].title;
      }
    },
    comStyle() {
      return {
        fontSize: this.titleFontSize + "px",
        color: getThemeValue(this.theme).titleColor
      };
    },
    marginStyle() {
      return {
        marginLeft: this.titleFontSize + "px"
      };
    },
    ...mapState(["theme"])
  },

  watch: {
    theme() {
      // console.log("主题切换了");
      this.chartInstance.dispose(); //销毁当前图表
      this.initChart();
      this.screenAdapter();
      this.updataChart();
    }
  },
  methods: {
    initChart() {
      this.chartInstance = this.$echarts.init(this.$refs.trend_ref, this.theme);

      const initOption = {
        grid: {
          top: "35%",
          left: "3%",
          right: "4%",
          bottom: "1%",
          containLabel: true
        },
        tooltip: {
          trigger: "axis"
        },
        legend: {
          left: 20,
          top: "15%",
          icon: "circle"
        },
        xAxis: {
          type: "category",
          boundaryGap: false
        },
        yAxis: {
          type: "value"
        }
      };
      this.chartInstance.setOption(initOption);
    },
    async getData() {
      // res 就是服务端发送给客户端的图表的数据
      // getData(res) {
      const { data: res } = await this.$http.get("trend");
      console.log(res);
      this.allData = res;
      console.log(this.allData);
      this.updataChart();
    },
    updataChart() {
      const colorArr1 = [
        "rgba(11,168,44,0.5)",
        "rgba(44,110,255,0.5)",
        "rgba(22,242,217,0.5)",
        "rgba(254,33,30,0.5)",
        "rgba(250,105,0,0.5)"
      ];
      const colorArr2 = [
        "rgba(11,168,44,0)",
        "rgba(44,110,255,0)",
        "rgba(22,242,217,0)",
        "rgba(254,33,30,0)",
        "rgba(250,105,0,0)"
      ];
      //处理数据
      const timeArr = this.allData.common.month; // x轴数据
      const valueArr = this.allData[this.choiceType].data; // y轴数据
      const seriesArr = valueArr.map((item, index) => {
        return {
          name: item.name,
          type: "line",
          data: item.data,
          stack: this.choiceType,
          areaStyle: {
            color: new this.$echarts.graphic.LinearGradient(0, 0, 0, 1, [
              {
                offset: 0,
                color: colorArr1[index]
              },
              {
                offset: 1,
                color: colorArr2[index]
              }
            ])
          }
        };
      });
      // 图例的数据
      const legendArr = valueArr.map(item => {
        return item.name;
      });
      const dataOption = {
        xAxis: {
          data: timeArr
        },
        legend: {
          data: legendArr
        },
        series: seriesArr
      };
      this.chartInstance.setOption(dataOption);
    },
    screenAdapter() {
      this.titleFontSize = (this.$refs.trend_ref.offsetWidth / 100) * 3.6;
      const adapteOption = {
        legend: {
          itemWidth: this.titleFontSize,
          itemHeight: this.titleFontSize,
          itemGap: this.titleFontSize,
          textStyle: {
            fontSize: this.titleFontSize / 2
          }
        }
      };
      this.chartInstance.setOption(adapteOption);
      this.chartInstance.resize();
    },
    handleSelect(currentType) {
      this.choiceType = currentType;
      this.updataChart();
      this.showChoice = false;
    }
  }
};
</script>
<style scoped lang="less">
.title {
  position: absolute;
  left: 20px;
  top: 20px;
  z-index: 10;
  color: #fff;
  .title-icon {
    margin-left: 10px;
    cursor: pointer;
  }
  .select-con {
    background-color: #222733;
  }
}
</style>
