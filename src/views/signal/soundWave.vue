<template>
  <div class="app-container">

    <div class="chart-container">
      <div class="title">
        <span>声音波形</span>
        <span class="radio-label" style="width:160px;position:absolute;right:860px">监测点1</span>
        <el-input v-model="length1" placeholder="监测点1距离" style="width:130px;position:absolute;right:790px" clearable @keyup.native="number" />
        <span class="radio-label" style="width:160px;position:absolute;right:640px">监测点2</span>
        <el-input v-model="length2" placeholder="监测点2距离" style="width:130px;position:absolute;right:570px" clearable @keyup.native="number" />
        <span class="radio-label" style="width:160px;position:absolute;right:420px">监测点3</span>
        <el-input v-model="length3" placeholder="监测点3距离" style="width:130px;position:absolute;right:350px" clearable @keyup.native="number" />
        <span class="radio-label" style="width:160px;position:absolute;right:200px">监测点4</span>
        <el-input v-model="length4" placeholder="监测点4距离" style="width:130px;position:absolute;right:130px" clearable @keyup.native="number" />
        <!-- <el-button style="position:absolute;right:110px" type="primary">修改</el-button> -->
        <el-button v-show="!state.length1 && !state.length2 && !state.length3 && !state.length4" :loading="loading.lengthAll" type="primary" @click="connecttedAll">全部连接</el-button>
        <el-button v-show="state.length1 || state.length2 || state.length3 || state.length4" :loading="loading.lengthAll" type="danger" @click="cancelAll">全部断开</el-button>
      </div>
      <div class="optical">
        <span style="background: #a5db8f;" class="text">光纤长度</span>
      </div>
      <div class="block">
        <el-tooltip v-for="(item,index) in pointList" :key="index" effect="dark" placement="top">
          <div slot="content" style="text-align: center;">{{ item.label }}<br>{{ item.value }}</div>
          <div class="point" :style="{'margin-left':Number(item.value)*100/fiberLength+'%'}" />
        </el-tooltip>
        <el-slider
          v-model="fiberLength"
          :marks="marks"
          :max="fiberLength"
          disabled
        />
      </div>

      <el-divider />
      <div class="title">
        <span>波形曲线图</span>
      </div>
      <el-container class="top-chart">
        <el-aside style="width:49%;">
          <div class="titleText">
            <span>监测点1</span>
            <el-button v-show="!state.length1" :loading="loading.length1" size="medium" style="position:absolute;right:20px" type="primary" @click="connectted1">连接</el-button>
            <el-button v-show="state.length1" :loading="loading.length1" size="medium" style="position:absolute;right:20px" type="danger" @click="cancel1">断开</el-button>
          </div>
          <chartMix height="90%" width="100%" />
        </el-aside>
        <el-main>
          <div class="titleText">
            <span>监测点2</span>
            <el-button v-show="!state.length2" :loading="loading.length2" size="medium" style="position:absolute;right:20px" type="primary" @click="connectted2">连接</el-button>
            <el-button v-show="state.length2" :loading="loading.length2" size="medium" style="position:absolute;right:20px" type="danger" @click="cancel2">断开</el-button>
          </div>
          <chartKeyboard height="90%" width="100%" />
        </el-main>
      </el-container>
      <el-container class="bot-chart">
        <el-aside style="width:49%">
          <div class="titleText">
            <span>监测点3</span>
            <el-button v-show="!state.length3" :loading="loading.length3" size="medium" style="position:absolute;right:20px" type="primary" @click="connectted3">连接</el-button>
            <el-button v-show="state.length3" :loading="loading.length3" size="medium" style="position:absolute;right:20px" type="danger" @click="cancel3">断开</el-button>
          </div>
          <chartLine height="90%" width="100%" />
        </el-aside>
        <el-main>
          <div class="titleText">
            <span>监测点4</span>
            <el-button v-show="!state.length4" :loading="loading.length4" size="medium" style="position:absolute;right:20px" type="primary" @click="connectted4">连接</el-button>
            <el-button v-show="state.length4" :loading="loading.length4" size="medium" style="position:absolute;right:20px" type="danger" @click="cancel4">断开</el-button>
          </div>
          <div id="main" style="height:90%;" />
        </el-main>
      </el-container>
    </div>
  </div>
