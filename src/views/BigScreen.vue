<template>
  <div class="big-screen-container">
    <!-- 顶部信息栏 -->
    <div class="top-info-bar">
      <div class="info-item">
        <i class="el-icon-time"></i>
        <span>{{ currentTime }}</span>
      </div>
      <div class="info-item">
        <i class="el-icon-sunny"></i>
        <span>{{ weather.temp }}°C {{ weather.status }}</span>
      </div>
    </div>

    <!-- 主内容区 -->
    <div class="main-content">
      <!-- 左侧：农场地图 -->
      <div class="left-section">
        <div class="section-title">
          <i class="el-icon-location"></i>
          农场分区总览
        </div>
        <div class="map-container">
          <div class="farm-map">
            <!-- 地块信息卡片 -->
            <div 
              v-for="(area, index) in areas" 
              :key="index"
              class="area-card"
              :style="{ top: area.top, left: area.left }"
              @click="selectArea(area)"
              :class="{ active: selectedArea.name === area.name }"
            >
              <div class="area-name">{{ area.name }}</div>
              <div class="area-info">
                <div>面积: {{ area.size }}亩</div>
                <div>存栏: {{ area.stock }}尾</div>
                <div>状态: <span :class="area.status === '正常' ? 'status-normal' : 'status-warning'">{{ area.status }}</span></div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 右侧：详细信息 -->
      <div class="right-section">
        <!-- 选中区域详情 -->
        <div class="detail-card">
          <div class="section-title">
            <i class="el-icon-view"></i>
            区域详情
          </div>
          <div class="detail-content">
            <div class="preview-image"></div>
            <div class="detail-info">
              <h3>{{ selectedArea.name || '请选择区域' }}</h3>
              <p>{{ selectedArea.description || '点击地图上的区域查看详情' }}</p>
              <div v-if="selectedArea.name" class="detail-stats">
                <el-row :gutter="10">
                  <el-col :span="12">
                    <div class="stat-box">
                      <div class="stat-label">面积</div>
                      <div class="stat-value">{{ selectedArea.size }}亩</div>
                    </div>
                  </el-col>
                  <el-col :span="12">
                    <div class="stat-box">
                      <div class="stat-label">存栏</div>
                      <div class="stat-value">{{ selectedArea.stock }}尾</div>
                    </div>
                  </el-col>
                </el-row>
              </div>
            </div>
          </div>
        </div>

        <!-- 实时监控 -->
        <div class="monitor-card">
          <div class="section-title">
            <i class="el-icon-data-line"></i>
            实时监控
          </div>
          <div class="gauge-chart" ref="gaugeChart"></div>
        </div>
      </div>
    </div>

    <!-- 底部统计区 -->
    <div class="bottom-stats">
      <!-- 生产指标 -->
      <div class="stat-panel">
        <div class="section-title">
          <i class="el-icon-s-data"></i>
          生产指标概览
        </div>
        <el-row :gutter="15" style="margin-top: 15px;">
          <el-col :span="8">
            <div class="stat-card">
              <div class="stat-icon" style="background: #409EFF;">
                <i class="el-icon-s-grid"></i>
              </div>
              <div class="stat-content">
                <div class="stat-name">养殖总面积</div>
                <div class="stat-num">{{ summary.totalArea }}<span class="unit">亩</span></div>
              </div>
            </div>
          </el-col>
          <el-col :span="8">
            <div class="stat-card">
              <div class="stat-icon" style="background: #67C23A;">
                <i class="el-icon-s-goods"></i>
              </div>
              <div class="stat-content">
                <div class="stat-name">预计产量</div>
                <div class="stat-num">{{ summary.expectedOutput }}<span class="unit">吨</span></div>
              </div>
            </div>
          </el-col>
          <el-col :span="8">
            <div class="stat-card">
              <div class="stat-icon" style="background: #E6A23C;">
                <i class="el-icon-s-data"></i>
              </div>
              <div class="stat-content">
                <div class="stat-name">当前存栏</div>
                <div class="stat-num">{{ summary.currentStock }}<span class="unit">万尾</span></div>
              </div>
            </div>
          </el-col>
        </el-row>
      </div>

      <!-- 月度产量图表 -->
      <div class="chart-panel">
        <div class="section-title">
          <i class="el-icon-s-marketing"></i>
          月度产量统计
        </div>
        <div class="bar-chart" ref="barChart"></div>
      </div>

      <!-- 告警列表 -->
      <div class="alert-panel">
        <div class="section-title">
          <i class="el-icon-warning"></i>
          近期告警
        </div>
        <div class="alert-list">
          <div 
            v-for="(alert, index) in alerts" 
            :key="index"
            class="alert-item"
            :class="'alert-' + alert.level"
          >
            <div class="alert-time">{{ alert.time }}</div>
            <div class="alert-content">{{ alert.content }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as echarts from 'echarts';

export default {
  name: 'BigScreen',
  data() {
    return {
      currentTime: '',
      selectedArea: {},
      
      // 天气信息
      weather: {
        temp: 28,
        status: '晴'
      },
      
      // 总体统计数据
      summary: {
        totalArea: 1250,
        expectedOutput: 156,
        currentStock: 8.5,
        waterUsage: 49
      },
      
      // 地块信息
      areas: [
        {
          name: '清水虾区A',
          size: 320,
          stock: 12500,
          status: '正常',
          top: '15%',
          left: '15%',
          description: '主要养殖南美白对虾，采用清水养殖模式，配备自动投喂和水质监测系统。'
        },
        {
          name: '清水虾区B',
          size: 280,
          stock: 11200,
          status: '正常',
          top: '15%',
          left: '55%',
          description: '南美白对虾养殖区，水质优良，溶氧充足。'
        },
        {
          name: '笼养区',
          size: 180,
          stock: 8600,
          status: '告警',
          top: '50%',
          left: '20%',
          description: '采用网箱笼养模式，适合高密度养殖，需注意水质监控。'
        },
        {
          name: '混养区',
          size: 220,
          stock: 9800,
          status: '正常',
          top: '50%',
          left: '60%',
          description: '虾蟹混养区域，生态平衡良好，产量稳定。'
        },
        {
          name: '育苗区',
          size: 150,
          stock: 15000,
          status: '正常',
          top: '75%',
          left: '40%',
          description: '虾苗培育专区，配备恒温设备和专业育苗池。'
        }
      ],
      
      // 告警列表
      alerts: [
        { time: '14:32', content: '笼养区溶氧偏低，请及时处理', level: 'warning' },
        { time: '12:15', content: '清水虾区A投喂完成', level: 'info' },
        { time: '10:48', content: '3号增氧机离线', level: 'error' },
        { time: '09:20', content: '水质检测正常', level: 'success' },
        { time: '08:00', content: '系统自检完成', level: 'info' }
      ],
      
      // 图表数据
      chartData: {
        months: ['1月', '2月', '3月', '4月', '5月', '6月', '7月', '8月', '9月', '10月', '11月', '12月'],
        output: [85, 92, 101, 98, 115, 128, 142, 138, 125, 132, 145, 156]
      }
    };
  },
  
  mounted() {
    this.updateTime();
    this.timeInterval = setInterval(this.updateTime, 1000);
    
    // 初始化图表
    this.$nextTick(() => {
      this.initBarChart();
      this.initGaugeChart();
    });
  },
  
  beforeDestroy() {
    if (this.timeInterval) {
      clearInterval(this.timeInterval);
    }
    // 销毁图表
    if (this.barChartInstance) {
      this.barChartInstance.dispose();
    }
    if (this.gaugeChartInstance) {
      this.gaugeChartInstance.dispose();
    }
  },
  
  methods: {
    // 更新时间
    updateTime() {
      const now = new Date();
      const year = now.getFullYear();
      const month = String(now.getMonth() + 1).padStart(2, '0');
      const day = String(now.getDate()).padStart(2, '0');
      const hour = String(now.getHours()).padStart(2, '0');
      const minute = String(now.getMinutes()).padStart(2, '0');
      const second = String(now.getSeconds()).padStart(2, '0');
      const weekDays = ['星期日', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六'];
      const weekDay = weekDays[now.getDay()];
      
      this.currentTime = `${year}-${month}-${day} ${hour}:${minute}:${second} ${weekDay}`;
    },
    
    // 选择区域
    selectArea(area) {
      this.selectedArea = area;
    },
    
    // 初始化柱状图
    initBarChart() {
      if (!this.$refs.barChart) return;
      
      this.barChartInstance = echarts.init(this.$refs.barChart);
      
      const option = {
        grid: {
          top: '15%',
          left: '12%',
          right: '8%',
          bottom: '15%'
        },
        xAxis: {
          type: 'category',
          data: this.chartData.months,
          axisLine: {
            lineStyle: { color: '#ddd' }
          },
          axisLabel: {
            color: '#666'
          }
        },
        yAxis: {
          type: 'value',
          name: '产量(吨)',
          nameTextStyle: {
            color: '#666'
          },
          axisLine: {
            lineStyle: { color: '#ddd' }
          },
          axisLabel: {
            color: '#666'
          },
          splitLine: {
            lineStyle: {
              color: '#eee',
              type: 'dashed'
            }
          }
        },
        series: [{
          data: this.chartData.output,
          type: 'bar',
          barWidth: '50%',
          itemStyle: {
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
              { offset: 0, color: '#409EFF' },
              { offset: 1, color: '#66b1ff' }
            ]),
            borderRadius: [4, 4, 0, 0]
          },
          label: {
            show: true,
            position: 'top',
            color: '#666',
            fontSize: 11
          }
        }]
      };
      
      this.barChartInstance.setOption(option);
      
      // 自适应
      window.addEventListener('resize', () => {
        this.barChartInstance && this.barChartInstance.resize();
      });
    },
    
    // 初始化仪表盘
    initGaugeChart() {
      if (!this.$refs.gaugeChart) return;
      
      this.gaugeChartInstance = echarts.init(this.$refs.gaugeChart);
      
      const option = {
        series: [{
          type: 'gauge',
          startAngle: 180,
          endAngle: 0,
          min: 0,
          max: 100,
          radius: '90%',
          center: ['50%', '70%'],
          splitNumber: 10,
          axisLine: {
            lineStyle: {
              width: 12,
              color: [
                [0.3, '#67C23A'],
                [0.7, '#E6A23C'],
                [1, '#F56C6C']
              ]
            }
          },
          pointer: {
            icon: 'path://M2090.36389,615.30999 L2090.36389,615.30999 C2091.48372,615.30999 2092.40383,616.194028 2092.44859,617.312956 L2096.90698,728.755929 C2097.05155,732.369577 2094.2393,735.416212 2090.62566,735.56078 C2090.53845,735.564269 2090.45117,735.566014 2090.36389,735.566014 L2090.36389,735.566014 C2086.74736,735.566014 2083.81557,732.63423 2083.81557,729.017692 C2083.81557,728.930412 2083.81732,728.84314 2083.82081,728.755929 L2088.2792,617.312956 C2088.32396,616.194028 2089.24407,615.30999 2090.36389,615.30999 Z',
            length: '75%',
            width: 8,
            offsetCenter: [0, '5%'],
            itemStyle: {
              color: '#409EFF'
            }
          },
          axisTick: {
            length: 8,
            lineStyle: {
              color: 'auto',
              width: 1
            }
          },
          splitLine: {
            length: 12,
            lineStyle: {
              color: 'auto',
              width: 2
            }
          },
          axisLabel: {
            color: '#666',
            fontSize: 10,
            distance: -40
          },
          title: {
            offsetCenter: [0, '-20%'],
            fontSize: 14,
            color: '#666'
          },
          detail: {
            fontSize: 24,
            offsetCenter: [0, '0%'],
            valueAnimation: true,
            formatter: '{value}%',
            color: '#409EFF'
          },
          data: [{
            value: this.summary.waterUsage,
            name: '用水率'
          }]
        }]
      };
      
      this.gaugeChartInstance.setOption(option);
      
      // 自适应
      window.addEventListener('resize', () => {
        this.gaugeChartInstance && this.gaugeChartInstance.resize();
      });
    }
  }
};
</script>

