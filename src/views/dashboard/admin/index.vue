<template>
  <el-container>
    <el-main>
      <el-row :gutter="40" class="panel-group">
        <el-col :xs="12" :sm="12" :lg="6" class="card-panel-col">
          <div class="card-panel" @click="handleSetLineChartData('newVisitis')">
            <div class="card-panel-icon-wrapper icon-people">
              <svg-icon icon-class="peoples" class-name="card-panel-icon" />
            </div>
            <div class="card-panel-description">
              <div class="card-panel-text">
                用户总数
              </div>
              <div style="font-weight: bold; font-size: 1.5em;">
                {{ data.user }}
              </div>
              <!--              <count-to :start-val="0" :end-val="data.user" :duration="2600" class="card-panel-num" />-->
            </div>
          </div>
        </el-col>
        <el-col :xs="12" :sm="12" :lg="6" class="card-panel-col">
          <div class="card-panel" @click="handleSetLineChartData('messages')">
            <div class="card-panel-icon-wrapper icon-message">
              <svg-icon icon-class="message" class-name="card-panel-icon" />
            </div>
            <div class="card-panel-description">
              <div class="card-panel-text">
                话题总数
              </div>
              <div style="font-weight: bold; font-size: 1.5em;">
                {{ data.topical }}
              </div>
              <!--              <count-to :start-val="0" :end-val="data.topical" :duration="3000" class="card-panel-num" />-->
            </div>
          </div>
        </el-col>
        <el-col :xs="12" :sm="12" :lg="6" class="card-panel-col">
          <div class="card-panel" @click="handleSetLineChartData('purchases')">
            <div class="card-panel-icon-wrapper icon-money">
              <i class="el-icon-s-opportunity card-panel-icon" />
            </div>
            <div class="card-panel-description">
              <div class="card-panel-text">
                经验总数
              </div>
              <div style="font-weight: bold; font-size: 1.5em;">
                {{ data.share }}
              </div>
              <!--              <count-to :start-val="0" :end-val="data.share" :duration="3200" class="card-panel-num" />-->
            </div>
          </div>
        </el-col>
        <el-col :xs="12" :sm="12" :lg="6" class="card-panel-col">
          <div class="card-panel" @click="handleSetLineChartData('shoppings')">
            <div class="card-panel-icon-wrapper icon-shopping">
              <i class="el-icon-postcard card-panel-icon" />
            </div>
            <div class="card-panel-description">
              <div class="card-panel-text">
                打卡总数
              </div>
              <div style="font-weight: bold; font-size: 1.5em;">
                {{ data.card }}
              </div>
              <!--              <count-to :start-val="0" :end-val="data.card" :duration="3600" class="card-panel-num" />-->
            </div>
          </div>
        </el-col>
      </el-row>
      <el-row style="width: 100%">
        <div id="chart1" style="width:50%;height:400px;float: left" />
        <div id="chart2" style="width:50%;height:400px;float: left" />
        <div id="chart4" style="width:50%;height:400px;float: left" />
        <div id="chart3" style="width:50%;height:400px;float: left" />
        <!--          <div id="view01" style="width: 90%;">Hello world</div>-->
        <!--          <div id="view02" style="width: 90%;">Hello world1</div>-->
        <!--          <div id="view03" style="width: 90%;">Hello world2</div>-->
      </el-row>
    </el-main>
  </el-container>
</template>

<script>

// import CountTo from 'vue-count-to'
import echarts from 'echarts'