</template>

<script>
// import { realtimeAudioQuery, vibQuery } from '@/api/signal/soundWave'
import waves from '@/directive/waves' // 水波纹指令
import resize from '@/components/Charts/mixins/resize'
import { baseStandInfo } from '@/api/public'
import ChartMix from '@/components/Charts/MixChart'
import ChartLine from '@/components/Charts/LineMarker'
import ChartKeyboard from '@/components/Charts/Keyboard'
import echarts from 'echarts'
export default {
  name: 'SoundWave',
  directives: {
    waves
  },
  components: { ChartLine, ChartMix, ChartKeyboard },
  mixins: [resize],
  data() {
    return {
      length1: '',
      length2: '',
      length3: '',
      length4: '',
      pointList: [
        { value: '', label: '监测点1' },
        { value: '', label: '监测点2' },
        { value: '', label: '监测点3' },
        { value: '', label: '监测点4' }
      ],
      fiberLength: 30000,
      baseStandInfo: {},
      state: {
        length1: false,
        length2: false,
        length3: false,
        length4: false
      },
      loading: {
        lengthAll: false,
        length1: false,
        length2: false,
        length3: false,
        length4: false
      },
      marks: {
        0: '0',
        5000: '5000',
        10000: '10000',
        15000: '15000',
        20000: '20000',
        25000: '25000',
        30000: '30000',
        35000: '35000',
        40000: '40000',
        45000: '45000',
        50000: '50000',
        55000: '55000',
        60000: '60000',
        65000: '65000',
        70000: '70000'
      }
    }
  },
  watch: {
    '$route.path': function() {
      this.getLength()
    },
    'length1': function(val) {
      this.pointList[0].value = val
      const that = this
      if (Number(val) > that.fiberLength) {
        this.$message.error('监测点距离不得超过光纤长度')
        this.length1 = ''
      }
    },
    'length2': function(val) {
      this.pointList[1].value = val
      const that = this
      if (Number(val) > that.fiberLength) {
        this.$message.error('监测点距离不得超过光纤长度')
        this.length2 = ''
      }
    },
    'length3': function(val) {
      this.pointList[2].value = val
      const that = this
      if (Number(val) > that.fiberLength) {
        this.$message.error('监测点距离不得超过光纤长度')
        this.length3 = ''
      }
    },
    'length4': function(val) {
      this.pointList[3].value = val
      const that = this
      if (Number(val) > that.fiberLength) {
        this.$message.error('监测点距离不得超过光纤长度')
        this.length4 = ''
      }
    }
  },
  created() {
    this.getLength()
  },
  mounted() {
    this.drawChart()
  },
  methods: {
    connecttedAll() {
      const _this = this
      _this.loading.lengthAll = true
      setTimeout(function() {
        _this.state.length1 = true
        _this.state.length2 = true
        _this.state.length3 = true
        _this.state.length4 = true
        _this.$message.success('四个监测点全部连接成功！')
      }, 500)
      this.loading.lengthAll = false
    },
    cancelAll() {
      const _this = this
      _this.loading.lengthAll = true
      setTimeout(function() {
        _this.state.length1 = false
        _this.state.length2 = false
        _this.state.length3 = false
        _this.state.length4 = false
        _this.$message.success('四个监测点全部断开连接！')
      }, 500)
      _this.loading.lengthAll = false
    },
    connectted1() {
      this.loading.length1 = true
      this.state.length1 = !this.state.length1
      this.loading.length1 = false
    },
    cancel1() {
      this.loading.length1 = true
      this.state.length1 = !this.state.length1
      this.loading.length1 = false
    },
    connectted2() {
      this.loading.length2 = true
      this.state.length2 = !this.state.length2
      this.loading.length2 = false
    },
    cancel2() {
      this.loading.length2 = true
      this.state.length2 = !this.state.length2
      this.loading.length2 = false
    },
    connectted3() {
      this.loading.length3 = true
      this.state.length3 = !this.state.length3
      this.loading.length3 = false
    },
    cancel3() {
      this.loading.length3 = true
      this.state.length3 = !this.state.length3
      this.loading.length3 = false
    },
    connectted4() {
      this.loading.length4 = true
      this.state.length4 = !this.state.length4
      this.loading.length4 = false
    },
    cancel4() {
      this.loading.length4 = true
      this.state.length4 = !this.state.length4
      this.loading.length4 = false
    },
    number(val) {
      this.length1 = this.length1.replace(/[^\.\d]/g, '')
      this.length1 = this.length1.replace('.', '')
      this.length2 = this.length2.replace(/[^\.\d]/g, '')
      this.length2 = this.length2.replace('.', '')
      this.length3 = this.length3.replace(/[^\.\d]/g, '')
      this.length3 = this.length3.replace('.', '')
      this.length4 = this.length4.replace(/[^\.\d]/g, '')
      this.length4 = this.length4.replace('.', '')
    },
    getLength() {
      baseStandInfo().then(response => {
        this.baseStandInfo = response.data
        console.log(response.data)
        this.fiberLength = Number(this.baseStandInfo.standNo)
      })
    },
    drawChart() {
    // 基于准备好的dom，初始化echarts实例
      const myChart = echarts.init(document.getElementById('main'))
      // 指定图表的配置项和数据
      const option = {
        title: {
          text: 'ECharts 入门示例'
        },
        tooltip: {},
        legend: {
          data: ['销量']
        },
        xAxis: {
          data: ['衬衫', '羊毛衫', '雪纺衫', '裤子', '高跟鞋', '袜子']
        },
        yAxis: {},
        series: [
          {
            name: '销量',
            type: 'bar',
            data: [5, 20, 36, 10, 10, 20]
          }
        ]
      }
      // 使用刚指定的配置项和数据显示图表。
      myChart.setOption(option)
    }
  }
}