<style scoped>
.big-screen-container {
  padding: 20px;
  background: #f5f7fa;
  min-height: calc(100vh - 60px);
}

/* 顶部信息栏 */
.top-info-bar {
  display: flex;
  justify-content: flex-end;
  gap: 30px;
  margin-bottom: 20px;
  padding: 12px 20px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.info-item {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #606266;
  font-size: 14px;
}

.info-item i {
  font-size: 18px;
  color: #409EFF;
}

/* 主内容区 */
.main-content {
  display: flex;
  gap: 20px;
  margin-bottom: 20px;
}

.left-section {
  flex: 2;
  background: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.right-section {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.detail-card,
.monitor-card {
  background: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.detail-card {
  flex: 1;
}

.section-title {
  font-size: 16px;
  font-weight: bold;
  color: #303133;
  margin-bottom: 15px;
  padding-bottom: 10px;
  border-bottom: 2px solid #409EFF;
  display: flex;
  align-items: center;
  gap: 8px;
}

.section-title i {
  color: #409EFF;
  font-size: 18px;
}

/* 农场地图 */
.map-container {
  height: 500px;
  position: relative;
}

.farm-map {
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 100%);
  border-radius: 8px;
  position: relative;
  overflow: hidden;
}

.area-card {
  position: absolute;
  width: 180px;
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid #409EFF;
  border-radius: 8px;
  padding: 12px;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0 2px 12px rgba(64, 158, 255, 0.3);
}

.area-card:hover,
.area-card.active {
  transform: scale(1.05);
  box-shadow: 0 4px 20px rgba(64, 158, 255, 0.5);
  border-color: #66b1ff;
  z-index: 10;
}

.area-name {
  font-size: 15px;
  font-weight: bold;
  color: #409EFF;
  margin-bottom: 8px;
  border-bottom: 1px solid #e4e7ed;
  padding-bottom: 6px;
}

.area-info {
  font-size: 12px;
  color: #606266;
  line-height: 1.8;
}

.status-normal {
  color: #67C23A;
  font-weight: bold;
}

.status-warning {
  color: #E6A23C;
  font-weight: bold;
}

/* 区域详情 */
.detail-content {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.preview-image {
  width: 100%;
  height: 160px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
}

.preview-image::before {
  content: '📹';
  font-size: 48px;
  opacity: 0.5;
}

.detail-info h3 {
  color: #303133;
  margin: 0 0 10px;
  font-size: 16px;
}

.detail-info p {
  color: #606266;
  font-size: 13px;
  line-height: 1.6;
  margin: 0;
}

.detail-stats {
  margin-top: 15px;
}

.stat-box {
  background: #f5f7fa;
  padding: 12px;
  border-radius: 6px;
  text-align: center;
}

.stat-label {
  font-size: 12px;
  color: #909399;
  margin-bottom: 5px;
}

.stat-value {
  font-size: 20px;
  font-weight: bold;
  color: #409EFF;
}

/* 仪表盘 */
.gauge-chart {
  width: 100%;
  height: 220px;
}

/* 底部统计区 */
.bottom-stats {
  display: flex;
  gap: 20px;
}

.stat-panel,
.chart-panel,
.alert-panel {
  flex: 1;
  background: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

/* 生产指标卡片 */
.stat-card {
  display: flex;
  align-items: center;
  gap: 15px;
  background: #f5f7fa;
  padding: 15px;
  border-radius: 8px;
  transition: all 0.3s;
}

.stat-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.stat-icon {
  width: 50px;
  height: 50px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  color: white;
  flex-shrink: 0;
}

.stat-content {
  flex: 1;
}

.stat-name {
  font-size: 13px;
  color: #909399;
  margin-bottom: 5px;
}

.stat-num {
  font-size: 22px;
  font-weight: bold;
  color: #303133;
}

.stat-num .unit {
  font-size: 14px;
  color: #909399;
  margin-left: 4px;
}

/* 柱状图 */
.bar-chart {
  width: 100%;
  height: 250px;
  margin-top: 10px;
}

/* 告警列表 */
.alert-list {
  max-height: 250px;
  overflow-y: auto;
  margin-top: 10px;
}

.alert-item {
  padding: 12px;
  margin-bottom: 10px;
  background: #f5f7fa;
  border-left: 3px solid #409EFF;
  border-radius: 4px;
  font-size: 13px;
  transition: all 0.3s;
}

.alert-item:hover {
  background: #ecf5ff;
}

.alert-item.alert-warning {
  border-left-color: #E6A23C;
}

.alert-item.alert-error {
  border-left-color: #F56C6C;
}

.alert-item.alert-success {
  border-left-color: #67C23A;
}

.alert-item.alert-info {
  border-left-color: #409EFF;
}

.alert-time {
  color: #909399;
  margin-bottom: 5px;
  font-size: 12px;
}

.alert-content {
  color: #606266;
  line-height: 1.5;
}

/* 滚动条样式 */
.alert-list::-webkit-scrollbar {
  width: 6px;
}

.alert-list::-webkit-scrollbar-thumb {
  background: #dcdfe6;
  border-radius: 3px;
}

.alert-list::-webkit-scrollbar-track {
  background: #f5f7fa;
}
</style>
