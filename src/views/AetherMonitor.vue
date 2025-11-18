<template>
  <div class="aether-monitor">
    <!-- 绿色粒子背景 -->
    <canvas ref="particleCanvas" class="particle-canvas"></canvas>
    
    <!-- 顶部标题栏 -->
    <div class="header">
      <div class="title">AetherHarvest · 智慧农业数据监控</div>
      <div class="status-group">
        <el-tag :type="deviceOnline ? 'success' : 'danger'" size="small">
          {{ deviceOnline ? '设备在线' : '设备离线' }}
        </el-tag>
      </div>
    </div>

    <!-- 关键指标卡片 -->
    <el-row :gutter="20" class="kpi-row">
      <!-- 室外温度 -->
      <el-col :span="6">
        <el-card class="kpi-card weather-card">
          <div class="kpi-label">室外温度</div>
          <div class="kpi-value">
            {{ weatherNow && weatherNow.temp ? weatherNow.temp + '℃' : '--' }}
          </div>
          <div class="kpi-desc">
            {{ weatherNow && weatherNow.text || '加载中...' }}
          </div>
        </el-card>
      </el-col>

      <!-- 室内温度 -->
      <el-col :span="6">
        <el-card class="kpi-card">
          <div class="kpi-label">室内温度</div>
          <div class="kpi-value">{{ temperature === null ? '--' : temperature + '℃' }}</div>
          <div class="kpi-desc">STM32 实时检测</div>
        </el-card>
      </el-col>

      <!-- 室内湿度 -->
      <el-col :span="6">
        <el-card class="kpi-card">
          <div class="kpi-label">室内湿度</div>
          <div class="kpi-value">{{ humidity === null ? '--' : humidity + '%' }}</div>
          <div class="kpi-desc">STM32 实时检测</div>
        </el-card>
      </el-col>

      <!-- 光照灯控制 -->
      <el-col :span="6">
        <el-card class="kpi-card lamp-card">
          <div class="kpi-label">光照灯</div>
          <div class="lamp-control">
            <i class="el-icon-sunny" :class="{ active: ledOn }"></i>
            <el-switch
              v-model="ledOn"
              @change="toggleLamp"
              active-color="#13ce66"
              inactive-color="#dcdfe6"
            ></el-switch>
          </div>
          <div class="kpi-desc">{{ ledOn ? '已开启' : '已关闭' }}</div>
        </el-card>
      </el-col>
    </el-row>

    <!-- 天气预报和预警 -->
    <el-row :gutter="20" class="weather-row">
      <!-- 天气预报 -->
      <el-col :span="16">
        <el-card class="weather-forecast-card">
          <div slot="header" class="card-header">
            <span>📅 天气预报</span>
          </div>
          <el-tabs v-model="activeWeatherTab">
            <el-tab-pane label="24小时" name="24h">
              <div class="forecast-list" v-if="weather24h.length > 0">
                <div v-for="(item, index) in weather24h.slice(0, 8)" :key="index" class="forecast-item">
                  <div class="time">{{ formatTime(item.fxTime) }}</div>
                  <div class="weather-icon">{{ getWeatherIcon(item.text) }}</div>
                  <div class="temp">{{ item.temp }}℃</div>
                </div>
              </div>
              <el-empty v-else description="暂无数据" :image-size="60"></el-empty>
            </el-tab-pane>
            <el-tab-pane label="7天" name="7d">
              <div class="forecast-list" v-if="weather7d.length > 0">
                <div v-for="(item, index) in weather7d" :key="index" class="forecast-item">
                  <div class="time">{{ formatDate(item.fxDate) }}</div>
                  <div class="weather-icon">{{ getWeatherIcon(item.textDay) }}</div>
                  <div class="temp">{{ item.tempMin }}~{{ item.tempMax }}℃</div>
                </div>
              </div>
              <el-empty v-else description="暂无数据" :image-size="60"></el-empty>
            </el-tab-pane>
          </el-tabs>
        </el-card>
      </el-col>

      <!-- 预警信息 -->
      <el-col :span="8">
        <el-card class="alert-card">
          <div slot="header" class="card-header">
            <span>⚠️ 预警信息</span>
          </div>
          <div class="alert-list">
            <!-- 天气预警 -->
            <div v-if="weatherAlerts.length > 0">
              <div v-for="(alert, index) in weatherAlerts" :key="'weather-' + index" class="alert-item warning">
                <i class="el-icon-warning"></i>
                <div class="alert-content">
                  <div class="alert-title">{{ alert.title }}</div>
                  <div class="alert-desc">{{ alert.text }}</div>
                </div>
              </div>
            </div>
            <!-- 温湿度预警 -->
            <div v-if="tempAlert" class="alert-item danger">
              <i class="el-icon-warning"></i>
              <div class="alert-content">
                <div class="alert-title">温度异常</div>
                <div class="alert-desc">{{ tempAlert }}</div>
              </div>
            </div>
            <div v-if="humiAlert" class="alert-item warning">
              <i class="el-icon-warning"></i>
              <div class="alert-content">
                <div class="alert-title">湿度异常</div>
                <div class="alert-desc">{{ humiAlert }}</div>
              </div>
            </div>
            <el-empty v-if="!weatherAlerts.length && !tempAlert && !humiAlert" description="暂无预警" :image-size="60"></el-empty>
          </div>
        </el-card>
      </el-col>
    </el-row>

    <!-- 历史数据图表 -->
    <el-row :gutter="20" class="chart-row">
      <el-col :span="12">
        <el-card>
          <div slot="header" class="card-header">
            <span>📈 温度趋势</span>
          </div>
          <div ref="tempChart" style="height: 300px;"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card>
          <div slot="header" class="card-header">
            <span>💧 湿度趋势</span>
          </div>
          <div ref="humiChart" style="height: 300px;"></div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import * as echarts from 'echarts';

