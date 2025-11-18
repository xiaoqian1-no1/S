<template>
  <div class="business-analysis">
    <div class="page-header">
      <h2>
        <i class="el-icon-s-data"></i>
        经营分析
      </h2>
      <el-button type="primary" @click="refreshData">
        <i class="el-icon-refresh"></i>
        刷新数据
      </el-button>
    </div>

    <!-- 统计卡片 -->
    <el-row :gutter="20" class="stats-row">
      <el-col :span="6">
        <el-card class="stat-card">
          <div class="stat-icon" style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);">
            <i class="el-icon-s-goods"></i>
          </div>
          <div class="stat-content">
            <div class="stat-label">预计总产量</div>
            <div class="stat-value">{{ formatNumber(totalYield) }}<span class="unit">公斤</span></div>
          </div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card class="stat-card">
          <div class="stat-icon" style="background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);">
            <i class="el-icon-money"></i>
          </div>
          <div class="stat-content">
            <div class="stat-label">预计总收益</div>
            <div class="stat-value">{{ formatNumber(totalRevenue) }}<span class="unit">元</span></div>
          </div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card class="stat-card">
          <div class="stat-icon" style="background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);">
            <i class="el-icon-s-finance"></i>
          </div>
          <div class="stat-content">
            <div class="stat-label">预计总成本</div>
            <div class="stat-value">{{ formatNumber(totalCost) }}<span class="unit">元</span></div>
          </div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card class="stat-card">
          <div class="stat-icon" style="background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%);">
            <i class="el-icon-success"></i>
          </div>
          <div class="stat-content">
            <div class="stat-label">预计总利润</div>
            <div class="stat-value" :style="{ color: totalProfit >= 0 ? '#67C23A' : '#F56C6C' }">
              {{ formatNumber(totalProfit) }}<span class="unit">元</span>
            </div>
          </div>
        </el-card>
      </el-col>
    </el-row>

    <!-- 图表区域 -->
    <el-row :gutter="20" class="charts-row">
      <!-- 各地块预计利润对比 -->
      <el-col :span="16">
        <el-card>
          <div slot="header" class="card-header">
            <span>📊 各地块预计利润对比</span>
          </div>
          <div ref="profitChart" style="height: 400px;"></div>
        </el-card>
      </el-col>

      <!-- 按作物种类的利润占比 -->
      <el-col :span="8">
        <el-card>
          <div slot="header" class="card-header">
            <span>🥧 按作物种类的利润占比</span>
          </div>
          <div ref="cropProfitChart" style="height: 400px;"></div>
        </el-card>
      </el-col>
    </el-row>

    <!-- 详细数据表格 -->
    <el-card class="table-card">
      <div slot="header" class="card-header">
        <span>📋 详细经营数据</span>
      </div>
      <el-table :data="farmlandData" border stripe style="width: 100%">
        <el-table-column prop="farmlandName" label="农田名称" width="150"></el-table-column>
        <el-table-column prop="crop" label="作物类型" width="120"></el-table-column>
        <el-table-column prop="area" label="面积（亩）" width="100" align="right"></el-table-column>
        <el-table-column prop="expectedYield" label="预计产量（公斤）" width="140" align="right">
          <template slot-scope="scope">
            {{ formatNumber(scope.row.expectedYield) }}
          </template>
        </el-table-column>
        <el-table-column prop="expectedRevenue" label="预计收益（元）" width="140" align="right">
          <template slot-scope="scope">
            {{ formatNumber(scope.row.expectedRevenue) }}
          </template>
        </el-table-column>
        <el-table-column prop="expectedCost" label="预计成本（元）" width="140" align="right">
          <template slot-scope="scope">
            {{ formatNumber(scope.row.expectedCost) }}
          </template>
        </el-table-column>
        <el-table-column prop="profit" label="预计利润（元）" width="140" align="right">
          <template slot-scope="scope">
            <span :style="{ color: scope.row.profit >= 0 ? '#67C23A' : '#F56C6C' }">
              {{ formatNumber(scope.row.profit) }}
            </span>
          </template>
        </el-table-column>
        <el-table-column label="利润率" width="100" align="right">
          <template slot-scope="scope">
            <span :style="{ color: scope.row.profitRate >= 0 ? '#67C23A' : '#F56C6C' }">
              {{ scope.row.profitRate }}%
            </span>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
import * as echarts from 'echarts';

