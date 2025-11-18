<template>
  <div class="farm-map-3d">
    <!-- 顶部信息栏 -->
    <div class="top-bar">
      <div class="title">
        <i class="el-icon-s-data"></i>
        智慧农场 3D 可视化大屏
      </div>
      <div class="time">{{ currentTime }}</div>
      <div class="weather">
        <i class="el-icon-sunny"></i>
        {{ weather.temp }}°C {{ weather.status }}
      </div>
    </div>

    <!-- 主内容区 -->
    <div class="main-container">
      <!-- 左侧：统计卡片和农田列表 -->
      <div class="left-panel">
        <!-- 统计卡片 -->
        <div class="stats-grid">
          <div class="stat-card" style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);">
            <div class="stat-icon">🏞️</div>
            <div class="stat-info">
              <div class="stat-label">农田总数</div>
              <div class="stat-value">{{ summary.farmCount }}<span class="unit">块</span></div>
            </div>
          </div>
          
          <div class="stat-card" style="background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);">
            <div class="stat-icon">📏</div>
            <div class="stat-info">
              <div class="stat-label">总面积</div>
              <div class="stat-value">{{ summary.totalArea }}<span class="unit">亩</span></div>
            </div>
          </div>
          
          <div class="stat-card" style="background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);">
            <div class="stat-icon">🌾</div>
            <div class="stat-info">
              <div class="stat-label">作物总量</div>
              <div class="stat-value">{{ summary.totalStock }}<span class="unit">株</span></div>
            </div>
          </div>
          
          <div class="stat-card" style="background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%);">
            <div class="stat-icon">✅</div>
            <div class="stat-info">
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
                <div class="info-row">
                  <span>🌾 {{ farm.crop }}</span>
                  <span>📏 {{ farm.area }}亩</span>
                </div>
                <div class="info-row">
                  <span>🌱 {{ farm.number }}株</span>
                  <span>📍 {{ farm.address }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 中间：3D 地图 -->
      <div class="center-panel">
        <div class="panel-title">
          <i class="el-icon-view"></i>
          农田 3D 可视化
        </div>
        <div class="map-3d" ref="threeContainer"></div>
      </div>

      <!-- 右侧：农田详情 -->
      <div class="right-panel">
        <div class="panel-title">
          <i class="el-icon-document"></i>
          农田详情
        </div>
        <div v-if="selectedFarm" class="farm-detail">
          <div class="detail-header">
            <h3>{{ selectedFarm.farm }}</h3>
            <el-tag :type="selectedFarm.state === '正常' ? 'success' : 'warning'">
              {{ selectedFarm.state }}
            </el-tag>
          </div>
          
          <div class="detail-grid">
            <div class="detail-item">
              <div class="label">作物类型</div>
              <div class="value">{{ selectedFarm.crop }}</div>
            </div>
            <div class="detail-item">
              <div class="label">种植面积</div>
              <div class="value">{{ selectedFarm.area }} 亩</div>
            </div>
            <div class="detail-item">
              <div class="label">作物数量</div>
              <div class="value">{{ selectedFarm.number }} 株</div>
            </div>
            <div class="detail-item">
              <div class="label">所在位置</div>
              <div class="value">{{ selectedFarm.address }}</div>
            </div>
            <div class="detail-item">
              <div class="label">负责人</div>
              <div class="value">{{ selectedFarm.keeper }}</div>
            </div>
          </div>

          <!-- 环境健康评分 -->
          <div class="env-health-score">
            <div class="score-circle" :style="{ background: getHealthScoreGradient(selectedFarm) }">
              <div class="score-inner">
                <div class="score-number">{{ calculateHealthScore(selectedFarm) }}</div>
                <div class="score-label">健康指数</div>
              </div>
            </div>
            <div class="score-status">
              <div class="status-badge" :class="getHealthLevel(selectedFarm).class">
                {{ getHealthLevel(selectedFarm).text }}
              </div>
              <div class="status-desc">{{ getHealthLevel(selectedFarm).desc }}</div>
            </div>
          </div>

          <!-- 环境监测数据 - 创新可视化 -->
          <div class="env-section">
            <div class="section-title">
              <span>🌡️ 实时环境监测</span>
              <span class="update-time">{{ currentTime.split(' ')[1] }}</span>
            </div>
            
            <!-- 环形进度条组 -->
            <div class="env-circles">
              <div class="circle-item" v-for="item in getEnvMetrics(selectedFarm)" :key="item.key">
                <div class="circle-progress">
                  <svg width="80" height="80" class="progress-ring">
                    <circle
                      class="progress-ring-bg"
                      cx="40"
                      cy="40"
                      r="32"
                      fill="none"
                      stroke="#f0f0f0"
                      stroke-width="6"
                    />
                    <circle
                      class="progress-ring-circle"
                      cx="40"
                      cy="40"
                      r="32"
                      fill="none"
                      :stroke="item.color"
                      stroke-width="6"
                      stroke-linecap="round"
                      :stroke-dasharray="calculateDashArray(item.percent)"
                      :stroke-dashoffset="0"
                      transform="rotate(-90 40 40)"
                    />
                  </svg>
                  <div class="circle-content">
                    <div class="circle-icon">{{ item.icon }}</div>
                    <div class="circle-value">{{ item.value }}</div>
                  </div>
                </div>
                <div class="circle-label">{{ item.label }}</div>
                <div class="circle-status" :style="{ color: item.statusColor }">
                  {{ item.status }}
                </div>
              </div>
            </div>

            <!-- 智能建议 -->
            <div class="smart-suggestions">
              <div class="suggestion-title">
                <i class="el-icon-magic-stick"></i>
                智能建议
              </div>
              <div class="suggestion-list">
                <div 
                  v-for="(suggestion, index) in getSmartSuggestions(selectedFarm)" 
                  :key="index"
                  class="suggestion-item"
                  :class="suggestion.type"
                >
                  <div class="suggestion-icon">{{ suggestion.icon }}</div>
                  <div class="suggestion-content">
                    <div class="suggestion-text">{{ suggestion.text }}</div>
                    <div class="suggestion-action">{{ suggestion.action }}</div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div v-else class="no-selection">
          <i class="el-icon-info"></i>
          <p>请点击左侧农田列表查看详情</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';

export default {
  name: 'FarmMap3D',
  data() {
    return {
      currentTime: '',
      selectedFarm: null,
      weather: {
        temp: 28,
        status: '晴'
      },
      summary: {
        farmCount: 0,
        totalArea: 0,
        totalStock: 0,
        normalCount: 0
      },
      farms: [],
      
      // Three.js 相关
      scene: null,
      camera: null,
      renderer: null,
      controls: null,
      farmBlocks: [],
      labels: [], // 存储所有标签精灵
      raycaster: null,
      mouse: null,
      selectedBlock: null,
      timeInterval: null, // 时间更新定时器
      lastRenderTime: null, // 上次渲染时间，用于帧率控制
      mouseMoveTimer: null, // 鼠标移动节流定时器
      boundMouseClick: null, // 绑定的事件处理函数
      boundMouseMove: null,
      boundWindowResize: null,
      
      // 模型加载器和缓存
      gltfLoader: null,
      modelCache: {}, // 缓存已加载的模型
      
      // 🌾 作物配置中心 - 添加新植物只需在这里配置即可
      cropConfig: {
        '小麦': {
          icon: '🌾',
          color: 0xF4D03F,  // 金黄色
          modelPath: '/models/crops/wheat.glb'
        },
        '玉米': {
          icon: '🌽',
          color: 0xF39C12,  // 橙黄色
          modelPath: '/models/crops/corn.glb'
        },
        '水稻': {
          icon: '🌾',
          color: 0x52BE80,  // 绿色
          modelPath: '/models/crops/rice.glb'
        },
        '大豆': {
          icon: '🫘',
          color: 0x85C1E2,  // 浅蓝色
          modelPath: '/models/crops/soybean.glb'
        },
        '番茄': {
          icon: '🍅',
          color: 0xE74C3C,  // 红色
          modelPath: '/models/crops/tomato.glb'
        },
        '黄瓜': {
          icon: '🥒',
          color: 0x27AE60,  // 深绿色
          modelPath: '/models/crops/cucumber.glb'
        },
        '辣椒': {
          icon: '🌶️',
          color: 0xC0392B,  // 深红色
          modelPath: '/models/crops/pepper.glb'
        },
        '茄子': {
          icon: '🍆',
          color: 0x8E44AD,  // 紫色
          modelPath: '/models/crops/eggplant.glb'
        },
        '白菜': {
          icon: '🥬',
          color: 0x2ECC71,  // 亮绿色
          modelPath: '/models/crops/cabbage.glb'
        },
        '萝卜': {
          icon: '🥕',
          color: 0xE67E22,  // 橙色
          modelPath: '/models/crops/carrot.glb'
        },
        '土豆': {
          icon: '🥔',
          color: 0xD4AC6E,  // 土黄色
          modelPath: '/models/crops/potato.glb'
        },
        '南瓜': {
          icon: '🎃',
          color: 0xFF8C00,  // 深橙色
          modelPath: '/models/crops/pumpkin.glb'
        }
      },
      
      // 默认配置（当作物不在配置中时使用）
      defaultCropConfig: {
        icon: '🌱',
        color: 0x52BE80,
        modelPath: null
      }
    };
  },
  
  mounted() {
    this.updateTime();
    this.timeInterval = setInterval(this.updateTime, 1000);
    this.fetchData();
  },
  
  beforeDestroy() {
    if (this.timeInterval) {
      clearInterval(this.timeInterval);
    }
    // 销毁Three.js资源
    if (this.renderer) {
      this.renderer.dispose();
      if (this.$refs.threeContainer) {
        this.$refs.threeContainer.removeChild(this.renderer.domElement);
      }
    }
    if (this.controls) {
      this.controls.dispose();
    }
    // 清理事件监听
    const container = this.$refs.threeContainer;
    if (container) {
      if (this.boundMouseClick) {
        container.removeEventListener('click', this.boundMouseClick);
      }
      if (this.boundMouseMove) {
        container.removeEventListener('mousemove', this.boundMouseMove);
      }
    }
    if (this.boundWindowResize) {
      window.removeEventListener('resize', this.boundWindowResize);
    }
    
    // 清理定时器
    if (this.mouseMoveTimer) {
      clearTimeout(this.mouseMoveTimer);
      this.mouseMoveTimer = null;
    }
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
          this.$nextTick(() => {
            this.init3DMap();
          });
        }
        
        const summaryRes = await this.request.get('/statistic/dashboard/summary');
        if (summaryRes.code === '200') {
          this.summary = summaryRes.data;
        }
      } catch (error) {
        console.error('获取数据失败:', error);
        this.$message.error('数据加载失败');
      }
    },
    
    selectFarm(farm) {
      this.selectedFarm = farm;
      // 在3D场景中高亮对应的方块
      this.highlightFarmBlock(farm);
    },
    
    // 计算环境健康评分（0-100）
    calculateHealthScore(farm) {
      if (!farm) return 0;
      
      const scores = [];
      
      // 温度评分（最佳范围 20-28°C）
      const temp = farm.temperature || 25;
      if (temp >= 20 && temp <= 28) {
        scores.push(100);
      } else if (temp >= 15 && temp <= 32) {
        scores.push(80);
      } else {
        scores.push(50);
      }
      
      // 空气湿度评分（最佳范围 60-80%）
      const airHum = farm.airhumidity || 70;
      if (airHum >= 60 && airHum <= 80) {
        scores.push(100);
      } else if (airHum >= 50 && airHum <= 90) {
        scores.push(75);
      } else {
        scores.push(50);
      }
      
      // 土壤湿度评分（最佳范围 40-70%）
      const soilHum = farm.soilhumidity || 60;
      if (soilHum >= 40 && soilHum <= 70) {
        scores.push(100);
      } else if (soilHum >= 30 && soilHum <= 80) {
        scores.push(75);
      } else {
        scores.push(50);
      }
      
      // 光照强度评分（最佳范围 20000-50000 lux）
      const light = farm.light || 30000;
      if (light >= 20000 && light <= 50000) {
        scores.push(100);
      } else if (light >= 10000 && light <= 60000) {
        scores.push(80);
      } else {
        scores.push(60);
      }
      
      // CO₂评分（最佳范围 350-450 ppm）
      const carbon = farm.carbon || 400;
      if (carbon >= 350 && carbon <= 450) {
        scores.push(100);
      } else if (carbon >= 300 && carbon <= 500) {
        scores.push(85);
      } else {
        scores.push(65);
      }
      
      // PH值评分（最佳范围 6.0-7.5）
      const ph = farm.ph || 6.8;
      if (ph >= 6.0 && ph <= 7.5) {
        scores.push(100);
      } else if (ph >= 5.5 && ph <= 8.0) {
        scores.push(80);
      } else {
        scores.push(55);
      }
      
      // 计算平均分
      const avgScore = scores.reduce((a, b) => a + b, 0) / scores.length;
      return Math.round(avgScore);
    },
    
    // 根据健康指数获取颜色（优秀=绿色，良好=蓝色，一般=黄色，需关注=红色）
    getHealthColor(score) {
      if (score >= 90) {
        return 0x67C23A; // 绿色 - 优秀
      } else if (score >= 75) {
        return 0x409EFF; // 蓝色 - 良好
      } else if (score >= 60) {
        return 0xE6A23C; // 黄色 - 一般
      } else {
        return 0xF56C6C; // 红色 - 需关注
      }
    },
    
    // 混合两种颜色
    blendColors(color1, color2, ratio) {
      const r1 = (color1 >> 16) & 0xFF;
      const g1 = (color1 >> 8) & 0xFF;
      const b1 = color1 & 0xFF;
      
      const r2 = (color2 >> 16) & 0xFF;
      const g2 = (color2 >> 8) & 0xFF;
      const b2 = color2 & 0xFF;
      
      const r = Math.round(r1 * (1 - ratio) + r2 * ratio);
      const g = Math.round(g1 * (1 - ratio) + g2 * ratio);
      const b = Math.round(b1 * (1 - ratio) + b2 * ratio);
      
      return (r << 16) | (g << 8) | b;
    },
    
    // 添加健康指数标签
    addHealthIndexLabel(score, x, y, z) {
      const canvas = document.createElement('canvas');
      const context = canvas.getContext('2d');
      canvas.width = 120;
      canvas.height = 40;
      
      // 根据分数设置背景颜色
      let bgColor = 'rgba(103, 194, 58, 0.9)'; // 绿色
      let textColor = '#ffffff';
      if (score >= 90) {
        bgColor = 'rgba(103, 194, 58, 0.9)'; // 绿色
      } else if (score >= 75) {
        bgColor = 'rgba(64, 158, 255, 0.9)'; // 蓝色
      } else if (score >= 60) {
        bgColor = 'rgba(230, 162, 60, 0.9)'; // 黄色
        textColor = '#ffffff';
      } else {
        bgColor = 'rgba(245, 108, 108, 0.9)'; // 红色
      }
      
      // 绘制背景
      context.fillStyle = bgColor;
      context.shadowColor = 'rgba(0, 0, 0, 0.2)';
      context.shadowBlur = 5;
      context.shadowOffsetX = 1;
      context.shadowOffsetY = 1;
      
      const radius = 6;
      context.beginPath();
      context.moveTo(radius, 0);
      context.lineTo(canvas.width - radius, 0);
      context.quadraticCurveTo(canvas.width, 0, canvas.width, radius);
      context.lineTo(canvas.width, canvas.height - radius);
      context.quadraticCurveTo(canvas.width, canvas.height, canvas.width - radius, canvas.height);
      context.lineTo(radius, canvas.height);
      context.quadraticCurveTo(0, canvas.height, 0, canvas.height - radius);
      context.lineTo(0, radius);
      context.quadraticCurveTo(0, 0, radius, 0);
      context.closePath();
      context.fill();
      
      // 绘制文字
      context.fillStyle = textColor;
      context.font = 'bold 18px Arial';
      context.textAlign = 'center';
      context.textBaseline = 'middle';
      context.fillText(`${score}`, canvas.width / 2, canvas.height / 2 - 5);
      
      context.font = '12px Arial';
      context.fillText('健康指数', canvas.width / 2, canvas.height / 2 + 10);
      
      // 创建纹理
      const texture = new THREE.CanvasTexture(canvas);
      texture.needsUpdate = true;
      
      // 创建精灵
      const spriteMaterial = new THREE.SpriteMaterial({
        map: texture,
        transparent: true,
        depthTest: false
      });
      const sprite = new THREE.Sprite(spriteMaterial);
      sprite.position.set(x, y, z);
      sprite.scale.set(2, 2, 1);
      
      this.scene.add(sprite);
      this.labels.push(sprite);
    },
    
    // 获取健康等级
    getHealthLevel(farm) {
      const score = this.calculateHealthScore(farm);
      
      if (score >= 90) {
        return {
          text: '优秀',
          desc: '环境条件非常适宜作物生长',
          class: 'level-excellent'
        };
      } else if (score >= 75) {
        return {
          text: '良好',
          desc: '环境条件适宜，可持续监测',
          class: 'level-good'
        };
      } else if (score >= 60) {
        return {
          text: '一般',
          desc: '部分指标需要调整优化',
          class: 'level-normal'
        };
      } else {
        return {
          text: '需改善',
          desc: '环境条件需要及时调整',
          class: 'level-poor'
        };
      }
    },
    
    // 获取健康评分渐变色
    getHealthScoreGradient(farm) {
      const score = this.calculateHealthScore(farm);
      
      if (score >= 90) {
        return 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)';
      } else if (score >= 75) {
        return 'linear-gradient(135deg, #43e97b 0%, #38f9d7 100%)';
      } else if (score >= 60) {
        return 'linear-gradient(135deg, #fa709a 0%, #fee140 100%)';
      } else {
        return 'linear-gradient(135deg, #f093fb 0%, #f5576c 100%)';
      }
    },
    
    // 获取环境指标数据
    getEnvMetrics(farm) {
      if (!farm) return [];
      
      return [
        {
          key: 'temp',
          label: '温度',
          value: `${farm.temperature}°C`,
          icon: '🌡️',
          percent: this.normalizeValue(farm.temperature, 0, 40, 100),
          color: this.getMetricColor(farm.temperature, 20, 28, 15, 32),
          status: this.getMetricStatus(farm.temperature, 20, 28, 15, 32),
          statusColor: this.getMetricStatusColor(farm.temperature, 20, 28, 15, 32)
        },
        {
          key: 'airHum',
          label: '空气湿度',
          value: `${farm.airhumidity}%`,
          icon: '💧',
          percent: farm.airhumidity,
          color: this.getMetricColor(farm.airhumidity, 60, 80, 50, 90),
          status: this.getMetricStatus(farm.airhumidity, 60, 80, 50, 90),
          statusColor: this.getMetricStatusColor(farm.airhumidity, 60, 80, 50, 90)
        },
        {
          key: 'soilHum',
          label: '土壤湿度',
          value: `${farm.soilhumidity}%`,
          icon: '🌱',
          percent: farm.soilhumidity,
          color: this.getMetricColor(farm.soilhumidity, 40, 70, 30, 80),
          status: this.getMetricStatus(farm.soilhumidity, 40, 70, 30, 80),
          statusColor: this.getMetricStatusColor(farm.soilhumidity, 40, 70, 30, 80)
        },
        {
          key: 'light',
          label: '光照',
          value: `${(farm.light / 1000).toFixed(1)}k`,
          icon: '☀️',
          percent: this.normalizeValue(farm.light, 0, 60000, 100),
          color: this.getMetricColor(farm.light, 20000, 50000, 10000, 60000),
          status: this.getMetricStatus(farm.light, 20000, 50000, 10000, 60000),
          statusColor: this.getMetricStatusColor(farm.light, 20000, 50000, 10000, 60000)
        },
        {
          key: 'carbon',
          label: 'CO₂',
          value: `${farm.carbon}ppm`,
          icon: '🌫️',
          percent: this.normalizeValue(farm.carbon, 300, 500, 100),
          color: this.getMetricColor(farm.carbon, 350, 450, 300, 500),
          status: this.getMetricStatus(farm.carbon, 350, 450, 300, 500),
          statusColor: this.getMetricStatusColor(farm.carbon, 350, 450, 300, 500)
        },
        {
          key: 'ph',
          label: 'PH值',
          value: farm.ph,
          icon: '⚗️',
          percent: this.normalizeValue(farm.ph, 5, 8, 100),
          color: this.getMetricColor(farm.ph, 6.0, 7.5, 5.5, 8.0),
          status: this.getMetricStatus(farm.ph, 6.0, 7.5, 5.5, 8.0),
          statusColor: this.getMetricStatusColor(farm.ph, 6.0, 7.5, 5.5, 8.0)
        }
      ];
    },
    
    // 标准化数值到百分比
    normalizeValue(value, min, max, scale = 100) {
      return Math.min(Math.max(((value - min) / (max - min)) * scale, 0), scale);
    },
    
    // 获取指标颜色
    getMetricColor(value, idealMin, idealMax, acceptMin, acceptMax) {
      if (value >= idealMin && value <= idealMax) {
        return '#67C23A'; // 绿色 - 理想
      } else if (value >= acceptMin && value <= acceptMax) {
        return '#E6A23C'; // 橙色 - 可接受
      } else {
        return '#F56C6C'; // 红色 - 需改善
      }
    },
    
    // 获取指标状态文本
    getMetricStatus(value, idealMin, idealMax, acceptMin, acceptMax) {
      if (value >= idealMin && value <= idealMax) {
        return '理想';
      } else if (value >= acceptMin && value <= acceptMax) {
        return '正常';
      } else {
        return '预警';
      }
    },
    
    // 获取指标状态颜色
    getMetricStatusColor(value, idealMin, idealMax, acceptMin, acceptMax) {
      if (value >= idealMin && value <= idealMax) {
        return '#67C23A';
      } else if (value >= acceptMin && value <= acceptMax) {
        return '#E6A23C';
      } else {
        return '#F56C6C';
      }
    },
    
    // 计算SVG圆环的dasharray
    calculateDashArray(percent) {
      const circumference = 2 * Math.PI * 32; // 半径32
      const filled = (percent / 100) * circumference;
      return `${filled} ${circumference}`;
    },
    
    // 获取智能建议
    getSmartSuggestions(farm) {
      if (!farm) return [];
      
      const suggestions = [];
      
      // 温度建议
      if (farm.temperature < 20) {
        suggestions.push({
          icon: '🌡️',
          text: '温度偏低',
          action: '建议增加保温措施或调整种植时间',
          type: 'warning'
        });
      } else if (farm.temperature > 28) {
        suggestions.push({
          icon: '🌡️',
          text: '温度偏高',
          action: '建议增加遮阳网或加强通风降温',
          type: 'warning'
        });
      }
      
      // 湿度建议
      if (farm.soilhumidity < 40) {
        suggestions.push({
          icon: '💧',
          text: '土壤湿度不足',
          action: '建议及时灌溉，保持土壤湿润',
          type: 'danger'
        });
      } else if (farm.soilhumidity > 70) {
        suggestions.push({
          icon: '💧',
          text: '土壤湿度过高',
          action: '建议加强排水，防止根系腐烂',
          type: 'warning'
        });
      }
      
      // 光照建议
      if (farm.light < 20000) {
        suggestions.push({
          icon: '☀️',
          text: '光照不足',
          action: '建议补充人工光源或调整种植密度',
          type: 'info'
        });
      }
      
      // PH值建议
      if (farm.ph < 6.0) {
        suggestions.push({
          icon: '⚗️',
          text: 'PH值偏酸',
          action: '建议施用石灰或草木灰调节土壤酸碱度',
          type: 'warning'
        });
      } else if (farm.ph > 7.5) {
        suggestions.push({
          icon: '⚗️',
          text: 'PH值偏碱',
          action: '建议施用硫磺粉或有机肥调节',
          type: 'warning'
        });
      }
      
      // 如果一切正常
      if (suggestions.length === 0) {
        suggestions.push({
          icon: '✅',
          text: '环境条件优良',
          action: '继续保持当前管理措施',
          type: 'success'
        });
      }
      
      return suggestions;
    },
    
    highlightFarmBlock(farm) {
      if (!farm || this.farmBlocks.length === 0) return;
      
      // 找到对应的农田组
      const group = this.farmBlocks.find(g => g.userData.farm && g.userData.farm.id === farm.id);
      if (group) {
        this.selectBlock(group);
      }
    },
    
    init3DMap() {
      const container = this.$refs.threeContainer;
      if (!container) {
        console.warn('3D容器未找到');
        return;
      }
      if (this.farms.length === 0) {
        console.warn('没有农田数据，无法初始化3D地图');
        return;
      }
      
      // 如果已经初始化过，先清理
      if (this.renderer) {
        this.renderer.dispose();
        if (container.contains(this.renderer.domElement)) {
          container.removeChild(this.renderer.domElement);
        }
      }
      
      const width = container.clientWidth;
      const height = container.clientHeight;
      
      // 创建场景
      this.scene = new THREE.Scene();
      
      // 创建相机
      this.camera = new THREE.PerspectiveCamera(60, width / height, 0.1, 1000);
      this.camera.position.set(40, 50, 40);
      this.camera.lookAt(0, 0, 0);
      
      // 创建渲染器 - 优化性能设置
      this.renderer = new THREE.WebGLRenderer({ 
        antialias: false, // 关闭抗锯齿提升性能
        powerPreference: 'high-performance' // 优先性能
      });
      this.renderer.setSize(width, height);
      // 限制像素比，避免高DPI设备过度渲染
      this.renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
      this.renderer.shadowMap.enabled = true;
      this.renderer.shadowMap.type = THREE.PCFSoftShadowMap; // 使用软阴影
      container.appendChild(this.renderer.domElement);
      
      // 添加轨道控制器 - 优化配置
      this.controls = new OrbitControls(this.camera, this.renderer.domElement);
      this.controls.enableDamping = true;
      this.controls.dampingFactor = 0.05;
      this.controls.minDistance = 20;
      this.controls.maxDistance = 150;
      this.controls.maxPolarAngle = Math.PI / 2.2;
      this.controls.enablePan = true; // 允许平移
      this.controls.enableZoom = true; // 允许缩放
      this.controls.enableRotate = true; // 允许旋转
      this.controls.rotateSpeed = 0.5; // 降低旋转速度，提升操作体验
      this.controls.zoomSpeed = 0.8; // 降低缩放速度
      this.controls.panSpeed = 0.8; // 降低平移速度
      
      // 添加光源系统
      // 1. 环境光（模拟天空散射光）
      const ambientLight = new THREE.AmbientLight(0xB0C4DE, 0.6);
      this.scene.add(ambientLight);
      
      // 2. 主光源（太阳光）- 降低阴影分辨率提升性能
      const sunLight = new THREE.DirectionalLight(0xFFF8DC, 1.2);
      sunLight.position.set(60, 100, 40);
      sunLight.castShadow = true;
      sunLight.shadow.mapSize.width = 1024; // 降低阴影分辨率：2048 -> 1024
      sunLight.shadow.mapSize.height = 1024;
      sunLight.shadow.camera.near = 0.5;
      sunLight.shadow.camera.far = 250;
      sunLight.shadow.camera.left = -100;
      sunLight.shadow.camera.right = 100;
      sunLight.shadow.camera.top = 100;
      sunLight.shadow.camera.bottom = -100;
      sunLight.shadow.bias = -0.0001;
      sunLight.shadow.radius = 4; // 添加阴影模糊半径，减少锯齿
      this.scene.add(sunLight);
      
      // 3. 半球光（天空和地面的颜色过渡）
      const hemisphereLight = new THREE.HemisphereLight(0x87CEEB, 0x6B5D4F, 0.5);
      this.scene.add(hemisphereLight);
      
      this.createGround();
      
      // 创建农田方块
      this.createFarmBlocks();
      
      // 初始化射线检测
      this.raycaster = new THREE.Raycaster();
      this.mouse = new THREE.Vector2();
      
      // 绑定事件处理函数（保存引用以便后续清理）
      this.boundMouseClick = this.onMouseClick.bind(this);
      this.boundMouseMove = this.onMouseMove.bind(this);
      this.boundWindowResize = this.onWindowResize.bind(this);
      
      // 添加鼠标事件
      container.addEventListener('click', this.boundMouseClick);
      container.addEventListener('mousemove', this.boundMouseMove, { passive: true }); // 使用passive提升性能
      
      // 窗口大小调整
      window.addEventListener('resize', this.boundWindowResize);
      
      // 开始动画循环
      this.animate();
    },
    
    createSoilTexture() {
      // 创建土壤纹理 - 优化纹理生成性能
      const canvas = document.createElement('canvas');
      canvas.width = 256; // 降低纹理分辨率：512 -> 256
      canvas.height = 256;
      const ctx = canvas.getContext('2d');
      
      // 基础土壤颜色
      const soilColors = [
        '#5D4E37', // 深棕
        '#6B5D4F', // 中棕
        '#7A6A52', // 浅棕
        '#8B7355', // 土黄
        '#4A3C2A'  // 暗棕
      ];
      
      // 填充基础颜色
      ctx.fillStyle = soilColors[1];
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      
      // 减少颗粒数量提升性能：5000 -> 1500
      for (let i = 0; i < 1500; i++) {
        const x = Math.random() * canvas.width;
        const y = Math.random() * canvas.height;
        const size = Math.random() * 2 + 0.5; // 稍微减小颗粒大小
        const colorIndex = Math.floor(Math.random() * soilColors.length);
        
        ctx.fillStyle = soilColors[colorIndex];
        ctx.globalAlpha = Math.random() * 0.5 + 0.3; // 降低透明度变化范围
        ctx.beginPath();
        ctx.arc(x, y, size, 0, Math.PI * 2);
        ctx.fill();
      }
      
      // 添加土壤裂纹
      ctx.globalAlpha = 0.3;
      ctx.strokeStyle = '#4A3C2A';
      for (let i = 0; i < 35; i++) {
        ctx.lineWidth = Math.random() * 2 + 0.5;
        ctx.beginPath();
        const startX = Math.random() * canvas.width;
        const startY = Math.random() * canvas.height;
        ctx.moveTo(startX, startY);
        
        const segments = Math.floor(Math.random() * 4) + 3;
        for (let j = 0; j < segments; j++) {
          const endX = startX + (Math.random() - 0.5) * 70;
          const endY = startY + (Math.random() - 0.5) * 70;
          ctx.lineTo(endX, endY);
        }
        ctx.stroke();
      }
      
      // 添加小石子
      ctx.globalAlpha = 0.7;
      for (let i = 0; i < 150; i++) {
        const x = Math.random() * canvas.width;
        const y = Math.random() * canvas.height;
        const size = Math.random() * 4 + 1;
        
        ctx.fillStyle = Math.random() > 0.5 ? '#9B8B7E' : '#8A7968';
        ctx.beginPath();
        ctx.arc(x, y, size, 0, Math.PI * 2);
        ctx.fill();
      }
      
      ctx.globalAlpha = 1.0;
      
      return new THREE.CanvasTexture(canvas);
    },
    
    createGround() {
      // 创建地面（带起伏的土壤纹理）- 降低分段数提升性能
      const groundGeometry = new THREE.PlaneGeometry(200, 200, 40, 40); // 降低分段数：80x80 -> 40x40
      const positions = groundGeometry.attributes.position;
      
      // 添加更自然的地形起伏（优化循环）
      const positionArray = positions.array;
      for (let i = 0; i < positionArray.length; i += 3) {
        const x = positionArray[i];
        const y = positionArray[i + 1];
        const noise = Math.sin(x * 0.05) * Math.cos(y * 0.05) * 0.5 + 
                     Math.sin(x * 0.2) * Math.cos(y * 0.2) * 0.2;
        positionArray[i + 2] = noise;
      }
      positions.needsUpdate = true;
      groundGeometry.computeVertexNormals();
      
      // 创建土壤纹理
      const soilTexture = this.createSoilTexture();
      soilTexture.wrapS = THREE.RepeatWrapping;
      soilTexture.wrapT = THREE.RepeatWrapping;
      soilTexture.repeat.set(18, 18);
      
      // 创建带纹理的土壤材质
      const groundMaterial = new THREE.MeshStandardMaterial({ 
        map: soilTexture,
        roughness: 0.95,
        metalness: 0.05,
        flatShading: false
      });
      
      const ground = new THREE.Mesh(groundGeometry, groundMaterial);
      ground.rotation.x = -Math.PI / 2;
      ground.receiveShadow = true;
      this.scene.add(ground);
      
      // 添加天空颜色（渐变背景）
      this.scene.background = new THREE.Color(0x87CEEB); // 天空蓝
      
      // 添加雾效果，增加景深
      this.scene.fog = new THREE.Fog(0x87CEEB, 50, 200);
    },
    
    createFarmBlocks() {
      // 清空之前的方块和标签
      this.farmBlocks.forEach(block => {
        this.scene.remove(block);
      });
      this.farmBlocks = [];
      
      // 清空之前的标签
      this.labels.forEach(label => {
        this.scene.remove(label);
      });
      this.labels = [];
      
      // 根据农田数量动态计算布局
      const farmCount = this.farms.length;
      const cols = Math.ceil(Math.sqrt(farmCount));
      const spacing = 15;
      
      this.farms.forEach((farm, index) => {
        const row = Math.floor(index / cols);
        const col = index % cols;
        
        // 计算位置（居中排列）
        const offsetX = (cols - 1) * spacing / 2;
        const offsetZ = (Math.ceil(farmCount / cols) - 1) * spacing / 2;
        const x = col * spacing - offsetX;
        const z = row * spacing - offsetZ;
        
        // 根据面积计算地块大小
        const area = parseFloat(farm.area) || 10;
        const size = Math.sqrt(area) * 0.8;
        
        // 创建农田地块组
        const farmGroup = new THREE.Group();
        
        // 1. 创建农田地面（平铺，略高于地面）- 降低分段数提升性能
        const fieldGeometry = new THREE.PlaneGeometry(size, size, 4, 4); // 降低分段数：8x8 -> 4x4
        
        // 添加作物纹理起伏（模拟作物生长）
        const fieldPositions = fieldGeometry.attributes.position;
        for (let i = 0; i < fieldPositions.count; i++) {
          const fx = fieldPositions.getX(i);
          const fy = fieldPositions.getY(i);
          const cropHeight = Math.sin(fx * 2) * Math.cos(fy * 2) * 0.2 + 0.3;
          fieldPositions.setZ(i, cropHeight);
        }
        fieldGeometry.computeVertexNormals();
        
        // 根据作物类型设置基础颜色（从配置中心获取）
        const config = this.cropConfig[farm.crop] || this.defaultCropConfig;
        const baseColor = config.color;
        
        // 根据健康指数动态调整颜色
        const healthScore = this.calculateHealthScore(farm);
        const healthColor = this.getHealthColor(healthScore);
        
        // 混合基础颜色和健康颜色（健康指数影响30%）
        const finalColor = this.blendColors(baseColor, healthColor, 0.3);
        
        const fieldMaterial = new THREE.MeshStandardMaterial({
          color: finalColor,
          roughness: 0.7,
          metalness: 0.1,
          flatShading: false,
          side: THREE.DoubleSide
        });
        
        const field = new THREE.Mesh(fieldGeometry, fieldMaterial);
        field.rotation.x = -Math.PI / 2;
        field.position.y = 0.3;
        field.receiveShadow = true;
        field.castShadow = true;
        farmGroup.add(field);
        
        // 2. 创建田埂边框（棕色土壤）
        const borderHeight = 0.3;
        const borderWidth = 0.2;
        const borderMaterial = new THREE.MeshStandardMaterial({
          color: 0x8B6F47,
          roughness: 0.9,
          metalness: 0.1
        });
        
        // 四条田埂
        const borders = [
          { w: size + borderWidth * 2, h: borderWidth, x: 0, z: -size / 2 - borderWidth / 2 }, // 前
          { w: size + borderWidth * 2, h: borderWidth, x: 0, z: size / 2 + borderWidth / 2 },  // 后
          { w: borderWidth, h: size, x: -size / 2 - borderWidth / 2, z: 0 },                    // 左
          { w: borderWidth, h: size, x: size / 2 + borderWidth / 2, z: 0 }                      // 右
        ];
        
        borders.forEach(b => {
          const borderGeom = new THREE.BoxGeometry(b.w, borderHeight, b.h);
          const border = new THREE.Mesh(borderGeom, borderMaterial);
          border.position.set(b.x, borderHeight / 2, b.z);
          border.castShadow = true;
          border.receiveShadow = true;
          farmGroup.add(border);
        });
        
        // 3. 设置整个农田组的位置
        farmGroup.position.set(x, 0, z);
        
        // 存储农田数据
        farmGroup.userData = {
          farm: farm,
          originalColor: baseColor,
          healthScore: healthScore,
          healthColor: healthColor
        };
        
        this.scene.add(farmGroup);
        this.farmBlocks.push(farmGroup);
        
        // 添加 3D 植物模型
        this.addCropModels(farm.crop, x, z, size, baseColor);
        
        // 添加作物标签（显示作物类型）
        this.addCropLabel(farm.crop, x, 2.5, z, baseColor);
        
        // 添加健康指数标签
        this.addHealthIndexLabel(healthScore, x, 3.0, z);
        
        // 添加地块名称标签（小而简洁）
        this.addFieldNameLabel(farm.farm, x, 3.5, z);
        
        // 添加地址标签
        if (farm.address) {
          this.addAddressLabel(farm.address, x, 4.2, z);
        }
      });
    },
    
    // 加载外部 3D 模型（支持缓存）
    async loadCropModel(cropType) {
      // 如果已缓存，直接返回
      if (this.modelCache[cropType]) {
        return this.modelCache[cropType].clone();
      }
      
      // 从配置中心获取模型路径
      const config = this.cropConfig[cropType] || this.defaultCropConfig;
      const modelPath = config.modelPath;
      if (!modelPath) {
        console.warn(`未找到 ${cropType} 的模型路径，使用默认几何体`);
        return null;
      }
      
      // 初始化加载器
      if (!this.gltfLoader) {
        this.gltfLoader = new GLTFLoader();
      }
      
      try {
        // 加载模型
        const gltf = await new Promise((resolve, reject) => {
          this.gltfLoader.load(
            modelPath,
            (gltf) => resolve(gltf),
            undefined,
            (error) => reject(error)
          );
        });
        
        // 缓存模型
        this.modelCache[cropType] = gltf.scene;
        console.log(`✅ 成功加载 ${cropType} 模型`);
        
        return gltf.scene.clone();
      } catch (error) {
        console.warn(`⚠️ 加载 ${cropType} 模型失败:`, error.message);
        return null;
      }
    },
    
    // 添加 3D 植物模型（优先使用外部模型，降级到几何体）
    async addCropModels(cropType, x, z, fieldSize, cropColor) {
      // 优化植物数量：根据面积动态调整，但限制最大值
      const baseCount = Math.floor(fieldSize * 1.5); // 降低密度：3 -> 1.5
      const maxCount = 20; // 限制最大植物数量，避免性能问题
      const plantCount = Math.min(baseCount, maxCount);
      const spacing = fieldSize / Math.max(plantCount, 1);
      
      // 尝试加载外部模型
      const externalModel = await this.loadCropModel(cropType);
      
      for (let i = 0; i < plantCount; i++) {
        for (let j = 0; j < plantCount; j++) {
          const offsetX = (i - plantCount / 2) * spacing + (Math.random() - 0.5) * spacing * 0.3;
          const offsetZ = (j - plantCount / 2) * spacing + (Math.random() - 0.5) * spacing * 0.3;
          
          const plantX = x + offsetX;
          const plantZ = z + offsetZ;
          
          let plant;
          
          // 如果有外部模型，使用外部模型
          if (externalModel) {
            plant = externalModel.clone();
            // 为外部模型启用阴影（仅部分植物投射阴影以提升性能）
            const enableShadow = (i + j) % 3 === 0; // 只有1/3的植物投射阴影
            plant.traverse((child) => {
              if (child.isMesh) {
                child.castShadow = enableShadow;
                child.receiveShadow = false; // 植物不接收阴影
              }
            });
          } else {
            // 降级：使用几何体创建植物
            switch(cropType) {
              case '小麦':
                plant = this.createWheatPlant(cropColor);
                break;
              case '玉米':
                plant = this.createCornPlant(cropColor);
                break;
              case '水稻':
                plant = this.createRicePlant(cropColor);
                break;
              case '大豆':
              case '白菜':
              case '黄瓜':
                plant = this.createLeafyPlant(cropColor);
                break;
              case '番茄':
              case '辣椒':
                plant = this.createVinePlant(cropColor);
                break;
              default:
                plant = this.createGenericPlant(cropColor);
            }
          }
          
          if (plant) {
            plant.position.set(plantX, 0.5, plantZ);
            // 减少随机缩放范围以提升性能
            const scale = 0.9 + Math.random() * 0.2; // 统一缩放值
            plant.scale.set(scale, scale, scale);
            // 仅部分植物投射阴影
            const enableShadow = (i + j) % 3 === 0;
            plant.traverse((child) => {
              if (child.isMesh) {
                child.castShadow = enableShadow;
                child.receiveShadow = false;
              }
            });
            this.scene.add(plant);
          }
        }
      }
    },
    
    // 创建小麦植物（细长的茎）
    createWheatPlant(color) {
      const group = new THREE.Group();
      
      // 茎秆
      const stemGeometry = new THREE.CylinderGeometry(0.02, 0.03, 1.2, 6);
      const stemMaterial = new THREE.MeshStandardMaterial({ 
        color: 0x8B7355,
        roughness: 0.8 
      });
      const stem = new THREE.Mesh(stemGeometry, stemMaterial);
      stem.position.y = 0.6;
      stem.castShadow = true;
      group.add(stem);
      
      // 麦穗（顶部）
      const headGeometry = new THREE.ConeGeometry(0.15, 0.4, 8);
      const headMaterial = new THREE.MeshStandardMaterial({ 
        color: color,
        roughness: 0.6 
      });
      const head = new THREE.Mesh(headGeometry, headMaterial);
      head.position.y = 1.4;
      head.castShadow = true;
      group.add(head);
      
      return group;
    },
    
    // 创建玉米植物（粗壮的茎 + 大叶子）
    createCornPlant(color) {
      const group = new THREE.Group();
      
      // 粗茎
      const stemGeometry = new THREE.CylinderGeometry(0.05, 0.08, 1.8, 8);
      const stemMaterial = new THREE.MeshStandardMaterial({ 
        color: 0x7CB342,
        roughness: 0.7 
      });
      const stem = new THREE.Mesh(stemGeometry, stemMaterial);
      stem.position.y = 0.9;
      stem.castShadow = true;
      group.add(stem);
      
      // 叶子
      const leafGeometry = new THREE.PlaneGeometry(0.3, 0.8);
      const leafMaterial = new THREE.MeshStandardMaterial({ 
        color: 0x558B2F,
        side: THREE.DoubleSide,
        roughness: 0.8 
      });
      
      for (let i = 0; i < 4; i++) {
        const leaf = new THREE.Mesh(leafGeometry, leafMaterial);
        leaf.position.y = 0.8 + i * 0.3;
        leaf.rotation.y = (Math.PI / 2) * i;
        leaf.rotation.z = Math.PI / 6;
        leaf.castShadow = true;
        group.add(leaf);
      }
      
      // 玉米棒
      const cornGeometry = new THREE.CylinderGeometry(0.1, 0.1, 0.4, 8);
      const cornMaterial = new THREE.MeshStandardMaterial({ 
        color: color,
        roughness: 0.6 
      });
      const corn = new THREE.Mesh(cornGeometry, cornMaterial);
      corn.position.set(0.15, 1.2, 0);
      corn.rotation.z = Math.PI / 4;
      corn.castShadow = true;
      group.add(corn);
      
      return group;
    },
    
    // 创建水稻植物（弯曲的茎）
    createRicePlant(color) {
      const group = new THREE.Group();
      
      // 茎秆
      const stemGeometry = new THREE.CylinderGeometry(0.02, 0.03, 1.0, 6);
      const stemMaterial = new THREE.MeshStandardMaterial({ 
        color: 0x7CB342,
        roughness: 0.8 
      });
      const stem = new THREE.Mesh(stemGeometry, stemMaterial);
      stem.position.y = 0.5;
      stem.castShadow = true;
      group.add(stem);
      
      // 稻穗（弯曲的）
      const riceGeometry = new THREE.SphereGeometry(0.12, 8, 8);
      const riceMaterial = new THREE.MeshStandardMaterial({ 
        color: color,
        roughness: 0.6 
      });
      
      for (let i = 0; i < 5; i++) {
        const rice = new THREE.Mesh(riceGeometry, riceMaterial);
        rice.position.set(
          Math.sin(i * 0.3) * 0.15,
          1.0 - i * 0.1,
          Math.cos(i * 0.3) * 0.15
        );
        rice.scale.set(1, 1.5, 1);
        rice.castShadow = true;
        group.add(rice);
      }
      
      return group;
    },
    
    // 创建叶菜类植物（大豆、白菜等）
    createLeafyPlant(color) {
      const group = new THREE.Group();
      
      // 多层叶子
      const leafGeometry = new THREE.CircleGeometry(0.2, 8);
      const leafMaterial = new THREE.MeshStandardMaterial({ 
        color: color,
        side: THREE.DoubleSide,
        roughness: 0.7 
      });
      
      for (let i = 0; i < 6; i++) {
        const leaf = new THREE.Mesh(leafGeometry, leafMaterial);
        leaf.position.y = 0.3 + i * 0.1;
        leaf.rotation.y = (Math.PI / 3) * i;
        leaf.rotation.x = -Math.PI / 2 + Math.PI / 8;
        leaf.scale.set(1 - i * 0.1, 1 - i * 0.1, 1);
        leaf.castShadow = true;
        group.add(leaf);
      }
      
      return group;
    },
    
    // 创建藤蔓类植物（番茄、辣椒等）
    createVinePlant(color) {
      const group = new THREE.Group();
      
      // 主茎
      const stemGeometry = new THREE.CylinderGeometry(0.03, 0.04, 1.2, 6);
      const stemMaterial = new THREE.MeshStandardMaterial({ 
        color: 0x558B2F,
        roughness: 0.8 
      });
      const stem = new THREE.Mesh(stemGeometry, stemMaterial);
      stem.position.y = 0.6;
      stem.castShadow = true;
      group.add(stem);
      
      // 叶子
      const leafGeometry = new THREE.CircleGeometry(0.15, 6);
      const leafMaterial = new THREE.MeshStandardMaterial({ 
        color: 0x4CAF50,
        side: THREE.DoubleSide,
        roughness: 0.7 
      });
      
      for (let i = 0; i < 4; i++) {
        const leaf = new THREE.Mesh(leafGeometry, leafMaterial);
        leaf.position.set(
          Math.cos(i * Math.PI / 2) * 0.2,
          0.5 + i * 0.2,
          Math.sin(i * Math.PI / 2) * 0.2
        );
        leaf.rotation.y = i * Math.PI / 2;
        leaf.castShadow = true;
        group.add(leaf);
      }
      
      // 果实
      const fruitGeometry = new THREE.SphereGeometry(0.1, 8, 8);
      const fruitMaterial = new THREE.MeshStandardMaterial({ 
        color: color,
        roughness: 0.4,
        metalness: 0.1 
      });
      
      for (let i = 0; i < 3; i++) {
        const fruit = new THREE.Mesh(fruitGeometry, fruitMaterial);
        fruit.position.set(
          Math.cos(i * Math.PI * 2 / 3) * 0.15,
          0.8 + Math.random() * 0.3,
          Math.sin(i * Math.PI * 2 / 3) * 0.15
        );
        fruit.castShadow = true;
        group.add(fruit);
      }
      
      return group;
    },
    
    // 创建通用植物
    createGenericPlant(color) {
      const group = new THREE.Group();
      
      const stemGeometry = new THREE.CylinderGeometry(0.03, 0.04, 0.8, 6);
      const stemMaterial = new THREE.MeshStandardMaterial({ 
        color: 0x7CB342,
        roughness: 0.8 
      });
      const stem = new THREE.Mesh(stemGeometry, stemMaterial);
      stem.position.y = 0.4;
      stem.castShadow = true;
      group.add(stem);
      
      const topGeometry = new THREE.SphereGeometry(0.15, 8, 8);
      const topMaterial = new THREE.MeshStandardMaterial({ 
        color: color,
        roughness: 0.6 
      });
      const top = new THREE.Mesh(topGeometry, topMaterial);
      top.position.y = 0.9;
      top.castShadow = true;
      group.add(top);
      
      return group;
    },
    
    // 添加简洁的地块名称标签
    addFieldNameLabel(farmName, x, y, z) {
      const canvas = document.createElement('canvas');
      const context = canvas.getContext('2d');
      canvas.width = 160;
      canvas.height = 40;
      
      // 半透明背景
      context.fillStyle = 'rgba(255, 255, 255, 0.88)';
      context.shadowColor = 'rgba(0, 0, 0, 0.15)';
      context.shadowBlur = 5;
      context.shadowOffsetX = 1;
      context.shadowOffsetY = 1;
      
      const radius = 6;
      context.beginPath();
      context.moveTo(radius, 0);
      context.lineTo(canvas.width - radius, 0);
      context.quadraticCurveTo(canvas.width, 0, canvas.width, radius);
      context.lineTo(canvas.width, canvas.height - radius);
      context.quadraticCurveTo(canvas.width, canvas.height, canvas.width - radius, canvas.height);
      context.lineTo(radius, canvas.height);
      context.quadraticCurveTo(0, canvas.height, 0, canvas.height - radius);
      context.lineTo(0, radius);
      context.quadraticCurveTo(0, 0, radius, 0);
      context.closePath();
      context.fill();
      
      context.shadowColor = 'transparent';
      
      // 地块名称
      context.font = 'Bold 16px Arial';
      context.fillStyle = '#606266';
      context.textAlign = 'center';
      context.textBaseline = 'middle';
      context.fillText(farmName, canvas.width / 2, canvas.height / 2);
      
      const texture = new THREE.CanvasTexture(canvas);
      const spriteMaterial = new THREE.SpriteMaterial({ 
        map: texture,
        sizeAttenuation: true
      });
      const sprite = new THREE.Sprite(spriteMaterial);
      sprite.position.set(x, y, z);
      sprite.scale.set(2.8, 0.7, 1);
      
      this.scene.add(sprite);
    },
    
    // 添加地址标签
    addAddressLabel(address, x, y, z) {
      const canvas = document.createElement('canvas');
      const context = canvas.getContext('2d');
      canvas.width = 200;
      canvas.height = 35;
      
      // 半透明背景（浅蓝色）
      context.fillStyle = 'rgba(230, 244, 255, 0.9)';
      context.shadowColor = 'rgba(0, 0, 0, 0.12)';
      context.shadowBlur = 4;
      context.shadowOffsetX = 1;
      context.shadowOffsetY = 1;
      
      const radius = 5;
      context.beginPath();
      context.moveTo(radius, 0);
      context.lineTo(canvas.width - radius, 0);
      context.quadraticCurveTo(canvas.width, 0, canvas.width, radius);
      context.lineTo(canvas.width, canvas.height - radius);
      context.quadraticCurveTo(canvas.width, canvas.height, canvas.width - radius, canvas.height);
      context.lineTo(radius, canvas.height);
      context.quadraticCurveTo(0, canvas.height, 0, canvas.height - radius);
      context.lineTo(0, radius);
      context.quadraticCurveTo(0, 0, radius, 0);
      context.closePath();
      context.fill();
      
      // 添加边框
      context.strokeStyle = '#409EFF';
      context.lineWidth = 1.5;
      context.stroke();
      
      context.shadowColor = 'transparent';
      
      // 绘制地址图标和文字
      context.font = '13px Arial';
      context.fillStyle = '#409EFF';
      context.textAlign = 'left';
      context.textBaseline = 'middle';
      
      // 地址图标
      context.fillText('📍', 10, canvas.height / 2);
      
      // 地址文字（如果太长则截断）
      context.fillStyle = '#606266';
      const maxWidth = canvas.width - 40;
      let displayAddress = address;
      if (context.measureText(address).width > maxWidth) {
        while (context.measureText(displayAddress + '...').width > maxWidth && displayAddress.length > 0) {
          displayAddress = displayAddress.slice(0, -1);
        }
        displayAddress += '...';
      }
      context.fillText(displayAddress, 30, canvas.height / 2);
      
      const texture = new THREE.CanvasTexture(canvas);
      const spriteMaterial = new THREE.SpriteMaterial({ 
        map: texture,
        sizeAttenuation: true,
        transparent: true
      });
      const sprite = new THREE.Sprite(spriteMaterial);
      sprite.position.set(x, y, z);
      sprite.scale.set(3.2, 0.56, 1);
      
      this.scene.add(sprite);
    },
    
    // 添加作物类型标签
    addCropLabel(cropName, x, y, z, cropColor) {
      const canvas = document.createElement('canvas');
      const context = canvas.getContext('2d');
      canvas.width = 160;
      canvas.height = 50;
      
      // 绘制背景卡片（带边框）
      context.fillStyle = 'rgba(255, 255, 255, 0.92)';
      context.shadowColor = 'rgba(0, 0, 0, 0.2)';
      context.shadowBlur = 6;
      context.shadowOffsetX = 1;
      context.shadowOffsetY = 1;
      
      // 圆角矩形
      const radius = 8;
      context.beginPath();
      context.moveTo(radius, 0);
      context.lineTo(canvas.width - radius, 0);
      context.quadraticCurveTo(canvas.width, 0, canvas.width, radius);
      context.lineTo(canvas.width, canvas.height - radius);
      context.quadraticCurveTo(canvas.width, canvas.height, canvas.width - radius, canvas.height);
      context.lineTo(radius, canvas.height);
      context.quadraticCurveTo(0, canvas.height, 0, canvas.height - radius);
      context.lineTo(0, radius);
      context.quadraticCurveTo(0, 0, radius, 0);
      context.closePath();
      context.fill();
      
      // 绘制边框
      context.strokeStyle = '#' + cropColor.toString(16).padStart(6, '0');
      context.lineWidth = 2;
      context.stroke();
      
      // 重置阴影
      context.shadowColor = 'transparent';
      context.shadowBlur = 0;
      
      // 绘制作物图标和名称
      context.font = 'Bold 20px Arial';
      context.fillStyle = '#' + cropColor.toString(16).padStart(6, '0');
      context.textAlign = 'center';
      context.textBaseline = 'middle';
      
      // 从配置中心获取作物图标
      const config = this.cropConfig[cropName] || this.defaultCropConfig;
      const icon = config.icon;
      context.fillText(icon + ' ' + cropName, canvas.width / 2, canvas.height / 2);
      
      const texture = new THREE.CanvasTexture(canvas);
      const spriteMaterial = new THREE.SpriteMaterial({ 
        map: texture,
        sizeAttenuation: true,
        transparent: true
      });
      const sprite = new THREE.Sprite(spriteMaterial);
      sprite.position.set(x, y, z);
      sprite.scale.set(2.8, 0.9, 1);
      
      this.scene.add(sprite);
    },
    
    addTextLabel(text, x, y, z) {
      const canvas = document.createElement('canvas');
      const context = canvas.getContext('2d');
      canvas.width = 256;
      canvas.height = 64;
      
      context.fillStyle = 'rgba(255, 255, 255, 0.9)';
      context.fillRect(0, 0, canvas.width, canvas.height);
      
      context.font = 'Bold 20px Arial';
      context.fillStyle = '#303133';
      context.textAlign = 'center';
      context.textBaseline = 'middle';
      context.fillText(text, canvas.width / 2, canvas.height / 2);
      
      const texture = new THREE.CanvasTexture(canvas);
      const spriteMaterial = new THREE.SpriteMaterial({ map: texture });
      const sprite = new THREE.Sprite(spriteMaterial);
      sprite.position.set(x, y, z);
      sprite.scale.set(6, 1.5, 1);
      
      this.scene.add(sprite);
    },
    
    onMouseClick(event) {
      const container = this.$refs.threeContainer;
      if (!container || !this.camera || !this.raycaster) return;
      
      const rect = container.getBoundingClientRect();
      
      this.mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
      this.mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;
      
      this.raycaster.setFromCamera(this.mouse, this.camera);
      
      // 优化：直接检测农田地块组，使用recursive参数
      const intersects = this.raycaster.intersectObjects(this.farmBlocks, true);
      
      if (intersects.length > 0) {
        // 找到对应的农田组
        let clickedGroup = intersects[0].object;
        while (clickedGroup.parent && (!clickedGroup.userData || !clickedGroup.userData.farm)) {
          clickedGroup = clickedGroup.parent;
        }
        if (clickedGroup.userData && clickedGroup.userData.farm) {
          this.selectBlock(clickedGroup);
        }
      }
    },
    
    onMouseMove(event) {
      // 节流处理：避免频繁触发导致性能问题
      if (this.mouseMoveTimer) {
        clearTimeout(this.mouseMoveTimer);
      }
      
      this.mouseMoveTimer = setTimeout(() => {
        const container = this.$refs.threeContainer;
        if (!container || !this.camera || !this.raycaster) return;
        
        const rect = container.getBoundingClientRect();
        
        this.mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
        this.mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;
        
        this.raycaster.setFromCamera(this.mouse, this.camera);
        
        // 优化：直接检测农田地块组，使用recursive参数，避免手动遍历
        const intersects = this.raycaster.intersectObjects(this.farmBlocks, true);
        
        // 重置所有地块的高亮
        this.farmBlocks.forEach(group => {
          if (group !== this.selectedBlock) {
            group.traverse(child => {
              if (child.isMesh && child.material) {
                child.material.emissive = new THREE.Color(0x000000);
                child.material.emissiveIntensity = 0;
              }
            });
          }
        });
        
        // 高亮悬停的地块
        if (intersects.length > 0) {
          let hoveredGroup = intersects[0].object;
          while (hoveredGroup.parent && (!hoveredGroup.userData || !hoveredGroup.userData.farm)) {
            hoveredGroup = hoveredGroup.parent;
          }
          
          if (hoveredGroup.userData && hoveredGroup.userData.farm && hoveredGroup !== this.selectedBlock) {
            hoveredGroup.traverse(child => {
              if (child.isMesh && child.material) {
                child.material.emissive = new THREE.Color(0x00AAFF);
                child.material.emissiveIntensity = 0.3;
              }
            });
            container.style.cursor = 'pointer';
          } else {
            container.style.cursor = 'default';
          }
        } else {
          container.style.cursor = 'default';
        }
      }, 50); // 50ms节流，减少性能开销
    },
    
    selectBlock(group) {
      // 重置之前选中的地块
      if (this.selectedBlock) {
        this.selectedBlock.traverse(child => {
          if (child.isMesh && child.material) {
            child.material.emissive = new THREE.Color(0x000000);
            child.material.emissiveIntensity = 0;
          }
        });
      }
      
      // 高亮新选中的地块
      this.selectedBlock = group;
      group.traverse(child => {
        if (child.isMesh && child.material) {
          child.material.emissive = new THREE.Color(0xFFAA00);
          child.material.emissiveIntensity = 0.5;
        }
      });
      
      // 更新选中的农田
      this.selectedFarm = group.userData.farm;
    },
    
    onWindowResize() {
      if (!this.camera || !this.renderer || !this.$refs.threeContainer) return;
      
      const width = this.$refs.threeContainer.clientWidth;
      const height = this.$refs.threeContainer.clientHeight;
      
      this.camera.aspect = width / height;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(width, height);
    },
    
    animate() {
      requestAnimationFrame(this.animate.bind(this));
      
      // 节流控制：只在需要时更新
      if (this.controls) {
        this.controls.update();
      }
      
      // 使用节流渲染：降低渲染频率
      if (!this.lastRenderTime) {
        this.lastRenderTime = performance.now();
      }
      
      const now = performance.now();
      const deltaTime = now - this.lastRenderTime;
      
      // 限制帧率为60fps（约16.67ms一帧）
      if (deltaTime >= 16.67) {
        if (this.renderer && this.scene && this.camera) {
          this.renderer.render(this.scene, this.camera);
        }
        this.lastRenderTime = now;
      }
    }
  }
};
</script>

