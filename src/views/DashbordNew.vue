<template>
  <div class="dashboard-container">
    <!-- 顶部标题栏 -->
    <div class="dashboard-header">
      <div class="header-left">
        <i class="el-icon-monitor"></i>
        <span class="header-title">农场环境监测中心</span>
      </div>
      <div class="header-right">
        <el-select v-model="selectedOption" placeholder="选择地块" class="farm-selector">
          <el-option 
            v-for="option in options" 
            :key="option.value"
            :value="option.value" 
            :label="option.label"
          >
            <span style="float: left">{{ option.label }}</span>
            <span style="float: right; color: #8492a6; font-size: 13px">{{ option.crop }}</span>
          </el-option>
        </el-select>
      </div>
    </div>

    <!-- 环境健康评分卡片 -->
    <div class="health-score-card">
      <div class="score-left">
        <div class="score-circle" :style="{ background: getHealthGradient() }">
          <div class="score-inner">
            <div class="score-number">{{ healthScore }}</div>
            <div class="score-label">健康指数</div>
          </div>
        </div>
      </div>
      <div class="score-right">
        <div class="score-status" :class="getHealthClass()">
          {{ getHealthText() }}
        </div>
        <div class="score-desc">{{ getHealthDesc() }}</div>
        <div class="score-metrics">
          <div class="metric-tag" v-for="metric in getTopMetrics()" :key="metric.name" :class="metric.status">
            {{ metric.icon }} {{ metric.name }}: {{ metric.status === 'good' ? '正常' : '异常' }}
          </div>
        </div>
      </div>
    </div>

    <!-- 主要内容区域 -->
    <div class="dashboard-content">
      <!-- 左侧：环境数据趋势 -->
      <div class="content-left">
        <div class="card env-trend-card">
          <div class="card-header">
            <span class="card-title">📈 环境数据趋势</span>
            <el-radio-group v-model="chartType" size="mini">
              <el-radio-button label="line">折线图</el-radio-button>
              <el-radio-button label="bar">柱状图</el-radio-button>
            </el-radio-group>
          </div>
          <div class="card-body">
            <div ref="envChart" class="chart-container"></div>
          </div>
        </div>

        <div class="card radar-card">
          <div class="card-header">
            <span class="card-title">🎯 实时环境雷达</span>
            <span class="update-time">{{ updateTime }}</span>
          </div>
          <div class="card-body">
            <div ref="radarChart" class="chart-container-small"></div>
          </div>
        </div>
      </div>

      <!-- 右侧：设备统计和视频监控 -->
      <div class="content-right">
        <div class="card device-card">
          <div class="card-header">
            <span class="card-title">🔧 设备运行状态</span>
          </div>
          <div class="card-body">
            <div ref="deviceChart" class="chart-container-small"></div>
            <div class="device-stats">
              <div class="device-stat-item" v-for="device in deviceStats" :key="device.name">
                <div class="device-icon" :style="{ background: device.color }">{{ device.icon }}</div>
                <div class="device-info">
                  <div class="device-name">{{ device.name }}</div>
                  <div class="device-count">{{ device.count }} 台</div>
                </div>
                <div class="device-status" :class="device.status">{{ device.statusText }}</div>
              </div>
            </div>
          </div>
        </div>

        <div class="card video-card">
          <div class="card-header">
            <span class="card-title">📹 实时视频监控</span>
            <div class="video-controls">
              <i class="el-icon-video-play" @click="play" v-if="!isPlaying"></i>
              <i class="el-icon-video-pause" @click="pause" v-else></i>
            </div>
          </div>
          <div class="card-body video-body">
            <video 
              ref="videoPlayer" 
              muted 
              autoplay 
              loop 
              @play="isPlaying = true"
              @pause="isPlaying = false"
              class="video-player"
            >
              <source :src="videoUrls[selectedOption - 1]" type="video/mp4">
            </video>
            <div class="video-overlay">
              <div class="video-info">
                <span class="video-location">{{ options[selectedOption - 1] ? options[selectedOption - 1].label : '加载中...' }}</span>
                <span class="video-time">{{ currentTime }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const echarts = require('echarts');

export default {
  name: 'Dashboard',
  data() {
    return {
      selectedOption: '1',
      chartType: 'line',
      isPlaying: true,
      currentTime: '',
      updateTime: '',
      
      options: [],
      farms: [],
      
      videoUrls: [
        'https://sf9-dycdn-tos.pstatp.com/obj/tos-cn-i-8gu37r9deh/2c45f950182347dd968921790cb9530f?filename=1.mp4',
        'https://lf1-cdn-tos.bytegoofy.com/obj/tos-cn-v-0000/55a120d8113b484686a83a0befe3e54e?filename=1.mp4',
        'https://lf1-cdn-tos.bytegoofy.com/obj/tos-cn-v-0000/605d6560789d4289b1947d3f480e9a32?filename=1.mp4'
      ],
      
      deviceStats: [
        { name: '采集设备', count: 20, icon: '📡', color: '#667eea', status: 'online', statusText: '在线' },
        { name: '控制设备', count: 10, icon: '🎮', color: '#f093fb', status: 'online', statusText: '在线' },
        { name: '气象站', count: 4, icon: '🌤️', color: '#4facfe', status: 'online', statusText: '在线' },
        { name: '监控设备', count: 4, icon: '📹', color: '#43e97b', status: 'warning', statusText: '警告' }
      ],
      
      envChart: null,
      radarChart: null,
      deviceChart: null,
      
      envData: {
        '1': {
          temp: [20, 32, 10, 14, 19, 23, 21],
          airHum: [60, 72, 61, 54, 57, 52, 70],
          soilHum: [50, 32, 20, 44, 40, 30, 35],
          carbon: [320, 432, 501, 434, 390, 430, 620],
          ph: [6.2, 7.0, 6.8, 8.4, 6.3, 8.0, 6.0],
          light: [2000, 1932, 1901, 1934, 2090, 2330, 2620],
          current: [31, 70, 40, 950, 8.2, 1800]
        },
        '2': {
          temp: [20, 32, 10, 34, 19, 23, 21],
          airHum: [62, 82, 91, 4, 90, 59, 73],
          soilHum: [50, 32, 20, 54, 40, 33, 41],
          carbon: [620, 532, 701, 634, 690, 530, 620],
          ph: [8.0, 9.2, 9.1, 9.4, 7.0, 6.3, 7.6],
          light: [2820, 1932, 2901, 1934, 2290, 3330, 2320],
          current: [20, 50, 30, 750, 6.0, 2000]
        },
        '3': {
          temp: [30, 22, 21, 34, 27, 20, 33],
          airHum: [60, 82, 91, 64, 90, 30, 70],
          soilHum: [50, 32, 20, 54, 60, 30, 41],
          carbon: [720, 832, 601, 734, 590, 730, 820],
          ph: [8.0, 9.2, 7.1, 9.4, 10.0, 7.4, 6.8],
          light: [2820, 2932, 1901, 3934, 2290, 3330, 2320],
          current: [27, 80, 43, 1735, 7, 2200]
        }
      }
    };
  },
  
  computed: {
    healthScore() {
      const data = this.envData[this.selectedOption];
      if (!data || !data.current) return 85;
      
      const [temp, airHum, soilHum, carbon, ph, light] = data.current;
      
      const metrics = [
        { value: temp, excellent: [20, 28], good: [15, 32] },
        { value: airHum, excellent: [60, 80], good: [50, 90] },
        { value: soilHum, excellent: [40, 70], good: [30, 80] },
        { value: carbon, excellent: [350, 450], good: [300, 1500] },
        { value: ph, excellent: [6.0, 7.5], good: [5.5, 8.0] },
        { value: light, excellent: [2000, 3000], good: [1500, 3500] }
      ];
      
      const score = metrics.reduce((total, metric) => {
        const { value, excellent, good } = metric;
        if (value >= excellent[0] && value <= excellent[1]) return total + 100;
        if (value >= good[0] && value <= good[1]) return total + 75;
        return total + 50;
      }, 0);
      
      return Math.round(score / metrics.length);
    }
  },
  
  mounted() {
    this.updateCurrentTime();
    this.timeInterval = setInterval(this.updateCurrentTime, 1000);
    
    // 先初始化图表容器
    this.$nextTick(() => {
      this.initCharts();
    });
    
    // 加载农田数据
    this.loadFarmData();
  },
  
  beforeDestroy() {
    if (this.timeInterval) {
      clearInterval(this.timeInterval);
    }
    if (this.envChart) this.envChart.dispose();
    if (this.radarChart) this.radarChart.dispose();
    if (this.deviceChart) this.deviceChart.dispose();
  },
  
  watch: {
    selectedOption() {
      this.updateCharts();
      this.$refs.videoPlayer.load();
    },
    chartType() {
      this.updateEnvChart();
    }
  },
  
  methods: {
    async loadFarmData() {
      try {
        const res = await this.request.get('/statistic/dashboard');
        if (res.code === '200' && res.data) {
          this.farms = res.data;
          
          // 将农田数据转换为options格式
          this.options = this.farms.map((farm, index) => ({
            label: farm.farm,
            value: String(index + 1),
            crop: farm.crop
          }));
          
          // 设置默认选中第一个
          if (this.options.length > 0) {
            this.selectedOption = this.options[0].value;
          }
          
          // 数据加载成功
          
          // 数据加载完成后，更新图表
          this.$nextTick(() => {
            if (this.envChart && this.envData[this.selectedOption]) {
              this.updateCharts();
            }
          });
        }
      } catch (error) {
        console.error('加载农田数据失败:', error);
        // 如果加载失败，使用默认数据
        this.options = [
          { label: 'A1号田', value: '1', crop: '小麦' },
          { label: 'A2号田', value: '2', crop: '玉米' },
          { label: 'A3号田', value: '3', crop: '水稻' }
        ];
        
        // 使用默认数据后也更新图表
        this.$nextTick(() => {
          if (this.envChart && this.envData[this.selectedOption]) {
            this.updateCharts();
          }
        });
      }
    },
    
    updateCurrentTime() {
      const now = new Date();
      const hours = String(now.getHours()).padStart(2, '0');
      const minutes = String(now.getMinutes()).padStart(2, '0');
      const seconds = String(now.getSeconds()).padStart(2, '0');
      this.currentTime = `${hours}:${minutes}:${seconds}`;
      this.updateTime = `更新时间: ${hours}:${minutes}`;
    },
    
    getHealthLevel() {
      const score = this.healthScore;
      const levels = [
        { min: 90, gradient: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)', class: 'excellent', text: '优秀', desc: '环境条件非常适宜作物生长' },
        { min: 75, gradient: 'linear-gradient(135deg, #43e97b 0%, #38f9d7 100%)', class: 'good', text: '良好', desc: '环境条件适宜，可持续监测' },
        { min: 60, gradient: 'linear-gradient(135deg, #fa709a 0%, #fee140 100%)', class: 'normal', text: '一般', desc: '部分指标需要调整优化' },
        { min: 0, gradient: 'linear-gradient(135deg, #f093fb 0%, #f5576c 100%)', class: 'poor', text: '需改善', desc: '环境条件需要及时调整' }
      ];
      return levels.find(level => score >= level.min);
    },
    
    getHealthGradient() {
      return this.getHealthLevel().gradient;
    },
    
    getHealthClass() {
      return this.getHealthLevel().class;
    },
    
    getHealthText() {
      return this.getHealthLevel().text;
    },
    
    getHealthDesc() {
      return this.getHealthLevel().desc;
    },
    
    getTopMetrics() {
      const envDataItem = this.envData[this.selectedOption];
      if (!envDataItem || !envDataItem.current) {
        return [
          { name: '温度', icon: '🌡️', status: 'good' },
          { name: '湿度', icon: '💧', status: 'good' },
          { name: 'PH值', icon: '⚗️', status: 'good' }
        ];
      }
      
      const data = envDataItem.current;
      return [
        { name: '温度', icon: '🌡️', status: data[0] >= 20 && data[0] <= 28 ? 'good' : 'bad' },
        { name: '湿度', icon: '💧', status: data[2] >= 40 && data[2] <= 70 ? 'good' : 'bad' },
        { name: 'PH值', icon: '⚗️', status: data[4] >= 6.0 && data[4] <= 7.5 ? 'good' : 'bad' }
      ];
    },
    
    initCharts() {
      if (!this.$refs.envChart || !this.$refs.radarChart || !this.$refs.deviceChart) {
        console.warn('图表容器未找到');
        return;
      }
      
      this.envChart = echarts.init(this.$refs.envChart);
      this.radarChart = echarts.init(this.$refs.radarChart);
      this.deviceChart = echarts.init(this.$refs.deviceChart);
      // 总是调用updateCharts，内部会处理没有数据的情况
      this.updateCharts();
      
      window.addEventListener('resize', () => {
        this.envChart && this.envChart.resize();
        this.radarChart && this.radarChart.resize();
        this.deviceChart && this.deviceChart.resize();
      });
    },
    
    updateCharts() {
      this.updateEnvChart();
      this.updateRadarChart();
      this.updateDeviceChart();
    },
    
    updateEnvChart() {
      if (!this.envChart) return;
      
      const data = this.envData[this.selectedOption];
      
      if (!data) {
        // 使用默认数据
        const defaultData = {
          temp: [22, 24, 23, 25, 24, 26, 25],
          airHum: [65, 68, 70, 67, 69, 71, 68],
          soilHum: [45, 48, 50, 47, 49, 51, 48],
          carbon: [400, 420, 410, 430, 415, 425, 420],
          ph: [6.5, 6.8, 6.7, 6.9, 6.6, 7.0, 6.8],
          light: [2200, 2400, 2300, 2500, 2350, 2450, 2400]
        };
        const option = this.buildEnvChartOption(defaultData, this.chartType);
        this.envChart.setOption(option);
        return;
      }
      
      const option = this.buildEnvChartOption(data, this.chartType);
      this.envChart.setOption(option);
    },
    
    buildEnvChartOption(data, chartType) {
      return {
        title: {
          text: '环境数据趋势分析',
          left: 'center',
          textStyle: {
            fontSize: 16,
            fontWeight: 'bold'
          }
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross'
          }
        },
        legend: {
          data: ['温度', '空气湿度', '土壤湿度', 'CO₂', 'PH值', '光照'],
          bottom: 10
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '15%',
          top: '15%',
          containLabel: true
        },
        xAxis: {
          type: 'category',
          boundaryGap: chartType === 'bar',
          data: ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
        },
        yAxis: {
          type: 'value'
        },
        series: [
          {
            name: '温度',
            type: chartType,
            data: data.temp,
            smooth: true,
            itemStyle: { color: '#E6A23C' }
          },
          {
            name: '空气湿度',
            type: chartType,
            data: data.airHum,
            smooth: true,
            itemStyle: { color: '#409EFF' }
          },
          {
            name: '土壤湿度',
            type: chartType,
            data: data.soilHum,
            smooth: true,
            itemStyle: { color: '#67C23A' }
          },
          {
            name: 'CO₂',
            type: chartType,
            data: data.carbon.map(v => v / 10),
            smooth: true,
            itemStyle: { color: '#909399' }
          },
          {
            name: 'PH值',
            type: chartType,
            data: data.ph,
            smooth: true,
            itemStyle: { color: '#F56C6C' }
          },
          {
            name: '光照',
            type: chartType,
            data: data.light.map(v => v / 100),
            smooth: true,
            itemStyle: { color: '#E6A23C' }
          }
        ]
      };
    },
    
    updateRadarChart() {
      if (!this.radarChart) return;
      
      const data = this.envData[this.selectedOption];
      
      const currentValue = (data && data.current) ? data.current : [26, 70, 50, 1500, 7.0, 2100];
      
      const option = {
        tooltip: {
          trigger: 'item'
        },
        legend: {
          data: ['当前数值', '标准数值'],
          bottom: 5
        },
        radar: {
          indicator: [
            { name: '温度', max: 60 },
            { name: '空气湿度', max: 100 },
            { name: '土壤湿度', max: 100 },
            { name: 'CO₂', max: 3000 },
            { name: 'PH值', max: 14 },
            { name: '光照', max: 4000 }
          ],
          shape: 'polygon',
          splitNumber: 4
        },
        series: [{
          type: 'radar',
          data: [
            {
              value: currentValue,
              name: '当前数值',
              areaStyle: {
                color: 'rgba(64, 158, 255, 0.3)'
              }
            },
            {
              value: [26, 70, 50, 1500, 7.0, 2100],
              name: '标准数值',
              areaStyle: {
                color: 'rgba(103, 194, 58, 0.3)'
              }
            }
          ]
        }]
      };
      
      this.radarChart.setOption(option);
    },
    
    updateDeviceChart() {
      const option = {
        tooltip: {
          trigger: 'item',
          formatter: '{b}: {c}台 ({d}%)'
        },
        series: [{
          type: 'pie',
          radius: ['40%', '70%'],
          avoidLabelOverlap: false,
          itemStyle: {
            borderRadius: 10,
            borderColor: '#fff',
            borderWidth: 2
          },
          label: {
            show: false
          },
          emphasis: {
            label: {
              show: true,
              fontSize: 14,
              fontWeight: 'bold'
            }
          },
          data: [
            { value: 20, name: '采集设备', itemStyle: { color: '#667eea' } },
            { value: 10, name: '控制设备', itemStyle: { color: '#f093fb' } },
            { value: 4, name: '气象站', itemStyle: { color: '#4facfe' } },
            { value: 4, name: '监控设备', itemStyle: { color: '#43e97b' } }
          ]
        }]
      };
      
      this.deviceChart.setOption(option);
    },
    
    play() {
      this.$refs.videoPlayer.play();
    },
    
    pause() {
      this.$refs.videoPlayer.pause();
    }
  }
};
</script>

