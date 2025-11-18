<template>
  <div class="big-screen-container">
    <!-- 顶部信息栏 -->
    <div class="top-info-bar">
      <div class="title">
        <i class="el-icon-s-data"></i>
        智慧农场可视化大屏
      </div>
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
      <!-- 左侧：统计概览 -->
      <div class="left-section">
        <div class="stat-card-group">
          <div class="stat-card">
            <div class="stat-icon" style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);">
              <i class="el-icon-s-grid"></i>
            </div>
            <div class="stat-content">
              <div class="stat-label">农田总数</div>
              <div class="stat-value">{{ summary.farmCount }}<span class="unit">块</span></div>
            </div>
          </div>

          <div class="stat-card">
            <div class="stat-icon" style="background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);">
              <i class="el-icon-s-data"></i>
            </div>
            <div class="stat-content">
              <div class="stat-label">总面积</div>
              <div class="stat-value">{{ summary.totalArea }}<span class="unit">亩</span></div>
            </div>
          </div>

          <div class="stat-card">
            <div class="stat-icon" style="background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);">
              <i class="el-icon-s-goods"></i>
            </div>
            <div class="stat-content">
              <div class="stat-label">作物总量</div>
              <div class="stat-value">{{ summary.totalStock }}<span class="unit">株</span></div>
            </div>
          </div>

          <div class="stat-card">
            <div class="stat-icon" style="background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%);">
              <i class="el-icon-success"></i>
            </div>
            <div class="stat-content">
              <div class="stat-label">正常运行</div>
              <div class="stat-value">{{ summary.normalCount }}<span class="unit">块</span></div>
            </div>
          </div>
        </div>

        <!-- 农田列表 -->
        <div class="farm-list-panel">
          <div class="panel-title">
            <i class="el-icon-menu"></i>
            农田列表
          </div>
          <div class="farm-list">
            <div 
              v-for="(farm, index) in farms" 
              :key="index"
              class="farm-item"
              :class="{ active: selectedFarm && selectedFarm.id === farm.id }"
              @click="selectFarm(farm)"
            >
              <div class="farm-header">
                <span class="farm-name">{{ farm.farm }}</span>
                <el-tag :type="farm.state === '正常' ? 'success' : 'warning'" size="mini">
                  {{ farm.state }}
                </el-tag>
              </div>
              <div class="farm-info">
                <div class="info-item">
                  <i class="el-icon-location"></i>
                  {{ farm.address }}
                </div>
                <div class="info-row">
                  <span>🌾 {{ farm.crop }}</span>
                  <span>📏 {{ farm.area }}亩</span>
                  <span>🌱 {{ farm.number }}株</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 中间：3D 可视化 -->
      <div class="center-section">
        <div class="panel-title">
          <i class="el-icon-view"></i>
          农田 3D 可视化
        </div>
        <div class="chart-3d" ref="chart3d"></div>
      </div>

      <!-- 右侧：详细信息 -->
      <div class="right-section">
        <!-- 选中农田详情 -->
        <div class="detail-panel">
          <div class="panel-title">
            <i class="el-icon-document"></i>
            农田详情
          </div>
          <div v-if="selectedFarm" class="detail-content">
            <div class="detail-header">
              <h3>{{ selectedFarm.farm }}</h3>
              <el-tag :type="selectedFarm.state === '正常' ? 'success' : 'warning'">
                {{ selectedFarm.state }}
              </el-tag>
            </div>
            
            <div class="detail-grid">
              <div class="detail-item">
                <div class="detail-label">作物类型</div>
                <div class="detail-value">{{ selectedFarm.crop }}</div>
              </div>
              <div class="detail-item">
                <div class="detail-label">种植面积</div>
                <div class="detail-value">{{ selectedFarm.area }} 亩</div>
              </div>
              <div class="detail-item">
                <div class="detail-label">作物数量</div>
                <div class="detail-value">{{ selectedFarm.number }} 株</div>
              </div>
              <div class="detail-item">
                <div class="detail-label">所在位置</div>
                <div class="detail-value">{{ selectedFarm.address }}</div>
              </div>
              <div class="detail-item">
                <div class="detail-label">负责人</div>
                <div class="detail-value">{{ selectedFarm.keeper }}</div>
              </div>
            </div>

            <div class="env-monitor">
              <div class="monitor-title">环境监测数据</div>
              <el-row :gutter="10">
                <el-col :span="12">
                  <div class="monitor-item">
                    <i class="el-icon-sunny" style="color: #E6A23C;"></i>
                    <span class="monitor-label">温度</span>
                    <span class="monitor-value">{{ selectedFarm.temperature }}°C</span>
                  </div>
                </el-col>
                <el-col :span="12">
                  <div class="monitor-item">
                    <i class="el-icon-water-cup" style="color: #409EFF;"></i>
                    <span class="monitor-label">空气湿度</span>
                    <span class="monitor-value">{{ selectedFarm.airhumidity }}%</span>
                  </div>
                </el-col>
                <el-col :span="12">
                  <div class="monitor-item">
                    <i class="el-icon-s-opportunity" style="color: #67C23A;"></i>
                    <span class="monitor-label">土壤湿度</span>
                    <span class="monitor-value">{{ selectedFarm.soilhumidity }}%</span>
                  </div>
                </el-col>
                <el-col :span="12">
                  <div class="monitor-item">
                    <i class="el-icon-light" style="color: #F56C6C;"></i>
                    <span class="monitor-label">光照强度</span>
                    <span class="monitor-value">{{ selectedFarm.light }}lux</span>
                  </div>
                </el-col>
                <el-col :span="12">
                  <div class="monitor-item">
                    <i class="el-icon-s-marketing" style="color: #909399;"></i>
                    <span class="monitor-label">CO₂含量</span>
                    <span class="monitor-value">{{ selectedFarm.carbon }}ppm</span>
                  </div>
                </el-col>
                <el-col :span="12">
                  <div class="monitor-item">
                    <i class="el-icon-s-tools" style="color: #E6A23C;"></i>
                    <span class="monitor-label">土壤PH值</span>
                    <span class="monitor-value">{{ selectedFarm.ph }}</span>
                  </div>
                </el-col>
              </el-row>
            </div>

            <!-- 健康指数显示 -->
            <div class="health-index-section" v-if="selectedFarm">
              <div class="monitor-title">健康指数</div>
              <div class="health-index-display">
                <div class="health-score" :style="{ background: getHealthScoreGradient(selectedFarm) }">
                  <div class="score-value">{{ calculateHealthScoreLocal(selectedFarm) }}</div>
                  <div class="score-label">健康指数</div>
                </div>
                <div class="health-level">
                  <el-tag :type="getHealthLevelTag(selectedFarm)" size="medium">
                    {{ getHealthLevelText(selectedFarm) }}
                  </el-tag>
                </div>
              </div>
            </div>

            <div class="device-status">
              <div class="monitor-title">设备状态</div>
              <div class="device-list">
                <div class="device-item">
                  <span>💡 补光灯</span>
                  <el-tag :type="selectedFarm.filllight === '开启' ? 'success' : 'info'" size="small">
                    {{ selectedFarm.filllight }}
                  </el-tag>
                </div>
                <div class="device-item">
                  <span>📹 摄像头</span>
                  <el-tag :type="selectedFarm.monitor === '开启' ? 'success' : 'info'" size="small">
                    {{ selectedFarm.monitor }}
                  </el-tag>
                </div>
                <div class="device-item">
                  <span>💧 水泵</span>
                  <el-tag :type="selectedFarm.pump === '开启' ? 'success' : 'info'" size="small">
                    {{ selectedFarm.pump }}
                  </el-tag>
                </div>
              </div>
            </div>
          </div>
          <div v-else class="no-selection">
            <i class="el-icon-info"></i>
            <p>请点击左侧农田列表查看详情</p>
          </div>
        </div>

        <!-- 今日任务清单 -->
        <div class="task-panel">
          <TodayTaskList />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as echarts from 'echarts';