export default {
  name: 'BusinessAnalysis',
  data() {
    return {
      farmlandData: [],
      cropProfitData: {},
      totalYield: 0,
      totalRevenue: 0,
      totalCost: 0,
      totalProfit: 0,
      profitChart: null,
      cropProfitChart: null
    }
  },
  mounted() {
    this.loadData();
    window.addEventListener('resize', this.handleResize);
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.handleResize);
    if (this.profitChart) {
      this.profitChart.dispose();
    }
    if (this.cropProfitChart) {
      this.cropProfitChart.dispose();
    }
  },
  methods: {
    async loadData() {
      try {
        const res = await this.request.get('/business-analysis/all');
        if (res.code === '200' && res.data) {
          this.farmlandData = res.data.farmlandData || [];
          this.cropProfitData = res.data.cropProfit || {};
          
          // 计算总计
          this.calculateTotals();
          
          // 更新图表
          this.$nextTick(() => {
            this.initCharts();
          });
        }
      } catch (error) {
        console.error('加载经营分析数据失败:', error);
        this.$message.error('加载数据失败');
      }
    },
    calculateTotals() {
      this.totalYield = this.farmlandData.reduce((sum, item) => {
        return sum + (parseFloat(item.expectedYield) || 0);
      }, 0);
      
      this.totalRevenue = this.farmlandData.reduce((sum, item) => {
        return sum + (parseFloat(item.expectedRevenue) || 0);
      }, 0);
      
      this.totalCost = this.farmlandData.reduce((sum, item) => {
        return sum + (parseFloat(item.expectedCost) || 0);
      }, 0);
      
      this.totalProfit = this.farmlandData.reduce((sum, item) => {
        return sum + (parseFloat(item.profit) || 0);
      }, 0);
      
      // 计算利润率
      this.farmlandData.forEach(item => {
        const revenue = parseFloat(item.expectedRevenue) || 0;
        const profit = parseFloat(item.profit) || 0;
        item.profitRate = revenue > 0 ? ((profit / revenue) * 100).toFixed(2) : 0;
      });
    },
    initCharts() {
      this.initProfitChart();
      this.initCropProfitChart();
    },
    initProfitChart() {
      if (!this.$refs.profitChart) return;
      
      this.profitChart = echarts.init(this.$refs.profitChart);
      
      const farmlandNames = this.farmlandData.map(item => item.farmlandName);
      const profits = this.farmlandData.map(item => parseFloat(item.profit) || 0);
      
      const option = {
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'shadow'
          },
          formatter: (params) => {
            const data = params[0];
            const item = this.farmlandData[data.dataIndex];
            return `
              <div style="padding: 5px;">
                <div><strong>${data.name}</strong></div>
                <div>预计利润: <span style="color: ${data.value >= 0 ? '#67C23A' : '#F56C6C'}">${this.formatNumber(data.value)}元</span></div>
                <div>预计收益: ${this.formatNumber(item.expectedRevenue)}元</div>
                <div>预计成本: ${this.formatNumber(item.expectedCost)}元</div>
              </div>
            `;
          }
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: {
          type: 'category',
          data: farmlandNames,
          axisLabel: {
            rotate: 45,
            interval: 0
          }
        },
        yAxis: {
          type: 'value',
          name: '利润（元）'
        },
        series: [{
          name: '预计利润',
          type: 'bar',
          data: profits,
          itemStyle: {
            color: (params) => {
              return params.value >= 0 ? '#67C23A' : '#F56C6C';
            }
          },
          label: {
            show: true,
            position: 'top',
            formatter: (params) => {
              return this.formatNumber(params.value);
            }
          }
        }]
      };
      
      this.profitChart.setOption(option);
    },
    initCropProfitChart() {
      if (!this.$refs.cropProfitChart) return;
      
      this.cropProfitChart = echarts.init(this.$refs.cropProfitChart);
      
      const cropNames = Object.keys(this.cropProfitData);
      const cropProfits = cropNames.map(name => {
        const profit = parseFloat(this.cropProfitData[name]) || 0;
        return {
          name: name,
          value: profit
        };
      });
      
      const option = {
        tooltip: {
          trigger: 'item',
          formatter: '{b}: {c}元 ({d}%)'
        },
        legend: {
          orient: 'vertical',
          left: 'left',
          top: 'middle'
        },
        series: [{
          name: '利润占比',
          type: 'pie',
          radius: ['40%', '70%'],
          avoidLabelOverlap: false,
          itemStyle: {
            borderRadius: 10,
            borderColor: '#fff',
            borderWidth: 2
          },
          label: {
            show: true,
            formatter: '{b}\n{d}%'
          },
          emphasis: {
            label: {
              show: true,
              fontSize: '16',
              fontWeight: 'bold'
            }
          },
          data: cropProfits
        }]
      };
      
      this.cropProfitChart.setOption(option);
    },
    handleResize() {
      if (this.profitChart) {
        this.profitChart.resize();
      }
      if (this.cropProfitChart) {
        this.cropProfitChart.resize();
      }
    },
    refreshData() {
      this.loadData();
      this.$message.success('数据已刷新');
    },
    formatNumber(num) {
      if (num === null || num === undefined) return '0.00';
      const value = parseFloat(num);
      if (isNaN(value)) return '0.00';
      return value.toLocaleString('zh-CN', {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2
      });
    }
  }
}
</script>

<style scoped>
.business-analysis {
  padding: 20px;
  background: #f5f7fa;
  min-height: calc(100vh - 60px);
}

.page-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.page-header h2 {
  margin: 0;
  font-size: 24px;
  color: #303133;
  display: flex;
  align-items: center;
  gap: 10px;
}

.stats-row {
  margin-bottom: 20px;
}

.stat-card {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 20px;
  transition: all 0.3s;
}

.stat-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}

.stat-icon {
  width: 60px;
  height: 60px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 28px;
  color: white;
  flex-shrink: 0;
}

.stat-content {
  flex: 1;
}

.stat-label {
  font-size: 14px;
  color: #909399;
  margin-bottom: 8px;
}

.stat-value {
  font-size: 28px;
  font-weight: bold;
  color: #303133;
}

.stat-value .unit {
  font-size: 14px;
  color: #909399;
  margin-left: 5px;
}

.charts-row {
  margin-bottom: 20px;
}

.card-header {
  font-weight: bold;
  font-size: 16px;
}

.table-card {
  margin-top: 20px;
}
</style>