<style scoped>
.farm-map-3d {
  width: 100%;
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  padding: 20px;
}

.top-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: white;
  padding: 15px 30px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  margin-bottom: 20px;
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

.time, .weather {
  font-size: 14px;
  color: #606266;
  display: flex;
  align-items: center;
  gap: 8px;
}

.weather i {
  font-size: 18px;
  color: #E6A23C;
}

.main-container {
  display: flex;
  gap: 20px;
  height: calc(100vh - 140px);
}

.left-panel {
  width: 320px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
}

.stat-card {
  padding: 15px;
  border-radius: 12px;
  color: white;
  display: flex;
  align-items: center;
  gap: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  cursor: pointer;
  transition: all 0.3s;
}

.stat-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.2);
}

.stat-icon {
  font-size: 32px;
}

.stat-info {
  flex: 1;
}

.stat-label {
  font-size: 12px;
  opacity: 0.9;
  margin-bottom: 5px;
}

.stat-value {
  font-size: 22px;
  font-weight: bold;
}

.stat-value .unit {
  font-size: 12px;
  margin-left: 4px;
  opacity: 0.8;
}

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

.info-row {
  display: flex;
  justify-content: space-between;
  margin-top: 5px;
}

.center-panel {
  flex: 1;
  background: white;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  display: flex;
  flex-direction: column;
}

