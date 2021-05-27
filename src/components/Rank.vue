<template>
  <div class="com-container">
    <div class="com-chart" ref="rank_ref"></div>
  </div>
</template>
<script>
import { mapState } from "vuex";

export default {
  data() {
    return {
      chartInstance: null,
      allData: null,
      startValue: 0, //区域缩放的起点值
      endValue: 9, // 区域缩放的终点值
      timerId: null
    };
  },
  mounted() {
    this.initChart();
    this.getData();
    window.addEventListener("rseize", this.screenAdapter);
    this.screenAdapter();
  },
  destroyed() {
    window.removeEventListener("resize", this.screenAdapter);
    clearInterval(this.timerId);
  },
  methods: {
    initChart() {
      this.chartInstance = this.$echarts.init(this.$refs.rank_ref, this.theme);
      const initOption = {
        title: {
          text: "｜地区销售排行",
          top: 20,
          left: 20
        },
        grid: {
          top: "40%",
          left: "5%",
          right: "5%",
          bottom: "5%",
          containLable: true
        },
        tooltip: {
          show: true
        },
        xAxis: {
          type: "category"
        },
        yAxis: {
          type: "value"
        },
        series: [
          {
            type: "bar"
          }
        ]
      };
      this.chartInstance.setOption(initOption);
      this.chartInstance.on("mouseover", () => {
        clearInterval(this.timerId);
      });
      this.chartInstance.on("mouseout", () => {
        this.startInterval();
      });
    },
    async getData() {
      const { data: res } = await this.$http.get("rank");
      this.allData = res;
      // 从大到小排序
      this.allData.sort((a, b) => {
        return b.value - a.value;
      });
      console.log(this.allData);
      this.updataChart();
      this.startInterval();
    },
    updataChart() {
      const colorArr = [
        ["#0ba82c", "#4ff778"],
        ["#2e72bf", "#23e5e5"],
        ["#5052ee", "#ab6ee5"]
      ];
      // 所有省份形成的数组
      const provinceArr = this.allData.map(item => {
        return item.name;
      });
      // 所有省份对应的销售金额
      const valueArr = this.allData.map(item => {
        return item.value;
      });
      const dataOption = {
        xAxis: {
          data: provinceArr
        },
        dataZoom: {
          show: false,
          startValue: this.startValue,
          endValue: this.endValue
        },
        series: [
          {
            data: valueArr,
            itemStyle: {
              color: arg => {
                let targetcolorArr = null;
                if (arg.value > 300) {
                  targetcolorArr = colorArr[0];
                } else if (arg.value > 200) {
                  targetcolorArr = colorArr[1];
                } else {
                  targetcolorArr = colorArr[2];
                }
                return new this.$echarts.graphic.LinearGradient(0, 0, 0, 1, [
                  {
                    offset: 0,
                    color: targetcolorArr[0]
                  },
                  {
                    offset: 1,
                    color: targetcolorArr[1]
                  }
                ]);
              }
            }
          }
        ]
      };
      this.chartInstance.setOption(dataOption);
    },
    screenAdapter() {
      const titleFontSize = (this.$refs.rank_ref.offsetWidth / 100) * 3.6;
      const adapterOption = {
        title: {
          textStyle: {
            fontSize: titleFontSize
          }
        },
        series: [
          {
            barWidth: titleFontSize,
            itemStyle: {
              barBorderRadius: [titleFontSize / 2, titleFontSize / 2, 0, 0]
            }
          }
        ]
      };
      this.chartInstance.setOption(adapterOption);
      this.chartInstance.resize();
    },
    startInterval() {
      if (this.timerId) {
        clearInterval(this.timerId);
      }
      this.timerId = setInterval(() => {
        this.startValue++;
        this.endValue++;
        if (this.endValue > this.allData.length - 1) {
          this.startValue = 0;
          this.endValue = 9;
        }
        this.updataChart();
      }, 2000);
    }
  },
  computed: {
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
  }
};
</script>
