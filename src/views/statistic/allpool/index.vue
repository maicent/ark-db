<template>
  <div class="app-container">
    <el-select v-model="poolName" clearable filterable placeholder="请选择卡池">
      <el-option
        v-for="item in options"
        :key="item.value"
        :value="item.value"
      />
    </el-select>
    <el-button class="filter-item" type="primary" icon="el-icon-search" @click="search()">
      搜索
    </el-button>

    <div id="main" :class="className" :style="{height:height,width:width}" />

    <el-alert
      title="卡池概率"
      type="success"
      :closable="false"
      :description="probability"
      show-icon
    />

    <div><h3>抽到的六星</h3></div>
    <el-table v-loading="listLoading" :data="list" element-loading-text="Loading" border fit highlight-current-row max-height="280">
      <el-table-column label="干员名称" align="center">
        <template slot-scope="scope">
          {{ scope.row.name }}
        </template>
      </el-table-column>
      <el-table-column label="抽取数量" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.num }}</span>
        </template>
      </el-table-column>
    </el-table>

  </div>
</template>

<script>
import { getPoolName, getPoolbyName } from '@/api/gacha'

import * as echarts from 'echarts'
require('echarts/theme/macarons')
import resize from '../../dashboard/components/mixins/resize'

export default {
  mixins: [resize],
  props: {
    className: {
      type: String,
      default: 'chart'
    },
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '400px'
    }
  },
  data() {
    return {
      options: [],
      chart: null,
      poolName: '',
      probability: '在上方选择卡池以获取概率', // 卡池概率
      pie: [], // 饼图数据
      listLoading: true, // 表格加载
      list: null // 表格数据
    }
  },
  watch: {
    // 监听卡池名称变化
    poolName() {
      this.getData().then(res => {
        this.drawPie()
      })
    }
  },
  mounted() {
    this.getItem() // 获取卡池名称
    this.initChart() // 渲染图表
  },
  beforeDestroy() {
    if (!this.chart) {
      return
    }
    this.chart.dispose()
    this.chart = null
  },
  methods: {
    getItem() {
      getPoolName().then(response => {
        this.options = response.data.name
        this.listLoading = false
      })
    },
    search() {

    },
    async getData() {
      await getPoolbyName({ 'poolName': this.poolName }).then(res => {
        this.probability = Number(res.data.data[0].value / res.data.data[1].value * 100).toFixed(1) + '%'
        this.pie = res.data.data
        this.list = res.data.data[0].top3
        this.listLoading = false
      })
      return 0
    },
    initChart() {
      this.chart = echarts.init(document.getElementById('main'), 'macarons')
      this.drawPie()
    },
    drawPie() {
      this.chart.setOption({
        title: {
          text: '全站卡池六星统计',
          subtext: '请在上方选择查看数据的卡池',
          left: 'center'
        },
        tooltip: {
          trigger: 'item'
        },
        legend: {
          orient: 'vertical',
          left: 'left'
        },
        series: [
          {
            name: '寻访',
            type: 'pie',
            radius: '50%',
            data: this.pie,
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
  }
}
</script>

<style scoped>
.filter-item{
  margin: 0px 0px 0px 10px
}
.showchart{
  margin-top: 20px;
}
</style>

