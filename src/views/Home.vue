<template>
  <div class="green-home-page">
    <!-- 欢迎横幅 -->
    <div class="welcome-banner">
      <div class="banner-content">
        <div class="welcome-text">
          <h1>🌿 欢迎使用智慧农田管理系统</h1>
          <p>现代化农业管理，让科技助力农田发展</p>
        </div>
        <div class="banner-stats">
          <div class="stat-item">
            <i class="el-icon-s-data"></i>
            <span>数据驱动</span>
          </div>
          <div class="stat-item">
            <i class="el-icon-view"></i>
            <span>3D可视化</span>
          </div>
          <div class="stat-item">
            <i class="el-icon-connection"></i>
            <span>硬件集成</span>
          </div>
        </div>
      </div>
    </div>

    <!-- 功能卡片网格 -->
    <div class="feature-grid">
      <el-row :gutter="20">
        <el-col :span="8">
          <el-card class="feature-card" @click.native="$router.push('/farmmap3d')">
            <div class="feature-icon">🗺️</div>
            <h3>农场3D地图</h3>
            <p>Three.js 3D可视化农场布局，健康指数实时展示</p>
            <div class="feature-tag">
              <el-tag type="success" size="small">核心功能</el-tag>
            </div>
          </el-card>
        </el-col>
        
        <el-col :span="8">
          <el-card class="feature-card" @click.native="$router.push('/farm-map-gaode')">
            <div class="feature-icon">🌍</div>
            <h3>农场地理地图</h3>
            <p>基于高德地图的真实地理位置展示</p>
            <div class="feature-tag">
              <el-tag type="primary" size="small">新功能</el-tag>
            </div>
          </el-card>
        </el-col>
        
        <el-col :span="8">
          <el-card class="feature-card" @click.native="$router.push('/aether-monitor')">
            <div class="feature-icon">📊</div>
            <h3>环境数据监控</h3>
            <p>STM32实时数据监控，温湿度智能控制</p>
            <div class="feature-tag">
              <el-tag type="warning" size="small">实时数据</el-tag>
            </div>
          </el-card>
        </el-col>
      </el-row>
      
      <el-row :gutter="20" style="margin-top: 20px;">
        <el-col :span="8">
          <el-card class="feature-card" @click.native="$router.push('/business-analysis')">
            <div class="feature-icon">💰</div>
            <h3>经营分析</h3>
            <p>产量预测、利润分析、经营决策支持</p>
            <div class="feature-tag">
              <el-tag type="success" size="small">智能分析</el-tag>
            </div>
          </el-card>
        </el-col>
        
        <el-col :span="8">
          <el-card class="feature-card" @click.native="$router.push('/farmland')">
            <div class="feature-icon">🌾</div>
            <h3>农田管理</h3>
            <p>农田信息管理、作物管理、负责人管理</p>
            <div class="feature-tag">
              <el-tag type="info" size="small">基础管理</el-tag>
            </div>
          </el-card>
        </el-col>
        
        <el-col :span="8">
          <el-card class="feature-card" @click.native="$router.push('/inventory')">
            <div class="feature-icon">📦</div>
            <h3>物资管理</h3>
            <p>库存管理、采购销售、智能补货提醒</p>
            <div class="feature-tag">
              <el-tag type="warning" size="small">库存管理</el-tag>
            </div>
          </el-card>
        </el-col>
      </el-row>
    </div>

    <!-- 系统公告 -->
    <el-card class="notice-card">
      <div slot="header" class="notice-header">
        <i class="el-icon-bell"></i>
        <span>系统公告</span>
      </div>
      <div class="notice-content">
        <div class="notice-item">
          <div class="notice-date">2025-11-18</div>
          <div class="notice-text">🎉 系统升级完成！新增农场地理地图、经营分析等功能</div>
        </div>
        <div class="notice-item">
          <div class="notice-date">2025-11-18</div>
          <div class="notice-text">🔌 STM32硬件集成成功，支持实时环境数据监控</div>
        </div>
        <div class="notice-item">
          <div class="notice-date">2025-11-18</div>
          <div class="notice-text">📊 健康指数模型上线，智能评估农田环境状况</div>
        </div>
      </div>
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'Home',
  data() {
    return {
      
    }
  },
  mounted() {
    
  },
  methods: {
    
  }
}
</script>

<style scoped>
.green-home-page {
  padding: 0;
  min-height: calc(100vh - 60px);
}

/* 欢迎横幅 */
.welcome-banner {
  background: linear-gradient(135deg, rgba(76, 175, 80, 0.1) 0%, rgba(129, 199, 132, 0.1) 100%);
  border: 2px solid rgba(76, 175, 80, 0.2);
  border-radius: 16px;
  padding: 30px;
  margin-bottom: 30px;
  backdrop-filter: blur(10px);
}

.banner-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.welcome-text h1 {
  font-size: 28px;
  color: #2c5530;
  margin-bottom: 8px;
  font-weight: 600;
}

.welcome-text p {
  font-size: 16px;
  color: #4caf50;
  margin: 0;
}

.banner-stats {
  display: flex;
  gap: 30px;
}

.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  color: #4caf50;
  font-weight: 500;
}

.stat-item i {
  font-size: 32px;
}

/* 功能卡片网格 */
.feature-grid {
  margin-bottom: 30px;
}

.feature-card {
  text-align: center;
  cursor: pointer;
  transition: all 0.3s ease;
  height: 200px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  position: relative;
  overflow: hidden;
}

.feature-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(76, 175, 80, 0.1), transparent);
  transition: all 0.6s;
}

.feature-card:hover::before {
  left: 100%;
}

.feature-card:hover {
  transform: translateY(-8px) scale(1.02);
  box-shadow: 0 12px 40px rgba(76, 175, 80, 0.3) !important;
  border-color: rgba(76, 175, 80, 0.6) !important;
}

.feature-icon {
  font-size: 48px;
  margin-bottom: 15px;
  filter: drop-shadow(0 2px 4px rgba(76, 175, 80, 0.3));
}

.feature-card h3 {
  font-size: 18px;
  color: #2c5530;
  margin-bottom: 10px;
  font-weight: 600;
}

.feature-card p {
  font-size: 14px;
  color: #666;
  line-height: 1.5;
  margin-bottom: 15px;
}

.feature-tag {
  position: absolute;
  top: 15px;
  right: 15px;
}

/* 公告卡片 */
.notice-card {
  margin-top: 20px;
}

.notice-header {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 16px;
  color: #2c5530;
}

.notice-header i {
  font-size: 20px;
  color: #4caf50;
}

.notice-content {
  max-height: 200px;
  overflow-y: auto;
}

.notice-item {
  display: flex;
  align-items: flex-start;
  gap: 15px;
  padding: 12px 0;
  border-bottom: 1px solid rgba(76, 175, 80, 0.1);
}

.notice-item:last-child {
  border-bottom: none;
}

.notice-date {
  font-size: 12px;
  color: #4caf50;
  font-weight: 600;
  white-space: nowrap;
  background: rgba(76, 175, 80, 0.1);
  padding: 4px 8px;
  border-radius: 4px;
}

.notice-text {
  font-size: 14px;
  color: #333;
  line-height: 1.5;
  flex: 1;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .banner-content {
    flex-direction: column;
    gap: 20px;
  }
  
  .banner-stats {
    gap: 20px;
  }
  
  .feature-card {
    height: 160px;
  }
  
  .feature-icon {
    font-size: 36px;
  }
}
</style>