export default {
  name: 'Dashboard',
  // components: {
  //   CountTo
  // },
  data() {
    const data = {
      user: 0,
      topical: 0,
      share: 0,
      card: 0
    }
    // 获取数据
    this.tools.requests(this.G.SERVER + '/api/admin/visualization/overall', {}, 'get').then((response) => {
      if (response != null && response.code === 1) {
        this.data = response.data
      }
    })
    return {
      data
    }
  },
  mounted() {
    const myChart = echarts.init(document.getElementById('chart1'))
    // 获取每个学院的人数分布
    this.tools.requests(this.G.SERVER + '/api/admin/data/getCollege', {}, 'get').then((response) => {
      if (response != null && response.code === 1) {
        myChart.setOption({
          title: {
            text: '学院分布情况',
            left: 'center'
          },
          tooltip: {
            trigger: 'item',
            formatter: '{a} <br/>{b} : {c} ({d}%)'
          },
          legend: {
            // orient: 'vertical',
            left: 'left',
            data: response.data.college
          },
          series: [
            {
              name: '访问来源',
              type: 'pie',
              radius: '55%',
              center: ['50%', '60%'],
              data: response.data.data,
              emphasis: {
                itemStyle: {
                  shadowBlur: 10,
                  shadowOffsetX: 0,
                  shadowColor: 'rgba(0, 0, 0, 0.5)'
                }
              }
            }
          ]
        })
      }
    })
    // 获取经验数据分布
    const myChart2 = echarts.init(document.getElementById('chart2'))
    this.tools.requests(this.G.SERVER + '/api/admin/data/getShare', {}, 'get').then((response) => {
      if (response != null && response.code === 1) {
        myChart2.setOption({
          title: {
            text: '经验数据分布统计',
            left: 'center'
          },
          legend: {
            left: 'right'
          },
          tooltip: {},
          dataset: {
            source: response.data
          },
          xAxis: { type: 'category' },
          yAxis: {},
          // Declare several bar series, each will be mapped
          // to a column of dataset.source by default.
          series: [
            { type: 'bar' },
            { type: 'bar' },
            { type: 'bar' }
          ]
        })
      }
    })
    // 获取话题分布统计
    const myChart3 = echarts.init(document.getElementById('chart3'))
    this.tools.requests(this.G.SERVER + '/api/admin/data/getTopical', {}, 'get').then((response) => {
      if (response != null && response.code === 1) {
        myChart3.setOption({
          title: {
            text: '话题分布统计',
            left: 'center'
          },
          tooltip: {
            trigger: 'axis',
            axisPointer: {
              type: 'shadow'
            }
          },
          legend: {
            data: ['话题标题', '点赞数', '评论数', '浏览量'],
            left: 'right'
          },
          grid: {
            left: '3%',
            right: '4%',
            bottom: '3%',
            containLabel: true
          },
          xAxis: {
            type: 'value',
            boundaryGap: [0, 0.01]
          },
          yAxis: {
            type: 'category',
            data: response.data[0]
          },
          series: [
            {
              name: '点赞数',
              type: 'bar',
              data: response.data[1]
            },
            {
              name: '评论数',
              type: 'bar',
              data: response.data[2]
            },
            {
              name: '浏览量',
              type: 'bar',
              data: response.data[3]
            }
          ]
        })
      }
    })
    // 获取打卡分布
    const myChart4 = echarts.init(document.getElementById('chart4'))
    this.tools.requests(this.G.SERVER + '/api/admin/data/getCard', {}, 'get').then((response) => {
      if (response != null && response.code === 1) {
        myChart4.setOption({
          title: {
            text: '打卡数据统计',
            left: 'center'
          },
          tooltip: {
            trigger: 'axis',
            axisPointer: {
              type: 'shadow'
            }
          },
          legend: {
            data: ['坚持天数', '加入人数'],
            left: 'left',
            textStyle: {
              color: '#ccc'
            }
          },
          xAxis: {
            data: response.data[0],
            axisLine: {
              lineStyle: {
                color: '#ccc'
              }
            }
          },
          yAxis: {
            splitLine: { show: false },
            axisLine: {
              lineStyle: {
                color: '#ccc'
              }
            }
          },
          series: [{
            name: '加入人数',
            type: 'line',
            smooth: true,
            showAllSymbol: true,
            symbol: 'emptyCircle',
            symbolSize: 15,
            data: response.data[2]
          }, {
            name: '坚持天数',
            type: 'bar',
            barWidth: 10,
            itemStyle: {
              barBorderRadius: 5,
              color: new echarts.graphic.LinearGradient(
                0, 0, 0, 1,
                [
                  { offset: 0, color: '#14c8d4' },
                  { offset: 1, color: '#43eec6' }
                ]
              )
            },
            data: response.data[1]
          }]
        })
      }
    })
  },
  methods: {
    handleSetLineChartData(type) {
    }
  }
}
</script>

<style lang="scss" scoped>
  .panel-group {
    margin-top: 18px;

    .card-panel-col {
      margin-bottom: 32px;
    }

    .card-panel {
      height: 108px;
      cursor: pointer;
      font-size: 12px;
      position: relative;
      overflow: hidden;
      color: #666;
      background: #fff;
      box-shadow: 4px 4px 40px rgba(0, 0, 0, .05);
      border-color: rgba(0, 0, 0, .05);

      &:hover {
        .card-panel-icon-wrapper {
          color: #fff;
        }

        .icon-people {
          background: #40c9c6;
        }

        .icon-message {
          background: #36a3f7;
        }

        .icon-money {
          background: #f4516c;
        }

        .icon-shopping {
          background: #34bfa3
        }
      }

      .icon-people {
        color: #40c9c6;
      }

      .icon-message {
        color: #36a3f7;
      }

      .icon-money {
        color: #f4516c;
      }

      .icon-shopping {
        color: #34bfa3
      }

      .card-panel-icon-wrapper {
        float: left;
        margin: 14px 0 0 14px;
        padding: 16px;
        transition: all 0.38s ease-out;
        border-radius: 6px;
      }

      .card-panel-icon {
        float: left;
        font-size: 48px;
      }

      .card-panel-description {
        float: right;
        font-weight: bold;
        margin: 26px;
        margin-left: 0px;

        .card-panel-text {
          line-height: 18px;
          color: rgba(0, 0, 0, 0.45);
          font-size: 16px;
          margin-bottom: 12px;
        }

        .card-panel-num {
          font-size: 20px;
        }
      }
    }
  }

  @media (max-width:550px) {
    .card-panel-description {
      display: none;
    }

    .card-panel-icon-wrapper {
      float: none !important;
      width: 100%;
      height: 100%;
      margin: 0 !important;

      .svg-icon {
        display: block;
        margin: 14px auto !important;
        float: none !important;
      }
    }
  }
</style>
