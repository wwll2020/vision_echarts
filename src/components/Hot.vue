<template>
  <div class="com-container">
    <div class="com-chart" ref="hot_ref"></div>
    <span class="iconfont arr-left" @click="toLeft" :style="comStyle">&#xe6ef</span>
    <span class="iconfont arr-right" @click="toRight" :style="comStyle">&#xe6ed</span>
    <span class="cat-name" :style="comStyle">{{catName}}</span>
  </div>
</template>
<script>
import { mapState } from "vuex";
import {getThemeValue} from '@/utils/theme_utils'
export default {
  data() {
    return {
      chartInstance: null,
      allData: null,
      currentIndex: 0 ,//当前展示的一级分类数据
      titleFontSize:0
    };
  },
  computed:{
    catName(){
      if(!this.allData){
        return ''
      }else{
        return this.allData[this.currentIndex].name
      }
    },
    comStyle(){
      return {
        fontSize:this.titleFontSize+'px',
        color:getThemeValue(this.theme).titleColor
      }
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
    initChart() {
      this.chartInstance = this.$echarts.init(this.$refs.hot_ref,this.theme);
      const initOption = {
        title:{
          text:'｜热销商品金额占比统计',
          left:20,
          top:20,
        },
        legend:{
          top:'15%',
          icon:'circle'
        },
        tooltip:{
          show:true,
          formatter:(arg)=>{
            // console.log(arg);
          const thirdCategory=  arg.data.children
          // 计算所有三级分类的数值总和
          let total=0;
          thirdCategory.forEach(item=>{
            total+= item.value
          })
          let resStr=''
          thirdCategory.forEach(item=>{
            resStr+=`
            ${item.name}:${parseInt(item.value/total*100)+'%'}
            <br/>
            `
          })
            return resStr
          }
        },
        series: [
          {
            type: "pie",
            label:{
              show:false
            },
            emphasis:{
              label:{
                show:true
              },
              labelLine:{
                show:false
              }
            }
          }
        ]
      };
      this.chartInstance.setOption(initOption);
    },
    async getData() {
      const { data: res } = await this.$http.get("hotproduct");
      this.allData = res;
      console.log(res);
      this.updataChart();
    },
    updataChart() {
      // this.allData[0]
      const legendData = this.allData[this.currentIndex].children.map(item => {
        return item.name;
      });
      const seriesData = this.allData[this.currentIndex].children.map(item => {
        return {
          name: item.name,
          value: item.value,
          children:item.children
        };
      });
      const dataOption = {
        legend: {
          data: legendData
        },
        series: [
          {
            data: seriesData
          }
        ]
      };
      this.chartInstance.setOption(dataOption);
    },
    screenAdapter() {
      this.titleFontSize= this.$refs.hot_ref.offsetWidth/100*3.6
      const adapterOption = {
title:{
  textStyle:{
    fontSize:this.titleFontSize
  }
},
legend:{
  itemWidth:this.titleFontSize,
  itemHeight:this.titleFontSize,
  itemGap:this.titleFontSize,
  textStyle:{
    fontSize:this.titleFontSize/2
  }
},
series:[
  {
  radius:this.titleFontSize*4.5,
  center:['50%','60%']
}
]

      };
      this.chartInstance.setOption(adapterOption);
      this.chartInstance.resize();
    },
    toLeft(){
      this.currentIndex--
      if(this.currentIndex<0){
        this.currentIndex=this.allData.length-1
      }
      this.updataChart()
    },
    toRight(){
      this.currentIndex++
      if(this.currentIndex>this.allData.length-1){
        this.currentIndex=0
      }
      this.updataChart()
    }
  }
};
</script>
<style scoped>
.arr-left {
  position: absolute;
  left: 10%;
  top: 50%;
  transform: translateY(-50%);
  cursor: pointer;
  color:#fff;
}
.arr-right {
  position: absolute;
  right: 10%;
  top: 50%;
  transform: translateY(-50%);
  cursor: pointer;
  color:#fff;

}
.cat-name{
  position: absolute;
  left:80%;
  bottom:20px;
  color:#fff;

}
</style>
