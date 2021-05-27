<template>
  <div class="com-container" @dblclick="revertMap">
    <div class="com-chart" ref="map_ref"></div>
  </div>
</template>
<script>
import axios from "axios";
import { getProvinceMapInfo } from "@/utils/map_utils";
import { mapState } from "vuex";

export default {
  data() {
    return {
      chartInstance: null,
      allData: null,
      mapData: {} // 所获取的省份的地图矢量数据
    };
  },
  mounted() {
    this.initChart();
    this.getData();
    window.addEventListener("resize", this.screenAdapter);
    this.screenAdapter();
  },
  destroyed() {
    window.removeEventListener("resize", this.screenAdapter);
  },
  methods: {
    async initChart() {
      this.chartInstance = this.$echarts.init(this.$refs.map_ref, this.theme);
      //获取中国地图的矢量数据
      //http://localhost:8080/static/map/china.json
      const res = await axios.get(
        "http://localhost:8080/static/map/china.json"
      );
      this.$echarts.registerMap("china", res.data);
      const initOption = {
        title: {
          text: "｜商家分布",
          left: 20,
          top: 20
        },
        geo: {
          type: "map",
          map: "china",
          top: "5%",
          bottom: "5%",
          itemStyle: {
            areaColor: "#2172bf",
            borderColor: "#333"
          }
        },
        legend: {
          left: "5%",
          bottom: "5%",
          orient: "vertical"
        }
      };
      this.chartInstance.setOption(initOption);
      this.chartInstance.on("click", async arg => {
        // arg.name 得到省份的中文名
        // console.log(arg);
        const provinceInfo = getProvinceMapInfo(arg.name);
        // console.log(provinceInfo);
        // 需要获取这个省份的地图矢量数据
        // 判断当前所点击的这个省份的地图矢量数据在mapData中是否存在
        if (!this.mapData[provinceInfo.key]) {
          const res = await axios.get(
            "http://localhost:8080" + provinceInfo.path
          );
          this.mapData[provinceInfo.key] = res.data;
          // console.log(res);
          this.$echarts.registerMap(provinceInfo.key, res.data);
        }

        const changeOption = {
          geo: {
            map: provinceInfo.key
          }
        };
        this.chartInstance.setOption(changeOption);
      });
    },
    async getData() {
      const { data: res } = await this.$http.get("map");
      this.allData = res;
      console.log(this.allData);
      this.updataChart();
    },
    updataChart() {
      // 处理图表需要的数据
      // 图例数据
      const legendArr = this.allData.map(item => {
        return item.name;
      });
      const seriesArr = this.allData.map(item => {
        // return 的对象代表一个类别下所有的散点数据
        // 要显示散点数据，需要给散点的图表增加一个配置，coordinateSystem:geo
        return {
          type: "effectScatter",
          rippleEffect: {
            scale: 5,
            brushType: "stroke"
          },
          name: item.name,
          data: item.children,
          coordinateSystem: "geo"
        };
      });
      const dataOption = {
        legend: {
          data: legendArr
        },
        series: seriesArr
      };
      this.chartInstance.setOption(dataOption);
    },
    screenAdapter() {
      const titleFontSize = (this.$refs.map_ref.offsetWidth / 100) * 3.6;
      const adapterOption = {
        title: {
          textStyle: {
            fontSize: titleFontSize
          }
        },
        legend: {
          itemWidth: titleFontSize / 2,
          itemHeight: titleFontSize / 2,
          itemGap: titleFontSize / 2,
          textStyle: {
            fontSize: titleFontSize / 2
          }
        }
      };
      this.chartInstance.setOption(adapterOption);
      this.chartInstance.resize();
    },
    // 回到中国地图
    revertMap() {
      const revertOption = {
        geo: {
          map: "china"
        }
      };
      this.chartInstance.setOption(revertOption);
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