<style scoped>
.dashboard-container {
  width: 100%;
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #e8eef5 100%);
  padding: 20px;
  position: relative;
  overflow: hidden;
}

/* 顶部标题栏 */
.dashboard-header {
  position: relative;
  z-index: 10;
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: linear-gradient(135deg, rgba(64, 158, 255, 0.1) 0%, rgba(100, 181, 246, 0.05) 100%);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(64, 158, 255, 0.2);
  padding: 20px 30px;
  border-radius: 16px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 12px;
}

.header-left i {
  font-size: 32px;
  color: #409EFF;
  filter: drop-shadow(0 0 10px rgba(64, 158, 255, 0.6));
}

.header-title {
  font-size: 26px;
  font-weight: bold;
  color: #fff;
  text-shadow: 0 2px 10px rgba(64, 158, 255, 0.3);
  background: linear-gradient(135deg, #fff 0%, #64B5F6 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.farm-selector {
  width: 200px;
}

/* 健康评分卡片 */
.health-score-card {
  position: relative;
  z-index: 10;
  display: flex;
  align-items: center;
  gap: 30px;
  background: linear-gradient(135deg, rgba(64, 158, 255, 0.1) 0%, rgba(100, 181, 246, 0.05) 100%);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(64, 158, 255, 0.2);
  padding: 25px 30px;
  border-radius: 16px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

.score-circle {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
  animation: pulse 2s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

.score-inner {
  width: 100px;
  height: 100px;
  background: white;
  border-radius: 50%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.score-number {
  font-size: 36px;
  font-weight: bold;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.score-label {
  font-size: 12px;
  color: #909399;
  margin-top: 4px;
}

.score-right {
  flex: 1;
}

.score-status {
  display: inline-block;
  padding: 8px 20px;
  border-radius: 20px;
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 10px;
}

.score-status.excellent {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.score-status.good {
  background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%);
  color: white;
}

.score-status.normal {
  background: linear-gradient(135deg, #fa709a 0%, #fee140 100%);
  color: white;
}

.score-status.poor {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  color: white;
}

.score-desc {
  font-size: 14px;
  color: #606266;
  margin-bottom: 15px;
}

.score-metrics {
  display: flex;
  gap: 10px;
}

.metric-tag {
  padding: 6px 12px;
  border-radius: 12px;
  font-size: 13px;
  font-weight: 500;
}

.metric-tag.good {
  background: rgba(103, 194, 58, 0.1);
  color: #67C23A;
}

.metric-tag.bad {
  background: rgba(245, 108, 108, 0.1);
  color: #F56C6C;
}

/* 主要内容区域 */
.dashboard-content {
  display: flex;
  gap: 20px;
}

.content-left {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.content-right {
  width: 420px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

/* 卡片样式 */
.card {
  background: white;
  border-radius: 16px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  overflow: hidden;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 25px;
  border-bottom: 1px solid #f0f0f0;
}

.card-title {
  font-size: 16px;
  font-weight: bold;
  color: #303133;
}

.update-time {
  font-size: 12px;
  color: #909399;
}

.card-body {
  padding: 20px;
}

/* 图表容器 */
.chart-container {
  width: 100%;
  height: 350px;
}

.chart-container-small {
  width: 100%;
  height: 280px;
}

.env-trend-card {
  flex: 1;
}

/* 设备统计 */
.device-stats {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-top: 15px;
}

.device-stat-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  background: #f5f7fa;
  border-radius: 12px;
  transition: all 0.3s;
}

.device-stat-item:hover {
  transform: translateX(5px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.device-icon {
  width: 40px;
  height: 40px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
}

.device-info {
  flex: 1;
}

.device-name {
  font-size: 14px;
  font-weight: bold;
  color: #303133;
}

.device-count {
  font-size: 12px;
  color: #909399;
  margin-top: 2px;
}

.device-status {
  padding: 4px 12px;
  border-radius: 10px;
  font-size: 12px;
  font-weight: bold;
}

.device-status.online {
  background: rgba(103, 194, 58, 0.1);
  color: #67C23A;
}

.device-status.warning {
  background: rgba(230, 162, 60, 0.1);
  color: #E6A23C;
}

/* 视频监控 */
.video-body {
  padding: 0 !important;
  position: relative;
}

.video-player {
  width: 100%;
  height: 280px;
  object-fit: cover;
  display: block;
}

.video-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(to top, rgba(0, 0, 0, 0.7), transparent);
  padding: 15px 20px;
}

.video-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: white;
}

.video-location {
  font-size: 14px;
  font-weight: bold;
}

.video-time {
  font-size: 13px;
  font-family: 'Courier New', monospace;
}

.video-controls {
  display: flex;
  gap: 10px;
}

.video-controls i {
  font-size: 18px;
  cursor: pointer;
  color: #606266;
  transition: color 0.3s;
}

.video-controls i:hover {
  color: #409EFF;
}
</style>
