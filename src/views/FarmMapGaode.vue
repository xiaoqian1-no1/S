<template>
  <div class="gaode-map-page">
    <!-- 顶部信息栏 -->
    <div class="top-bar">
      <div class="title">
        <i class="el-icon-map-location"></i>
        农场地理地图
      </div>
      <div class="position-info">
        <i class="el-icon-location-outline"></i>
        <span>{{ currentLocation }}</span>
      </div>
    </div>

    <!-- 主内容区 -->
    <div class="main-container">
      <!-- 左侧：农场列表 -->
      <div class="left-panel">
        <div class="panel-header">
          <h3>农场列表</h3>
          <el-button size="small" type="success" icon="el-icon-refresh" @click="loadFarms">刷新</el-button>
        </div>

        <!-- 搜索框 -->
        <div class="search-box">
          <el-input
            v-model="searchKeyword"
            placeholder="搜索农场名称或作物"
            prefix-icon="el-icon-search"
            size="small"
            clearable
          ></el-input>
        </div>

        <!-- 农场列表（按地址分组） -->
        <div class="farm-list-container">
          <div v-if="filteredFarms.length === 0" class="empty-tip">
            <el-empty description="暂无农场数据" :image-size="80"></el-empty>
          </div>

          <div
            v-for="(farm, index) in filteredFarms"
            :key="farm.id || index"
            class="farm-item"
            :class="{ active: selectedFarm && selectedFarm.id === farm.id }"
            @click="selectFarm(farm)"
          >
            <div class="farm-icon">🌾</div>
            <div class="farm-info">
              <div class="farm-name">{{ farm.farm }}</div>
              <div class="farm-details">
                <span class="detail-item">🌱 {{ farm.crop }}</span>
                <span class="detail-item">📏 {{ farm.area }}亩</span>
              </div>
              <div class="farm-address">📍 {{ farm.address || '暂无地址' }}</div>
            </div>
          </div>
        </div>
      </div>

      <!-- 右侧：地图 -->
      <div class="right-map-panel">
        <!-- 加载提示 -->
        <div v-if="!mapLoaded" class="loading-overlay">
          <el-spinner></el-spinner>
          <p>地图加载中...</p>
        </div>

        <div id="gaode-container" class="map-container"></div>
        
        <!-- 地图控制按钮 -->
        <div class="map-controls">
          <el-button-group>
            <el-tooltip content="放大" placement="left">
              <el-button size="mini" icon="el-icon-zoom-in" @click="zoomIn"></el-button>
            </el-tooltip>
            <el-tooltip content="缩小" placement="left">
              <el-button size="mini" icon="el-icon-zoom-out" @click="zoomOut"></el-button>
            </el-tooltip>
            <el-tooltip content="定位" placement="left">
              <el-button size="mini" icon="el-icon-location" @click="autoLocate"></el-button>
            </el-tooltip>
            <el-tooltip content="重置" placement="left">
              <el-button size="mini" icon="el-icon-refresh" @click="resetView"></el-button>
            </el-tooltip>
          </el-button-group>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'FarmMapGaode',
  data() {
    return {
      map: null,
      amapKey: 'a0dc4534ab26be714e94cef345e480aa', // 使用固定Key，避免依赖后端
      currentLocation: '张家界市',
      searchKeyword: '',
      allFarms: [],
      selectedFarm: null,
      farmMarkers: [],
      mapLoaded: false
    }
  },
  
  computed: {
    // 过滤农场
    filteredFarms() {
      if (!this.searchKeyword) {
        return this.allFarms;
      }
      const keyword = this.searchKeyword.toLowerCase();
      return this.allFarms.filter(farm => {
        const name = (farm.farm || '').toLowerCase();
        const crop = (farm.crop || '').toLowerCase();
        return name.includes(keyword) || crop.includes(keyword);
      });
    }
  },
  
  mounted() {
    this.loadAmapScript();
    this.loadFarms();
  },
  
  beforeDestroy() {
    if (this.map) {
      this.map.destroy();
    }
  },
  
  methods: {
    // 加载高德地图SDK
    loadAmapScript() {
      if (window.AMap) {
        this.createMap();
        return;
      }
      
      const script = document.createElement('script');
      script.src = `https://webapi.amap.com/maps?v=2.0&key=${this.amapKey}&plugin=AMap.Marker,AMap.InfoWindow,AMap.ToolBar`;
      
      script.onload = () => {
        console.log('✅ 高德地图SDK加载成功');
        this.createMap();
      };
      
      script.onerror = () => {
        console.error('❌ 地图SDK加载失败');
        this.$message.error('地图加载失败，请检查网络');
      };
      
      document.head.appendChild(script);
    },
    
    // 创建地图
    createMap() {
      if (!window.AMap) {
        console.error('AMap未加载');
        return;
      }
      
      this.$nextTick(() => {
        const container = document.getElementById('gaode-container');
        if (!container) {
          setTimeout(() => this.createMap(), 100);
          return;
        }
        
        try {
          // 创建3D地图
          this.map = new window.AMap.Map('gaode-container', {
            viewMode: '3D',
            pitch: 60,
            zoom: 13,
            center: [110.479, 29.117], // 张家界市中心
            showBuildingBlock: true,
            resizeEnable: true
          });
          
          // 添加卫星图层
          this.map.addLayer(new window.AMap.TileLayer.Satellite());
          this.map.addLayer(new window.AMap.TileLayer.RoadNet());
          
          // 添加工具栏
          this.map.addControl(new window.AMap.ToolBar({
            locate: true
          }));
          
          this.mapLoaded = true;
          console.log('✅ 地图创建成功');
          this.$message.success('地图加载成功');
          
          // 渲染农场标记
          if (this.allFarms.length > 0) {
            this.renderFarmMarkers();
          }
        } catch (error) {
          console.error('地图创建失败:', error);
          this.$message.error('地图创建失败');
        }
      });
    },
    
    // 加载农场数据
    async loadFarms() {
      try {
        const res = await this.request.get('/statistic/page', {
          params: {
            pageNum: 1,
            pageSize: 1000,
            farm: ''
          }
        });
        
        if (res.code === '200' && res.data && res.data.records) {
          this.allFarms = res.data.records;
          console.log('✅ 加载农场数据成功:', this.allFarms.length);
          
          // 如果地图已加载，渲染标记
          if (this.mapLoaded) {
            this.renderFarmMarkers();
          }
        }
      } catch (error) {
        console.error('加载农场数据失败:', error);
        this.$message.error('加载农场数据失败');
      }
    },
    
    // 渲染农场标记
    renderFarmMarkers() {
      if (!this.map || !window.AMap || this.allFarms.length === 0) return;
      
      // 清除现有标记
      this.farmMarkers.forEach(marker => marker.setMap(null));
      this.farmMarkers = [];
      
      const baseCenter = this.map.getCenter();
      
      this.allFarms.forEach((farm, index) => {
        // 使用黄金角分布算法分布农场标记
        const angle = (index * 137.5) * Math.PI / 180;
        const distance = 0.008 + (index % 8) * 0.004;
        
        const position = [
          baseCenter.lng + Math.cos(angle) * distance,
          baseCenter.lat + Math.sin(angle) * distance
        ];
        
        // 创建标记
        const marker = new window.AMap.Marker({
          position: position,
          title: farm.farm,
          label: {
            content: `<div style="background: #4caf50; color: white; padding: 4px 10px; border-radius: 4px; font-size: 12px; font-weight: 600; box-shadow: 0 2px 8px rgba(0,0,0,0.3); white-space: nowrap;">${farm.farm}</div>`,
            direction: 'top',
            offset: new window.AMap.Pixel(0, -10)
          }
        });
        
        marker.setMap(this.map);
        
        // 创建信息窗口
        const infoWindow = new window.AMap.InfoWindow({
          content: this.createInfoWindowContent(farm),
          offset: new window.AMap.Pixel(0, -30)
        });
        
        marker.on('click', () => {
          infoWindow.open(this.map, marker.getPosition());
          this.selectedFarm = farm;
        });
        
        this.farmMarkers.push(marker);
      });
      
      console.log('✅ 渲染了', this.farmMarkers.length, '个农场标记');
    },
    
    // 创建信息窗口内容
    createInfoWindowContent(farm) {
      return `
        <div style="padding: 15px; min-width: 250px; font-family: Arial, sans-serif;">
          <h3 style="margin: 0 0 12px 0; color: #2e7d32; font-size: 16px; border-bottom: 2px solid #4caf50; padding-bottom: 8px;">
            ${farm.farm}
          </h3>
          <p style="margin: 6px 0; color: #666; font-size: 14px;">
            <strong>🌾 作物：</strong>${farm.crop || '未知'}
          </p>
          <p style="margin: 6px 0; color: #666; font-size: 14px;">
            <strong>📏 面积：</strong>${farm.area || '未知'}亩
          </p>
          <p style="margin: 6px 0; color: #666; font-size: 14px;">
            <strong>🌱 数量：</strong>${farm.number || 0}株
          </p>
          <p style="margin: 6px 0; color: #666; font-size: 14px;">
            <strong>📊 状态：</strong><span style="color: ${farm.state === '正常' ? '#67c23a' : '#e6a23c'}">${farm.state || '未知'}</span>
          </p>
          <p style="margin: 6px 0; color: #666; font-size: 14px;">
            <strong>👤 负责人：</strong>${farm.keeper || '未知'}
          </p>
          <p style="margin: 6px 0; color: #999; font-size: 12px;">
            📍 ${farm.address || '暂无地址'}
          </p>
        </div>
      `;
    },
    
    // 选择农场
    selectFarm(farm) {
      if (!this.map || !this.mapLoaded) {
        this.$message.warning('地图未加载完成');
        return;
      }
      
      this.selectedFarm = farm;
      
      // 找到对应的标记
      const marker = this.farmMarkers.find(m => m.getTitle() === farm.farm);
      
      if (marker) {
        const position = marker.getPosition();
        
        // 定位到该农场
        this.map.setCenter(position);
        this.map.setZoom(16);
        
        // 创建并显示信息窗口
        const infoWindow = new window.AMap.InfoWindow({
          content: this.createInfoWindowContent(farm),
          offset: new window.AMap.Pixel(0, -30)
        });
        
        infoWindow.open(this.map, position);
        
        // 添加跳动动画标记
        const highlightMarker = new window.AMap.Marker({
          position: position,
          icon: new window.AMap.Icon({
            size: new window.AMap.Size(40, 40),
            image: 'data:image/svg+xml;base64,' + btoa(`
              <svg xmlns="http://www.w3.org/2000/svg" width="40" height="40">
                <circle cx="20" cy="20" r="18" fill="#ff4444" stroke="#fff" stroke-width="3"/>
                <circle cx="20" cy="20" r="8" fill="#fff"/>
              </svg>
            `),
            imageSize: new window.AMap.Size(40, 40)
          }),
          zIndex: 200,
          animation: 'AMAP_ANIMATION_BOUNCE'
        });
        
        highlightMarker.setMap(this.map);
        
        // 5秒后移除高亮
        setTimeout(() => {
          highlightMarker.setMap(null);
        }, 5000);
        
        this.$message.success(`已定位到：${farm.farm}`);
      }
    },
    
    // 自动定位
    autoLocate() {
      if (!this.map) return;
      
      if (navigator.geolocation) {
        const loading = this.$loading({ text: '定位中...' });
        
        navigator.geolocation.getCurrentPosition(
          (position) => {
            const { longitude, latitude } = position.coords;
            this.map.setCenter([longitude, latitude]);
            this.map.setZoom(16);
            this.currentLocation = `${latitude.toFixed(6)}, ${longitude.toFixed(6)}`;
            
            loading.close();
            this.$message.success('定位成功');
          },
          (error) => {
            loading.close();
            console.error('定位失败:', error);
            this.$message.warning('定位失败，使用默认位置');
          }
        );
      } else {
        this.$message.error('浏览器不支持定位功能');
      }
    },
    
    // 地图控制
    zoomIn() {
      if (this.map) this.map.zoomIn();
    },
    
    zoomOut() {
      if (this.map) this.map.zoomOut();
    },
    
    resetView() {
      if (this.map) {
        this.map.setZoomAndCenter(13, [110.479, 29.117]);
        this.map.setPitch(60);
        this.$message.success('视图已重置');
      }
    }
  }
}
</script>