export default {
  name: 'AetherMonitor',
  data() {
    return {
      // 设备状态
      deviceOnline: false,
      temperature: null,
      humidity: null,
      ledOn: false,
      
      // 天气数据
      weatherNow: null,
      weather24h: [],
      weather7d: [],
      weatherAlerts: [],
      activeWeatherTab: '24h',
      
      // 预警
      tempAlert: '',
      humiAlert: '',
      
      // 阈值
      tempThresholdHigh: 30,
      tempThresholdLow: 10,
      humiThresholdHigh: 75,
      humiThresholdLow: 25,
      
      // 历史数据
      historyData: [],
      
      // 图表实例
      tempChartInstance: null,
      humiChartInstance: null,
      
      // 轮询定时器
      pollingTimer: null,
      weatherTimer: null,
      
      // 粒子动画相关
      particles: [],
      canvas: null,
      ctx: null,
      animationId: null,
      canvasWidth: 0,
      canvasHeight: 0,
      mouseX: 0,
      mouseY: 0,
      mouseMoving: false
    }
  },
  
  mounted() {
    this.initParticleCanvas();
    this.initCharts();
    this.fetchDeviceStatus();
    this.fetchWeatherData();
    this.fetchHistoryData();
    
    // 启动轮询
    this.pollingTimer = setInterval(() => {
      this.fetchDeviceStatus();
    }, 5000); // 每5秒更新一次设备状态
    
    this.weatherTimer = setInterval(() => {
      this.fetchWeatherData();
    }, 300000); // 每5分钟更新一次天气
  },
  
  beforeDestroy() {
    if (this.pollingTimer) {
      clearInterval(this.pollingTimer);
    }
    if (this.weatherTimer) {
      clearInterval(this.weatherTimer);
    }
    if (this.tempChartInstance) {
      this.tempChartInstance.dispose();
    }
    if (this.humiChartInstance) {
      this.humiChartInstance.dispose();
    }
    if (this.animationId) {
      cancelAnimationFrame(this.animationId);
    }
    window.removeEventListener('resize', this.resizeCanvas);
  },
  
  methods: {
    // 获取设备状态
    async fetchDeviceStatus() {
      try {
        const res = await this.request.get('/aether/device/status');
        
        if (res.code === '200' && res.data) {
          const data = res.data;
          this.deviceOnline = data.online;
          this.temperature = data.temperature;
          this.humidity = data.humidity;
          this.ledOn = data.led === 1;
          
          // 检查阈值预警
          this.checkThresholds();
        }
      } catch (error) {
        console.error('获取设备状态失败:', error);
        this.deviceOnline = false;
      }
    },
    
    // 控制LED灯
    async toggleLamp() {
      try {
        const res = await this.request.post('/aether/device/control/led', {
          led: this.ledOn ? 1 : 0
        });
        
        if (res.code === '200') {
          this.$message.success(this.ledOn ? '灯已开启' : '灯已关闭');
        } else {
          this.$message.error('控制失败');
          this.ledOn = !this.ledOn; // 恢复状态
        }
      } catch (error) {
        console.error('控制LED失败:', error);
        this.$message.error('控制失败');
        this.ledOn = !this.ledOn;
      }
    },
    
    // 获取天气数据
    async fetchWeatherData() {
      try {
        // 获取实时天气
        const nowRes = await this.request.get('/aether/weather/now');
        if (nowRes.code === '200' && nowRes.data && nowRes.data.data) {
          this.weatherNow = nowRes.data.data;
        }
        
        // 获取7天天气
        const weekRes = await this.request.get('/aether/weather/7d');
        if (weekRes.code === '200' && weekRes.data && weekRes.data.data) {
          this.weather7d = weekRes.data.data.daily || [];
          this.weather24h = weekRes.data.data.hourly || [];
        }
        
        // 获取预警
        const alertRes = await this.request.get('/aether/weather/alerts');
        if (alertRes.code === '200' && alertRes.data) {
          this.weatherAlerts = alertRes.data.data || [];
        }
      } catch (error) {
        console.error('获取天气数据失败:', error);
      }
    },
    
    // 获取历史数据
    async fetchHistoryData() {
      try {
        const res = await this.request.get('/aether/readings/detail?days=7');
        
        if (res.code === '200' && res.data && res.data.data) {
          this.historyData = res.data.data || [];
          this.updateCharts();
        }
      } catch (error) {
        console.error('获取历史数据失败:', error);
      }
    },
    
    // 检查阈值
    checkThresholds() {
      this.tempAlert = '';
      this.humiAlert = '';
      
      if (this.temperature !== null) {
        if (this.temperature > this.tempThresholdHigh) {
          this.tempAlert = `当前温度 ${this.temperature}℃ 超过上限 ${this.tempThresholdHigh}℃`;
        } else if (this.temperature < this.tempThresholdLow) {
          this.tempAlert = `当前温度 ${this.temperature}℃ 低于下限 ${this.tempThresholdLow}℃`;
        }
      }
      
      if (this.humidity !== null) {
        if (this.humidity > this.humiThresholdHigh) {
          this.humiAlert = `当前湿度 ${this.humidity}% 超过上限 ${this.humiThresholdHigh}%`;
        } else if (this.humidity < this.humiThresholdLow) {
          this.humiAlert = `当前湿度 ${this.humidity}% 低于下限 ${this.humiThresholdLow}%`;
        }
      }
    },
    
    // 初始化图表
    initCharts() {
      this.tempChartInstance = echarts.init(this.$refs.tempChart);
      this.humiChartInstance = echarts.init(this.$refs.humiChart);
      
      window.addEventListener('resize', () => {
        this.tempChartInstance && this.tempChartInstance.resize();
        this.humiChartInstance && this.humiChartInstance.resize();
      });
    },
    
    // 更新图表
    updateCharts() {
      if (!this.historyData || this.historyData.length === 0) return;
      
      const times = this.historyData.map(item => {
        const date = new Date(item.date);
        return `${date.getMonth() + 1}/${date.getDate()} ${date.getHours()}:${String(date.getMinutes()).padStart(2, '0')}`;
      });
      const temps = this.historyData.map(item => item.temp);
      const humis = this.historyData.map(item => item.humi);
      
      // 温度图表
      const tempOption = {
        tooltip: {
          trigger: 'axis'
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: {
          type: 'category',
          data: times,
          boundaryGap: false
        },
        yAxis: {
          type: 'value',
          name: '温度(℃)'
        },
        series: [{
          name: '温度',
          type: 'line',
          smooth: true,
          data: temps,
          itemStyle: {
            color: '#409EFF'
          },
          areaStyle: {
            color: {
              type: 'linear',
              x: 0,
              y: 0,
              x2: 0,
              y2: 1,
              colorStops: [{
                offset: 0, color: 'rgba(64, 158, 255, 0.3)'
              }, {
                offset: 1, color: 'rgba(64, 158, 255, 0.05)'
              }]
            }
          }
        }]
      };
      
      // 湿度图表
      const humiOption = {
        tooltip: {
          trigger: 'axis'
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: {
          type: 'category',
          data: times,
          boundaryGap: false
        },
        yAxis: {
          type: 'value',
          name: '湿度(%)'
        },
        series: [{
          name: '湿度',
          type: 'line',
          smooth: true,
          data: humis,
          itemStyle: {
            color: '#67C23A'
          },
          areaStyle: {
            color: {
              type: 'linear',
              x: 0,
              y: 0,
              x2: 0,
              y2: 1,
              colorStops: [{
                offset: 0, color: 'rgba(103, 194, 58, 0.3)'
              }, {
                offset: 1, color: 'rgba(103, 194, 58, 0.05)'
              }]
            }
          }
        }]
      };
      
      this.tempChartInstance.setOption(tempOption);
      this.humiChartInstance.setOption(humiOption);
    },
    
    // 格式化时间
    formatTime(timeStr) {
      const date = new Date(timeStr);
      return `${date.getHours()}:00`;
    },
    
    // 格式化日期
    formatDate(dateStr) {
      const date = new Date(dateStr);
      return `${date.getMonth() + 1}/${date.getDate()}`;
    },
    
    // 获取天气图标
    getWeatherIcon(text) {
      const iconMap = {
        '晴': '☀️',
        '多云': '⛅',
        '阴': '☁️',
        '雨': '🌧️',
        '雪': '❄️',
        '雷': '⚡'
      };
      
      for (let key in iconMap) {
        if (text && text.includes(key)) {
          return iconMap[key];
        }
      }
      return '🌤️';
    },
    
    // ========== 粒子动画方法 ==========
    
    initParticleCanvas() {
      this.$nextTick(() => {
        this.canvas = this.$refs.particleCanvas;
        if (!this.canvas) return;
        
        this.ctx = this.canvas.getContext('2d');
        this.resizeCanvas();
        window.addEventListener('resize', this.resizeCanvas);
        this.initParticles();
        this.animate();
      });
    },
    
    resizeCanvas() {
      if (!this.canvas) return;
      const container = this.canvas.parentElement;
      if (container) {
        const { width, height } = container.getBoundingClientRect();
        const dpr = window.devicePixelRatio || 1;
        this.canvas.width = width * dpr;
        this.canvas.height = height * dpr;
        this.ctx.scale(dpr, dpr);
        this.canvas.style.width = `${width}px`;
        this.canvas.style.height = `${height}px`;
        this.canvasWidth = width;
        this.canvasHeight = height;
        if (this.particles.length === 0) {
          this.initParticles(Math.floor(width * height / 10000));
        }
      }
    },
    
    initParticles(count = 80) {
      const particles = [];
      const width = this.canvasWidth;
      const height = this.canvasHeight;
      for (let i = 0; i < count; i++) {
        const type = Math.random() > 0.9 ? 'special' : (Math.random() > 0.8 ? 'bright' : 'normal');
        const angle = Math.random() * Math.PI * 2;
        const speed = 0.3 + Math.random() * 0.3;
        particles.push({
          x: Math.random() * width,
          y: Math.random() * height,
          vx: Math.cos(angle) * speed,
          vy: Math.sin(angle) * speed,
          r: type === 'special' ? 2.5 : (type === 'bright' ? 2 : 1.5),
          alpha: type === 'special' ? 0.8 : (type === 'bright' ? 0.6 : 0.5),
          color: this.getRandomGreenColor(),
          type: type
        });
      }
      this.particles = particles;
    },
    
    getRandomGreenColor() {
      const colors = [[76, 175, 80], [102, 187, 106], [129, 199, 132], [165, 214, 167], [144, 238, 144], [50, 205, 50]];
      return colors[Math.floor(Math.random() * colors.length)];
    },
    
    animate() {
      if (!this.ctx) return;
      const ctx = this.ctx;
      const particles = this.particles;
      const width = this.canvasWidth;
      const height = this.canvasHeight;
      
      ctx.clearRect(0, 0, width, height);
      const bgGradient = ctx.createLinearGradient(0, 0, width, height);
      bgGradient.addColorStop(0, "rgba(214, 234, 212, 0.3)");
      bgGradient.addColorStop(0.3, "rgba(200, 230, 201, 0.25)");
      bgGradient.addColorStop(0.6, "rgba(180, 220, 180, 0.2)");
      bgGradient.addColorStop(1, "rgba(165, 214, 167, 0.15)");
      ctx.fillStyle = bgGradient;
      ctx.fillRect(0, 0, width, height);
      
      for (let i = 0; i < particles.length; i++) {
        const p = particles[i];
        p.x = (p.x + p.vx + width) % width;
        p.y = (p.y + p.vy + height) % height;
        const speed = Math.sqrt(p.vx * p.vx + p.vy * p.vy);
        if (speed > 1) {
          p.vx *= 0.98;
          p.vy *= 0.98;
        }
        
        ctx.beginPath();
        ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2);
        ctx.fillStyle = `rgba(${p.color[0]}, ${p.color[1]}, ${p.color[2]}, ${p.alpha})`;
        ctx.fill();
        
        if (p.type === 'special' || p.type === 'bright') {
          const gradient = ctx.createRadialGradient(p.x, p.y, 0, p.x, p.y, p.r * 3);
          gradient.addColorStop(0, `rgba(${p.color[0]}, ${p.color[1]}, ${p.color[2]}, ${p.alpha * 0.3})`);
          gradient.addColorStop(1, 'rgba(0, 0, 0, 0)');
          ctx.fillStyle = gradient;
          ctx.fillRect(p.x - p.r * 3, p.y - p.r * 3, p.r * 6, p.r * 6);
        }
        
        for (let j = i + 1; j < particles.length; j++) {
          const p2 = particles[j];
          const dx = p.x - p2.x;
          const dy = p.y - p2.y;
          const distance = Math.sqrt(dx * dx + dy * dy);
          if (distance < 120) {
            ctx.beginPath();
            ctx.moveTo(p.x, p.y);
            ctx.lineTo(p2.x, p2.y);
            const opacity = (1 - distance / 120) * 0.3;
            ctx.strokeStyle = `rgba(129, 199, 132, ${opacity})`;
            ctx.lineWidth = 0.5;
            ctx.stroke();
          }
        }
      }
      this.animationId = requestAnimationFrame(this.animate);
    }
  }
}
</script>

<style scoped>
.aether-monitor {
  position: relative;
  padding: 20px;
  min-height: 100vh;
  overflow: hidden;
}

/* 粒子画布 */
.particle-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  pointer-events: none;
}