import 'echarts-gl';
import TodayTaskList from '@/components/TodayTaskList.vue';

export default {
  name: 'BigScreen3D',
  components: {
    TodayTaskList
  },
  data() {
    return {
      currentTime: '',
      selectedFarm: null,
      // 天气信息
      weather: {
        temp: 28,
        status: '晴'
      },
      // 统计数据
      summary: {
        farmCount: 0,
        totalArea: 0,
        totalStock: 0,
        normalCount: 0
      },
      // 农田数据
      farms: []
    };
  },
  mounted() {
    this.updateTime();
    setInterval(this.updateTime, 1000);
    this.fetchData();
  },
  methods: {
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
    
    async fetchData() {
      try {
        const farmsRes = await this.request.get('/statistic/dashboard');
        if (farmsRes.code === '200') {
          this.farms = farmsRes.data;
          if (this.farms.length > 0) {
            this.selectedFarm = this.farms[0];
          }
          
          // 数据加载完成后初始化3D图表
          this.$nextTick(() => {
            this.init3DChart();
          });
        }
        
        const summaryRes = await this.request.get('/statistic/dashboard/summary');
        if (summaryRes.code === '200') {
          this.summary = summaryRes.data;
        }
      } catch (error) {
        console.error('获取数据失败:', error);
      }
    },
    
    selectFarm(farm) {
      this.selectedFarm = farm;
    },
    
    init3DChart() {
      if (!this.$refs.chart3d) {
        console.error('图表容器未找到');
        return;
      }
      
      if (!this.farms || this.farms.length === 0) {
        console.warn('没有农田数据');
        return;
      }
      
      console.log('初始化3D图表，农田数量:', this.farms.length);
      
      const chart = echarts.init(this.$refs.chart3d);
      
      // 准备3D柱状图数据 - 每个农田一个柱子
      const barData = [];
      this.farms.forEach((farm, index) => {
        const x = index % 5;
        const y = Math.floor(index / 5);
        const z = parseFloat(farm.area) || 5;
        
        barData.push({
          value: [x, y, z],
          itemStyle: {
            color: this.getCropColor(farm.crop)
          },
          label: {
            show: true,
            formatter: farm.farm + '\n' + farm.area + '亩\n' + farm.crop
          },
          farmData: farm
        });
      });
      
      console.log('3D柱状图数据:', barData);
      
      const option = {
        tooltip: {
          formatter: function(params) {
            if (params.data && params.data.farmData) {
              const f = params.data.farmData;
              return f.farm + '<br/>面积: ' + f.area + '亩<br/>作物: ' + f.crop + '<br/>数量: ' + f.number + '株';
            }
            return '';
          }
        },
        xAxis3D: {
          type: 'value'
        },
        yAxis3D: {
          type: 'value'
        },
        zAxis3D: {
          type: 'value',
          name: '面积(亩)'
        },
        grid3D: {
          boxWidth: 100,
          boxDepth: 80,
          viewControl: {
            autoRotate: true,
            autoRotateSpeed: 10
          }
        },
        series: [{
          type: 'bar3D',
          data: barData,
          shading: 'lambert',
          label: {
            textStyle: {
              fontSize: 12,
              borderWidth: 1,
              borderColor: '#fff',
              backgroundColor: 'rgba(0,0,0,0.6)',
              color: '#fff',
              padding: 3
            }
          },
          itemStyle: {
            opacity: 0.8
          },
          emphasis: {
            label: {
              textStyle: {
                fontSize: 14,
                backgroundColor: 'rgba(64,158,255,0.8)'
              }
            },
            itemStyle: {
              opacity: 1
            }
          }
        }]
      };
      
      chart.setOption(option);
      console.log('3D图表配置完成');
      
      // 点击事件
      chart.on('click', (params) => {
        if (params.data && params.data.farmData) {
          this.selectFarm(params.data.farmData);
        }
      });
      
      window.addEventListener('resize', () => {
        chart.resize();
      });
    },
    
    getCropColor(crop) {
      const colors = {
        '小麦': '#F4D03F',
        '玉米': '#F39C12',
        '水稻': '#52BE80',
        '大豆': '#85C1E2',
        '番茄': '#E74C3C',
        '黄瓜': '#27AE60',
        '辣椒': '#C0392B',
        '茄子': '#8E44AD',
        '白菜': '#2ECC71',
        '萝卜': '#E67E22',
        '土豆': '#D4AC6E',
        '南瓜': '#FF8C00'
      };
      return colors[crop] || '#409EFF';
    },
    
    // 计算健康指数
    async calculateHealthScore(farm) {
      if (!farm || !farm.id) return 0;
      try {
        const res = await this.request.get(`/health-index/calculate/${farm.id}`);
        if (res.code === '200' && res.data) {
          return res.data.healthIndex || 0;
        }
      } catch (error) {
        console.error('计算健康指数失败:', error);
      }
      // 如果API失败，使用本地计算
      return this.calculateHealthScoreLocal(farm);
    },
    
    // 本地计算健康指数（备用方法）
    calculateHealthScoreLocal(farm) {
      if (!farm) return 0;
      let totalScore = 0;
      let count = 0;
      
      // 温度评分
      if (farm.temperature != null) {
        const temp = parseFloat(farm.temperature);
        if (temp >= 20 && temp <= 28) totalScore += 100;
        else if (temp >= 15 && temp <= 32) totalScore += 75;
        else totalScore += 50;
        count++;
      }
      
      // 空气湿度评分
      if (farm.airhumidity != null) {
        const airHum = parseInt(farm.airhumidity);
        if (airHum >= 60 && airHum <= 80) totalScore += 100;
        else if (airHum >= 50 && airHum <= 90) totalScore += 75;
        else totalScore += 50;
        count++;
      }
      
      // 土壤湿度评分
      if (farm.soilhumidity != null) {
        const soilHum = parseInt(farm.soilhumidity);
        if (soilHum >= 40 && soilHum <= 70) totalScore += 100;
        else if (soilHum >= 30 && soilHum <= 80) totalScore += 75;
        else totalScore += 50;
        count++;
      }
      
      // CO2评分
      if (farm.carbon != null) {
        const carbon = parseInt(farm.carbon);
        if (carbon >= 350 && carbon <= 450) totalScore += 100;
        else if (carbon >= 300 && carbon <= 1500) totalScore += 75;
        else totalScore += 50;
        count++;
      }
      
      // pH评分
      if (farm.ph != null) {
        const ph = parseFloat(farm.ph);
        if (ph >= 6.0 && ph <= 7.5) totalScore += 100;
        else if (ph >= 5.5 && ph <= 8.0) totalScore += 75;
        else totalScore += 50;
        count++;
      }
      
      // 光照评分
      if (farm.light != null) {
        const light = parseInt(farm.light);
        if (light >= 2000 && light <= 3000) totalScore += 100;
        else if (light >= 1500 && light <= 3500) totalScore += 75;
        else totalScore += 50;
        count++;
      }
      
      if (count === 0) return 0;
      return Math.round(totalScore / count);
    },
    
    // 获取健康指数渐变背景
    getHealthScoreGradient(farm) {
      const score = this.calculateHealthScoreLocal(farm);
      if (score >= 90) {
        return 'linear-gradient(135deg, #67C23A 0%, #85CE61 100%)';
      } else if (score >= 75) {
        return 'linear-gradient(135deg, #409EFF 0%, #66B1FF 100%)';
      } else if (score >= 60) {
        return 'linear-gradient(135deg, #E6A23C 0%, #EEBE77 100%)';
      } else {
        return 'linear-gradient(135deg, #F56C6C 0%, #F78989 100%)';
      }
    },
    
    // 获取健康等级标签类型
    getHealthLevelTag(farm) {
      const score = this.calculateHealthScoreLocal(farm);
      if (score >= 90) return 'success';
      if (score >= 75) return 'primary';
      if (score >= 60) return 'warning';
      return 'danger';
    },
    
    // 获取健康等级文本
    getHealthLevelText(farm) {
      const score = this.calculateHealthScoreLocal(farm);
      if (score >= 90) return '优秀';
      if (score >= 75) return '良好';
      if (score >= 60) return '一般';
      return '需关注';
    }
  }
};
</script>