<style scoped>
.gaode-map-page {
  width: 100%;
  height: 100vh;
  background: #f5f7fa;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* 顶部信息栏 */
.top-bar {
  height: 60px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 30px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  z-index: 100;
  flex-shrink: 0;
}

.title {
  font-size: 24px;
  font-weight: bold;
  color: white;
  display: flex;
  align-items: center;
  gap: 10px;
}

.position-info {
  display: flex;
  align-items: center;
  gap: 8px;
  background: rgba(255, 255, 255, 0.2);
  padding: 8px 16px;
  border-radius: 20px;
  color: white;
  font-size: 14px;
}

/* 主内容区 */
.main-container {
  flex: 1;
  display: flex;
  gap: 20px;
  padding: 20px;
  overflow: hidden;
  min-height: 0;
}

/* 左侧面板 */
.left-panel {
  width: 350px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  flex-shrink: 0;
}

.panel-header {
  padding: 16px;
  border-bottom: 1px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.panel-header h3 {
  margin: 0;
  font-size: 16px;
  color: #333;
}

.search-box {
  padding: 12px 16px;
  border-bottom: 1px solid #eee;
}

.farm-list-container {
  flex: 1;
  overflow-y: auto;
  padding: 12px;
}

.empty-tip {
  padding: 40px 20px;
  text-align: center;
}

/* 农场列表项 */
.farm-item {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  padding: 14px;
  margin-bottom: 10px;
  background: #fafafa;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
  border: 2px solid transparent;
}

.farm-item:hover {
  background: #e8f5e9;
  border-color: #4caf50;
  transform: translateX(4px);
}

.farm-item.active {
  background: #e8f5e9;
  border-color: #4caf50;
}

.farm-icon {
  font-size: 28px;
  flex-shrink: 0;
}

.farm-info {
  flex: 1;
  min-width: 0;
}

.farm-name {
  font-size: 15px;
  font-weight: 600;
  color: #333;
  margin-bottom: 6px;
}

.farm-details {
  display: flex;
  gap: 12px;
  margin-bottom: 4px;
}

.detail-item {
  font-size: 12px;
  color: #666;
}

.farm-address {
  font-size: 12px;
  color: #999;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* 右侧地图面板 */
.right-map-panel {
  flex: 1;
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  position: relative;
  min-height: 600px;
}

.loading-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.95);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  gap: 15px;
}

.loading-overlay p {
  font-size: 14px;
  color: #666;
}

.map-container {
  width: 100%;
  height: 100%;
  min-height: 600px;
}

/* 地图控制按钮 */
.map-controls {
  position: absolute;
  top: 20px;
  right: 20px;
  z-index: 10;
}

/* 滚动条美化 */
.farm-list-container::-webkit-scrollbar {
  width: 6px;
}

.farm-list-container::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 3px;
}

.farm-list-container::-webkit-scrollbar-thumb {
  background: #4caf50;
  border-radius: 3px;
}

.farm-list-container::-webkit-scrollbar-thumb:hover {
  background: #45a049;
}

/* 隐藏高德地图logo */
:deep(.amap-logo) {
  display: none !important;
}

:deep(.amap-copyright) {
  opacity: 0.3;
}
</style>