/* 确保内容在粒子之上 */
.aether-monitor > *:not(.particle-canvas) {
  position: relative;
  z-index: 1;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding: 15px 20px;
  background: rgba(255, 255, 255, 0.45) !important;
  backdrop-filter: blur(10px);
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
}

.title {
  font-size: 24px;
  font-weight: bold;
  color: #303133;
}

.kpi-row {
  margin-bottom: 20px;
}

.kpi-card {
  text-align: center;
  transition: all 0.3s;
  background: rgba(255, 255, 255, 0.45) !important;
  backdrop-filter: blur(10px);
}

.kpi-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}

.kpi-label {
  font-size: 14px;
  color: #909399;
  margin-bottom: 10px;
}

.kpi-value {
  font-size: 36px;
  font-weight: bold;
  color: #409EFF;
  margin: 10px 0;
}

.kpi-desc {
  font-size: 12px;
  color: #C0C4CC;
}

.weather-card .kpi-value {
  color: #E6A23C;
}

.lamp-card .lamp-control {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  margin: 15px 0;
}

.lamp-card i {
  font-size: 48px;
  color: #DCDFE6;
  transition: all 0.3s;
}

.lamp-card i.active {
  color: #F56C6C;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.6; }
}

.weather-row,
.chart-row {
  margin-bottom: 20px;
}