.map-3d {
  flex: 1;
  width: 100%;
  min-height: 500px;
}

.right-panel {
  width: 380px;
  background: white;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  overflow-y: auto;
}

.farm-detail {
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

.detail-item .label {
  font-size: 12px;
  color: #909399;
  margin-bottom: 5px;
}

.detail-item .value {
  font-size: 15px;
  font-weight: bold;
  color: #303133;
}

.env-section {
  margin-top: 20px;
}

.section-title {
  font-size: 14px;
  font-weight: bold;
  color: #303133;
  margin-bottom: 12px;
  padding-left: 10px;
  border-left: 3px solid #409EFF;
}

.env-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
}

.env-item {
  background: #f5f7fa;
  padding: 10px;
  border-radius: 6px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.env-label {
  font-size: 12px;
  color: #606266;
  flex: 1;
}

.env-value {
  font-size: 14px;
  font-weight: bold;
  color: #303133;
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

.farm-list::-webkit-scrollbar,
.right-panel::-webkit-scrollbar {
  width: 6px;
}

.farm-list::-webkit-scrollbar-thumb,
.right-panel::-webkit-scrollbar-thumb {
  background: #dcdfe6;
  border-radius: 3px;
}

.farm-list::-webkit-scrollbar-track,
.right-panel::-webkit-scrollbar-track {
  background: #f5f7fa;
}

/* 环境健康评分 */
.env-health-score {
  display: flex;
  align-items: center;
  gap: 20px;
  padding: 20px;
  background: linear-gradient(135deg, #f5f7fa 0%, #e8eef5 100%);
  border-radius: 12px;
  margin: 20px 0;
}

.score-circle {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.15);
  animation: pulse 2s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
}

.score-inner {
  width: 85px;
  height: 85px;
  background: white;
  border-radius: 50%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.score-number {
  font-size: 32px;
  font-weight: bold;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.score-label {
  font-size: 11px;
  color: #909399;
  margin-top: 2px;
}

.score-status {
  flex: 1;
}

.status-badge {
  display: inline-block;
  padding: 6px 16px;
  border-radius: 20px;
  font-size: 14px;
  font-weight: bold;
  margin-bottom: 8px;
}

.status-badge.level-excellent {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.status-badge.level-good {
  background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%);
  color: white;
}

.status-badge.level-normal {
  background: linear-gradient(135deg, #fa709a 0%, #fee140 100%);
  color: white;
}

.status-badge.level-poor {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  color: white;
}

.status-desc {
  font-size: 13px;
  color: #606266;
  line-height: 1.6;
}

/* 环境监测标题 */
.section-title {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 15px;
  font-weight: bold;
  color: #303133;
  margin-bottom: 20px;
  padding-left: 10px;
  border-left: 4px solid #409EFF;
}

.update-time {
  font-size: 12px;
  color: #909399;
  font-weight: normal;
}

/* 环形进度条组 */
.env-circles {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  margin-bottom: 25px;
}

.circle-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}

.circle-progress {
  position: relative;
  width: 80px;
  height: 80px;
}

.progress-ring {
  transform: rotate(0deg);
}

.progress-ring-circle {
  transition: stroke-dasharray 0.5s ease;
}

.circle-content {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 2px;
}

.circle-icon {
  font-size: 20px;
}

.circle-value {
  font-size: 12px;
  font-weight: bold;
  color: #303133;
}

.circle-label {
  font-size: 12px;
  color: #606266;
  text-align: center;
}

.circle-status {
  font-size: 11px;
  font-weight: bold;
  padding: 2px 8px;
  border-radius: 10px;
  background: rgba(103, 194, 58, 0.1);
}

/* 智能建议 */
.smart-suggestions {
  background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
  border-radius: 12px;
  padding: 15px;
  margin-top: 20px;
}

.suggestion-title {
  font-size: 14px;
  font-weight: bold;
  color: #303133;
  margin-bottom: 12px;
  display: flex;
  align-items: center;
  gap: 6px;
}

.suggestion-title i {
  color: #409EFF;
  font-size: 16px;
}

.suggestion-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.suggestion-item {
  display: flex;
  gap: 12px;
  padding: 12px;
  background: white;
  border-radius: 8px;
  border-left: 3px solid #409EFF;
  transition: all 0.3s ease;
}

.suggestion-item:hover {
  transform: translateX(5px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.suggestion-item.warning {
  border-left-color: #E6A23C;
}

.suggestion-item.danger {
  border-left-color: #F56C6C;
}

.suggestion-item.success {
  border-left-color: #67C23A;
}

.suggestion-item.info {
  border-left-color: #409EFF;
}

.suggestion-icon {
  font-size: 24px;
  flex-shrink: 0;
}

.suggestion-content {
  flex: 1;
}

.suggestion-text {
  font-size: 13px;
  font-weight: bold;
  color: #303133;
  margin-bottom: 4px;
}

.suggestion-action {
  font-size: 12px;
  color: #606266;
  line-height: 1.5;
}
</style>
