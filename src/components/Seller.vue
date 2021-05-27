<!-- 商家销量统计的横向柱状图 -->
<template>
  <div class="com-container">
    <div class="com-chart" ref="seller_ref"></div>
  </div>
</template>
<script>
import { mapState } from "vuex";
export default {
  data() {
    return {
      chartInstance: null,
      allData: null, //服务器返回的数据
      currentPage: 1, // 当前显示的页数
      totalPage: 0, //一共有多少页
      timerId: null //定时器的标识
    };
  },
  mounted() {
    this.initChart();
    this.getData();
    window.addEventListener("resize", this.screenAdapter);
    // 界面加载完成，主动进行屏幕的适配
    this.screenAdapter();
  },
  destroyed() {
    clearInterval(this.timerId);
    // 在组件销毁的时候，取消屏幕适配的监听
    window.removeEventListener("resize", this.screenAdapter);
  },
  methods: {
    // 1.初始化echartInstance对象
    initChart() {
      this.chartInstance = this.$echarts.init(
        this.$refs.seller_ref,
        this.theme
      );
      // 对图表初始化配置的控制
      const initOption = {
        title: {
          text: "｜商家销售统计",
          textStyle: {
            // fontSize: 66
          },
          left: 20,
          top: 20
        },
        grid: {
          top: "20%",
          left: "3%",
          right: "6%",
          bottom: "3%",
          containLabel: true //距离是包含坐标轴上的文字
        },
        xAxis: {
          type: "value"
        },
        yAxis: {
          type: "category"
        },
        tooltip: {
          trigger: "axis",
          axisPointer: {
            type: "line",
            z: 0,
            lineStyle: {
              // width: 66,
              color: "#2d3443"
            }
          }
        },
        series: [
          {
            type: "bar",
            // barWidth: 66,
            label: {
              show: true,
              position: "right",
              textStyle: {
                color: "#fff"
              }
            },
            itemStyle: {
              // barBorderRadius: [0, 33, 33, 0],
              // 指明颜色渐变的方向
              // 指明不同百分比之下颜色的值
              color: new this.$echarts.graphic.LinearGradient(0, 0, 1, 0, [
                {
                  offset: 0,
                  color: "#5052ee"
                },
                {
                  offset: 1,
                  color: "#ab6ee5"
                }
              ])
            }
          }
        ]
      };
      this.chartInstance.setOption(initOption);
      // 对图表对象进行鼠标事件的监听
      this.chartInstance.on("mouseover", () => {
        clearInterval(this.timerId);
      });
      this.chartInstance.on("mouseout", () => {
        this.startInterval();
      });
    },
    // 2.获取服务器数据
    async getData() {
      // http://127.0.0.1:8888/api/seller
      const { data: res } = await this.$http.get("seller");
      console.log(res);
      this.allData = res;
      // 对数据排序
      this.allData.sort((a, b) => {
        return a.value - b.value; //从小到大
      });
      // 每5个元素显示一页
      this.totalPage =
        this.allData.length % 5 === 0
          ? this.allData.length / 5
          : this.allData.length / 5 + 1;
      this.updataChart();
      // 启动定时器
      this.startInterval();
    },
    // 3.更新图表
    updataChart() {
      const start = (this.currentPage - 1) * 5;
      const end = this.currentPage * 5;
      const showData = this.allData.slice(start, end);
      const sellerNames = showData.map(item => {
        return item.name;
      });
      const sellerValues = showData.map(item => {
        return item.value;
      });
      // const option = {
      // title: {
      //   text: "｜商家销售统计",
      //   textStyle: {
      //     fontSize: 66
      //   },
      //   left: 20,
      //   top: 20
      // },
      // grid: {
      //   top: "20%",
      //   left: "3%",
      //   right: "6%",
      //   bottom: "3%",
      //   containLabel: true //距离是包含坐标轴上的文字
      // },
      // xAxis: {
      //   type: "value"
      // },
      // yAxis: {
      //   type: "category",
      //   data: sellerNames
      // },
      // tooltip: {
      //   trigger: "axis",
      //   axisPointer: {
      //     type: "line",
      //     z: 0,
      //     lineStyle: {
      //       width: 66,
      //       color: "#2d3443"
      //     }
      //   }
      // },
      // series: [
      //   {
      //     data: sellerValues,
      //     type: "bar",
      //     barWidth: 66,
      //     label: {
      //       show: true,
      //       position: "right",
      //       textStyle: {
      //         color: "#fff"
      //       }
      //     },
      //     itemStyle: {
      //       barBorderRadius: [0, 33, 33, 0],
      //       // 指明颜色渐变的方向
      //       // 指明不同百分比之下颜色的值
      //       color: new this.$echarts.graphic.LinearGradient(0, 0, 1, 0, [
      //         {
      //           offset: 0,
      //           color: "#5052ee"
      //         },
      //         {
      //           offset: 1,
      //           color: "#ab6ee5"
      //         }
      //       ])
      //     }
      //   }
      // ]
      // };
      // this.chartInstance.setOption(option);
      const dataOption = {
        yAxis: {
          data: sellerNames
        },
        series: [
          {
            data: sellerValues
          }
        ]
      };
      this.chartInstance.setOption(dataOption);
    },
    startInterval() {
      if (this.timerId) {
        clearInterval(this.timerId);
      }
      this.timerId = setInterval(() => {
        this.currentPage++;
        if (this.currentPage > this.totalPage) {
          this.currentPage = 1;
        }
        this.updataChart();
      }, 3000);
    },
    // 当浏览器的大小发生变化的时候，会调用方法来完成屏幕的适配
    screenAdapter() {
      // console.log(this.$refs.seller_ref.offsetWidth);
      const titleFontSize = (this.$refs.seller_ref.offsetWidth / 100) * 3.6;
      // 和分辨率大小相关的配置项
      const adapterOption = {
        title: {
          text: "｜商家销售统计",
          textStyle: {
            fontSize: titleFontSize
          }
        },

        tooltip: {
          axisPointer: {
            lineStyle: {
              width: titleFontSize
            }
          }
        },
        series: [
          {
            barWidth: titleFontSize,

            itemStyle: {
              barBorderRadius: [0, titleFontSize / 2, titleFontSize / 2, 0]
            }
          }
        ]
      };
      this.chartInstance.setOption(adapterOption);
      // 手动的调用图表对象的 resize 才能产生效果
      this.chartInstance.resize();
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
<style lang="less" scoped></style>