.card-header {
  font-weight: bold;
  font-size: 16px;
}

.forecast-list {
  display: flex;
  justify-content: space-around;
  flex-wrap: wrap;
  gap: 10px;
}

.forecast-item {
  text-align: center;
  padding: 10px;
  border-radius: 8px;
  background: #f5f7fa;
  min-width: 80px;
}

.forecast-item .time {
  font-size: 12px;
  color: #909399;
  margin-bottom: 5px;
}

.forecast-item .weather-icon {
  font-size: 32px;
  margin: 5px 0;
}

.forecast-item .temp {
  font-size: 14px;
  font-weight: bold;
  color: #303133;
}

.alert-list {
  max-height: 400px;
  overflow-y: auto;
}

.alert-item {
  display: flex;
  align-items: flex-start;
  padding: 12px;
  margin-bottom: 10px;
  border-radius: 8px;
  background: #FEF0F0;
  border-left: 4px solid #F56C6C;
}

.alert-item.warning {
  background: #FDF6EC;
  border-left-color: #E6A23C;
}

.alert-item i {
  font-size: 20px;
  margin-right: 10px;
  color: #F56C6C;
}

.alert-item.warning i {
  color: #E6A23C;
}

.alert-content {
  flex: 1;
}

.alert-title {
  font-weight: bold;
  font-size: 14px;
  margin-bottom: 5px;
}

.alert-desc {
  font-size: 12px;
  color: #606266;
}

/* 所有卡片统一半透明效果 */
.weather-forecast-card,
.alert-card,
.chart-row .el-card {
  background: rgba(255, 255, 255, 0.45) !important;
  backdrop-filter: blur(10px);
}

/* Element UI卡片内部背景也要透明 */
.el-card {
  background: rgba(255, 255, 255, 0.45) !important;
  backdrop-filter: blur(10px);
}

.el-card__header {
  background: rgba(255, 255, 255, 0.3) !important;
  backdrop-filter: blur(5px);
}

.el-card__body {
  background: transparent !important;
}
</style>