<style scoped>
.big-screen-container {
  width: 100%;
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  padding: 20px;
}

/* 顶部信息栏 */
.top-info-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding: 15px 30px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}

.title {
  font-size: 24px;
  font-weight: bold;
  color: #303133;
  display: flex;
  align-items: center;
  gap: 10px;
}

.title i {
  font-size: 28px;
  color: #409EFF;
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
  height: calc(100vh - 160px);
}

.left-section {
  width: 320px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.center-section {
  flex: 1;
  background: white;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  display: flex;
  flex-direction: column;
}

.right-section {
  width: 380px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.task-panel {
  flex: 1;
  overflow: hidden;
}

/* 统计卡片组 */
.stat-card-group {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
}

.stat-card {
  background: white;
  border-radius: 12px;
  padding: 15px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  display: flex;
  align-items: center;
  gap: 12px;
  transition: all 0.3s;
  cursor: pointer;
}

.stat-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.12);
}

.stat-icon {
  width: 50px;
  height: 50px;
  border-radius: 10px;
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

.stat-label {
  font-size: 12px;
  color: #909399;
  margin-bottom: 5px;
}

.stat-value {
  font-size: 22px;
  font-weight: bold;
  color: #303133;
}

.stat-value .unit {
  font-size: 12px;
  color: #909399;
  margin-left: 4px;
}

/* 农田列表 */
.farm-list-panel {
  flex: 1;
  background: white;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  display: flex;
  flex-direction: column;
}

.panel-title {
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

.panel-title i {
  color: #409EFF;
  font-size: 18px;
}

.farm-list {
  flex: 1;
  overflow-y: auto;
}

.farm-item {
  padding: 12px;
  margin-bottom: 10px;
  background: #f5f7fa;
  border-radius: 8px;
  border: 2px solid transparent;
  cursor: pointer;
  transition: all 0.3s;
}

.farm-item:hover {
  background: #ecf5ff;
  border-color: #409EFF;
}

.farm-item.active {
  background: #ecf5ff;
  border-color: #409EFF;
  box-shadow: 0 2px 8px rgba(64, 158, 255, 0.3);
}

.farm-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.farm-name {
  font-size: 15px;
  font-weight: bold;
  color: #303133;
}

.farm-info {
  font-size: 12px;
  color: #606266;
}

.info-item {
  margin-bottom: 5px;
}

.info-row {
  display: flex;
  gap: 10px;
  margin-top: 5px;
}

/* 3D 图表 */
.chart-3d {
  flex: 1;
  width: 100%;
  min-height: 600px;
  background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
  border-radius: 8px;
  position: relative;
}

/* 详情面板 */
.detail-panel {
  background: white;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  height: 100%;
  overflow-y: auto;
}

.detail-content {
  margin-top: 15px;
}

.detail-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 1px solid #eee;
}

.detail-header h3 {
  margin: 0;
  color: #303133;
  font-size: 18px;
}

.detail-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
  margin-bottom: 20px;
}