</script>
<style lang="css">
.bot-chart{
  margin-bottom: 0px !important;
  margin-top: 0px !important;
}
.el-aside {
  /* backbackground-color: #D3DCE6; */
  background:transparent !important;
  color: #333;
  text-align: left;
  height: 450px;
  margin-right: 2% !important;
  padding: 0 !important;
}
.el-main {
  /* background-color: #E9EEF3; */
  color: #333;
  text-align: center;
  height: 450px;
  padding: 0 !important;
}
.el-slider__runway{
  background-color: #1890ff;
}
.el-slider__runway.disabled .el-slider__bar{
  background-color: #1890ff
}
.el-slider__runway.disabled .el-slider__button {
    opacity: 0.0;
}
.el-slider__button {
    opacity: 0.0;
}
.optical{
  height: 30px;
  text-align: center;
  margin: 20px 40% 20px 40%;
}
.text{
  font-size: 22px;
  height: 12px;
  width: 66px;
  margin-top: 5px;
}
.block{
  position: relative;
  margin: 10px;
  height: 55px;
  padding-bottom: 10px;
}
</style>
<style rel="stylesheet/scss" lang="scss" scoped>
.chart-container {
  width: 100%;
  height: 100%;
  position: relative;
}
.radio-label {
  font-size: 18px;
  color: #606266;
  line-height: 40px;
  padding: 0 12px 0 30px;
}
.line{
  width: 99%;
  height: 2px;
  background-color: #1890ff;
  border-top-left-radius: 3px;
  border-bottom-left-radius: 3px;
  position: absolute;
}
.point{
    position: absolute;
    z-index: 999;
    height: 10px;
    width: 10px;
    border-radius: 100%;
    -webkit-transform: translateX(-50%);
    transform: translateX(-50%);
    background-color: red;
    top: 14px;
}
.title {
  height: 40px;
  position: relative;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size:22px;
  color:rgba(0,0,0,1);
  line-height:42px;
}
.titleText {
  height: 10%;
  position: relative;
  margin-left: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size:20px;
  color:rgba(0,0,0,1);
  line-height:10%;
}
</style>
<style>
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
}
input[type="number"] {
  -moz-appearance: textfield;
}
</style>