.detail-item {
  background: #f5f7fa;
  padding: 12px;
  border-radius: 8px;
}

.detail-label {
  font-size: 12px;
  color: #909399;
  margin-bottom: 5px;
}

.detail-value {
  font-size: 15px;
  font-weight: bold;
  color: #303133;
}

.env-monitor,
.health-index-section {
  margin-top: 20px;
  padding: 15px;
  background: #f5f7fa;
  border-radius: 8px;
}

.health-index-display {
  display: flex;
  align-items: center;
  gap: 15px;
}

.health-score {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: white;
  font-weight: bold;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.score-value {
  font-size: 32px;
  line-height: 1;
}

.score-label {
  font-size: 12px;
  margin-top: 5px;
  opacity: 0.9;
}

.health-level {
  flex: 1;
}

.device-status {
  margin-top: 20px;
}

.monitor-title {
  font-size: 14px;
  font-weight: bold;
  color: #303133;
  margin-bottom: 12px;
  padding-left: 10px;
  border-left: 3px solid #409EFF;
}

.monitor-item {
  background: #f5f7fa;
  padding: 10px;
  border-radius: 6px;
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 10px;
}

.monitor-label {
  font-size: 12px;
  color: #606266;
  flex: 1;
}

.monitor-value {
  font-size: 14px;
  font-weight: bold;
  color: #303133;
}

.device-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.device-item {
  background: #f5f7fa;
  padding: 10px 15px;
  border-radius: 6px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.no-selection {
  text-align: center;
  padding: 60px 20px;
  color: #909399;
}

.no-selection i {
  font-size: 48px;
  margin-bottom: 15px;
  display: block;
}

/* 滚动条样式 */
.farm-list::-webkit-scrollbar,
.detail-panel::-webkit-scrollbar {
  width: 6px;
}

.farm-list::-webkit-scrollbar-thumb,
.detail-panel::-webkit-scrollbar-thumb {
  background: #dcdfe6;
  border-radius: 3px;
}

.farm-list::-webkit-scrollbar-track,
.detail-panel::-webkit-scrollbar-track {
  background: #f5f7fa;
}
</style>
