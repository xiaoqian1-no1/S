<template>
  <div class="big-screen">
    <!-- 顶部条 -->
    <header class="top-bar">
      <div class="left-title">
        <i class="el-icon-s-data"></i>
        智慧农场一体化监控平台
      </div>
      <div class="center-time">
        {{ currentTime }}
      </div>
      <div class="right-weather">
        <i class="el-icon-sunny"></i>
        {{ weather.temp }}°C {{ weather.status }}
      </div>
    </header>

    <!-- 左侧菜单 -->
    <aside class="left-sidebar">
      <div 
        v-for="(menu, index) in menus" 
        :key="index"
        class="menu-item"
        :class="{ active: activeMenu === index }"
        @click="activeMenu = index"
      >
        <i :class="menu.icon"></i>
        <span>{{ menu.name }}</span>
      </div>
    </aside>

    <!-- 中心主视图区 -->
    <main class="center-map">
      <div class="map-container">
        <!-- Three.js 3D 场景容器 -->
        <div ref="threeContainer" class="three-container"></div>
      </div>
    </main>

    <!-- 右侧信息面板 -->
    <aside class="right-panel">
      <!-- 监控预览 -->
      <div class="monitor-preview">
        <div class="panel-title">监控预览</div>
        <div class="preview-image"></div>
        <div class="preview-info">
          <h4>{{ selectedArea.name || '请选择区域' }}</h4>
          <p>{{ selectedArea.description || '点击地图上的区域查看详情' }}</p>
        </div>
      </div>

      <!-- 统计信息 -->
      <div class="stats-panel">
        <div class="panel-title">实时监控</div>
        
        <!-- 环形仪表盘 -->
        <div class="gauge-chart" ref="gaugeChart"></div>
        
        <!-- 统计卡片 -->
        <div class="stat-cards">
          <div class="stat-card">
            <div class="stat-label">今日告警</div>
            <div class="stat-value warning">{{ summary.alerts }}</div>
          </div>
          <div class="stat-card">
            <div class="stat-label">设备在线</div>
            <div class="stat-value success">{{ summary.onlineDevices }}/{{ summary.totalDevices }}</div>
          </div>
        </div>
      </div>
    </aside>

    <!-- 底部数据与图表区域 -->
    <footer class="bottom-stats">
      <!-- 左下：生产指标 -->
      <div class="production-stats">
        <div class="panel-title">生产指标概览</div>
        <div class="stat-items">
          <div class="stat-item">
            <div class="stat-icon" style="background: #409EFF;">
              <i class="el-icon-s-grid"></i>
            </div>
            <div class="stat-content">
              <div class="stat-name">养殖总面积</div>
              <div class="stat-num">{{ summary.totalArea }}<span class="unit">亩</span></div>
            </div>
          </div>
          <div class="stat-item">
            <div class="stat-icon" style="background: #67C23A;">
              <i class="el-icon-s-goods"></i>
            </div>
            <div class="stat-content">
              <div class="stat-name">预计产量</div>
              <div class="stat-num">{{ summary.expectedOutput }}<span class="unit">吨</span></div>
            </div>
          </div>
          <div class="stat-item">
            <div class="stat-icon" style="background: #E6A23C;">
              <i class="el-icon-s-data"></i>
            </div>
            <div class="stat-content">
              <div class="stat-name">当前存栏</div>
              <div class="stat-num">{{ summary.currentStock }}<span class="unit">万尾</span></div>
            </div>
          </div>
        </div>
      </div>

      <!-- 中间：柱状图 -->
      <div class="chart-panel">
        <div class="panel-title">月度产量统计</div>
        <div class="bar-chart" ref="barChart"></div>
      </div>

      <!-- 右下：告警列表 -->
      <div class="alert-list">
        <div class="panel-title">近期告警</div>
        <div class="alert-items">
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
    </footer>
  </div>
</template>

<script>
import * as echarts from 'echarts';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

export default {
  name: 'FarmBigScreen',
  data() {
    return {
      currentTime: '',
      activeMenu: 0,
      selectedArea: {},
      
      // Three.js 相关
      scene: null,
      camera: null,
      renderer: null,
      controls: null,
      farmBlocks: [], // 存储农田方块对象
      raycaster: null,
      mouse: null,
      selectedBlock: null,
      
      // 天气信息
      weather: {
        temp: 28,
        status: '晴'
      },
      
      // 左侧菜单
      menus: [
        { name: '自由巡视', icon: 'el-icon-view' },
        { name: '基础信息', icon: 'el-icon-document' },
        { name: '监控管理', icon: 'el-icon-video-camera' },
        { name: '分区总览', icon: 'el-icon-s-grid' }
      ],
      
      // 总体统计数据
      summary: {
        totalArea: 1250,
        expectedOutput: 156,
        currentStock: 8.5,
        alerts: 3,
        onlineDevices: 24,
        totalDevices: 28,
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
          left: '20%',
          description: '主要养殖南美白对虾，采用清水养殖模式，配备自动投喂和水质监测系统。'
        },
        {
          name: '清水虾区B',
          size: 280,
          stock: 11200,
          status: '正常',
          top: '15%',
          left: '50%',
          description: '南美白对虾养殖区，水质优良，溶氧充足。'
        },
        {
          name: '笼养区',
          size: 180,
          stock: 8600,
          status: '告警',
          top: '45%',
          left: '25%',
          description: '采用网箱笼养模式，适合高密度养殖，需注意水质监控。'
        },
        {
          name: '混养区',
          size: 220,
          stock: 9800,
          status: '正常',
          top: '45%',
          left: '55%',
          description: '虾蟹混养区域，生态平衡良好，产量稳定。'
        },
        {
          name: '育苗区',
          size: 150,
          stock: 15000,
          status: '正常',
          top: '70%',
          left: '35%',
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
    
    // 初始化图表和3D场景
    this.$nextTick(() => {
      this.initThreeScene();
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
    
    // 初始化Three.js场景
    initThreeScene() {
      const container = this.$refs.threeContainer;
      if (!container) return;
      
      const width = container.clientWidth;
      const height = container.clientHeight;
      
      // 创建场景
      this.scene = new THREE.Scene();
      this.scene.background = new THREE.Color(0x0a1929);
      this.scene.fog = new THREE.Fog(0x0a1929, 50, 200);
      
      // 创建相机
      this.camera = new THREE.PerspectiveCamera(60, width / height, 0.1, 1000);
      this.camera.position.set(30, 40, 30);
      this.camera.lookAt(0, 0, 0);
      
      // 创建渲染器
      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setSize(width, height);
      this.renderer.setPixelRatio(window.devicePixelRatio);
      this.renderer.shadowMap.enabled = true;
      this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;
      container.appendChild(this.renderer.domElement);
      
      // 添加轨道控制器
      this.controls = new OrbitControls(this.camera, this.renderer.domElement);
      this.controls.enableDamping = true;
      this.controls.dampingFactor = 0.05;
      this.controls.minDistance = 20;
      this.controls.maxDistance = 100;
      this.controls.maxPolarAngle = Math.PI / 2.2;
      
      // 添加光源
      const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
      this.scene.add(ambientLight);
      
      const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
      directionalLight.position.set(50, 50, 50);
      directionalLight.castShadow = true;
      directionalLight.shadow.mapSize.width = 2048;
      directionalLight.shadow.mapSize.height = 2048;
      directionalLight.shadow.camera.near = 0.5;
      directionalLight.shadow.camera.far = 200;
      directionalLight.shadow.camera.left = -50;
      directionalLight.shadow.camera.right = 50;
      directionalLight.shadow.camera.top = 50;
      directionalLight.shadow.camera.bottom = -50;
      this.scene.add(directionalLight);
      
      // 创建地面
      const groundGeometry = new THREE.PlaneGeometry(100, 100);
      const groundMaterial = new THREE.MeshStandardMaterial({ 
        color: 0x1a3a52,
        roughness: 0.8,
        metalness: 0.2
      });
      const ground = new THREE.Mesh(groundGeometry, groundMaterial);
      ground.rotation.x = -Math.PI / 2;
      ground.receiveShadow = true;
      this.scene.add(ground);
      
      // 添加网格辅助线
      const gridHelper = new THREE.GridHelper(100, 20, 0x4facfe, 0x1e4d7b);
      gridHelper.position.y = 0.01;
      this.scene.add(gridHelper);
      
      // 创建农田方块
      this.createFarmBlocks();
      
      // 初始化射线检测
      this.raycaster = new THREE.Raycaster();
      this.mouse = new THREE.Vector2();
      
      // 添加鼠标事件
      container.addEventListener('click', this.onMouseClick.bind(this));
      container.addEventListener('mousemove', this.onMouseMove.bind(this));
      
      // 窗口大小调整
      window.addEventListener('resize', this.onWindowResize.bind(this));
      
      // 开始动画循环
      this.animate();
    },
    
    // 创建农田方块
    createFarmBlocks() {
      // 根据areas数据创建3D方块
      const positions = [
        { x: -15, z: -15 },
        { x: 5, z: -15 },
        { x: -10, z: 5 },
        { x: 10, z: 5 },
        { x: 0, z: 20 }
      ];
      
      this.areas.forEach((area, index) => {
        const pos = positions[index] || { x: 0, z: 0 };
        
        // 根据面积计算方块大小
        const size = Math.sqrt(area.size) / 3;
        const height = area.stock / 2000; // 根据存栏数量设置高度
        
        const geometry = new THREE.BoxGeometry(size, height, size);
        const material = new THREE.MeshStandardMaterial({
          color: area.status === '正常' ? 0x4facfe : 0xe6a23c,
          roughness: 0.5,
          metalness: 0.3,
          emissive: area.status === '正常' ? 0x1e4d7b : 0x8b5a00,
          emissiveIntensity: 0.2
        });
        
        const block = new THREE.Mesh(geometry, material);
        block.position.set(pos.x, height / 2, pos.z);
        block.castShadow = true;
        block.receiveShadow = true;
        
        // 存储农田数据
        block.userData = {
          area: area,
          originalColor: material.color.getHex(),
          originalEmissive: material.emissive.getHex()
        };
        
        this.scene.add(block);
        this.farmBlocks.push(block);
        
        // 添加文字标签（使用Sprite）
        this.addTextLabel(area.name, pos.x, height + 2, pos.z);
      });
    },
    
    // 添加文字标签
    addTextLabel(text, x, y, z) {
      const canvas = document.createElement('canvas');
      const context = canvas.getContext('2d');
      canvas.width = 256;
      canvas.height = 64;
      
      context.fillStyle = 'rgba(15, 32, 56, 0.8)';
      context.fillRect(0, 0, canvas.width, canvas.height);
      
      context.font = 'Bold 24px Arial';
      context.fillStyle = '#4facfe';
      context.textAlign = 'center';
      context.textBaseline = 'middle';
      context.fillText(text, canvas.width / 2, canvas.height / 2);
      
      const texture = new THREE.CanvasTexture(canvas);
      const spriteMaterial = new THREE.SpriteMaterial({ map: texture });
      const sprite = new THREE.Sprite(spriteMaterial);
      sprite.position.set(x, y, z);
      sprite.scale.set(8, 2, 1);
      
      this.scene.add(sprite);
    },
    
    // 鼠标点击事件
    onMouseClick(event) {
      const container = this.$refs.threeContainer;
      const rect = container.getBoundingClientRect();
      
      this.mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
      this.mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;
      
      this.raycaster.setFromCamera(this.mouse, this.camera);
      const intersects = this.raycaster.intersectObjects(this.farmBlocks);
      
      if (intersects.length > 0) {
        const clickedBlock = intersects[0].object;
        this.selectBlock(clickedBlock);
      }
    },
    
    // 鼠标移动事件（高亮效果）
    onMouseMove(event) {
      const container = this.$refs.threeContainer;
      const rect = container.getBoundingClientRect();
      
      this.mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
      this.mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;
      
      this.raycaster.setFromCamera(this.mouse, this.camera);
      const intersects = this.raycaster.intersectObjects(this.farmBlocks);
      
      // 重置所有方块
      this.farmBlocks.forEach(block => {
        if (block !== this.selectedBlock) {
          block.material.emissiveIntensity = 0.2;
        }
      });
      
      // 高亮悬停的方块
      if (intersects.length > 0) {
        const hoveredBlock = intersects[0].object;
        if (hoveredBlock !== this.selectedBlock) {
          hoveredBlock.material.emissiveIntensity = 0.5;
        }
        container.style.cursor = 'pointer';
      } else {
        container.style.cursor = 'default';
      }
    },
    
    // 选中方块
    selectBlock(block) {
      // 重置之前选中的方块
      if (this.selectedBlock) {
        this.selectedBlock.material.emissiveIntensity = 0.2;
      }
      
      // 高亮新选中的方块
      this.selectedBlock = block;
      block.material.emissiveIntensity = 0.8;
      
      // 更新右侧面板信息
      this.selectedArea = block.userData.area;
    },
    
    // 窗口大小调整
    onWindowResize() {
      const container = this.$refs.threeContainer;
      if (!container) return;
      
      const width = container.clientWidth;
      const height = container.clientHeight;
      
      this.camera.aspect = width / height;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(width, height);
    },
    
    // 动画循环
    animate() {
      requestAnimationFrame(this.animate.bind(this));
      
      if (this.controls) {
        this.controls.update();
      }
      
      if (this.renderer && this.scene && this.camera) {
        this.renderer.render(this.scene, this.camera);
      }
    },
    
    // 初始化柱状图
    initBarChart() {
      if (!this.$refs.barChart) return;
      
      this.barChartInstance = echarts.init(this.$refs.barChart);
      
      const option = {
        grid: {
          top: '15%',
          left: '10%',
          right: '10%',
          bottom: '15%'
        },
        xAxis: {
          type: 'category',
          data: this.chartData.months,
          axisLine: {
            lineStyle: { color: '#4a5f7a' }
          },
          axisLabel: {
            color: '#8b9bb3'
          }
        },
        yAxis: {
          type: 'value',
          name: '产量(吨)',
          nameTextStyle: {
            color: '#8b9bb3'
          },
          axisLine: {
            lineStyle: { color: '#4a5f7a' }
          },
          axisLabel: {
            color: '#8b9bb3'
          },
          splitLine: {
            lineStyle: {
              color: '#2a3f5f',
              type: 'dashed'
            }
          }
        },
        series: [{
          data: this.chartData.output,
          type: 'bar',
          barWidth: '40%',
          itemStyle: {
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
              { offset: 0, color: '#4facfe' },
              { offset: 1, color: '#00f2fe' }
            ]),
            borderRadius: [4, 4, 0, 0]
          },
          label: {
            show: true,
            position: 'top',
            color: '#8b9bb3',
            fontSize: 10
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
              color: '#4facfe'
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
            color: '#8b9bb3',
            fontSize: 10,
            distance: -40
          },
          title: {
            offsetCenter: [0, '-20%'],
            fontSize: 14,
            color: '#8b9bb3'
          },
          detail: {
            fontSize: 24,
            offsetCenter: [0, '0%'],
            valueAnimation: true,
            formatter: '{value}%',
            color: '#4facfe'
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
.big-screen {
  width: 100vw;
  height: 100vh;
  background: linear-gradient(to bottom, #0a1929, #1a2332);
  color: #e0e6ed;
  overflow: hidden;
  position: relative;
  font-family: 'Microsoft YaHei', Arial, sans-serif;
}

/* ========== 顶部条 ========== */
.top-bar {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 60px;
  background: linear-gradient(to right, rgba(15, 32, 56, 0.9), rgba(26, 35, 50, 0.9));
  border-bottom: 2px solid #1e4d7b;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 30px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  z-index: 100;
}

.left-title {
  font-size: 24px;
  font-weight: bold;
  color: #4facfe;
  letter-spacing: 2px;
}

.left-title i {
  margin-right: 10px;
  font-size: 28px;
}

.center-time {
  font-size: 18px;
  color: #8b9bb3;
  letter-spacing: 1px;
}

.right-weather {
  font-size: 16px;
  color: #8b9bb3;
}

.right-weather i {
  color: #f39c12;
  margin-right: 8px;
  font-size: 20px;
}

/* ========== 左侧菜单 ========== */
.left-sidebar {
  position: absolute;
  left: 0;
  top: 60px;
  bottom: 0;
  width: 100px;
  background: rgba(15, 32, 56, 0.8);
  border-right: 1px solid #1e4d7b;
  padding-top: 20px;
  z-index: 90;
}

.menu-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px 10px;
  cursor: pointer;
  transition: all 0.3s;
  color: #8b9bb3;
}

.menu-item i {
  font-size: 28px;
  margin-bottom: 8px;
}

.menu-item span {
  font-size: 12px;
  text-align: center;
}

.menu-item:hover,
.menu-item.active {
  background: rgba(79, 172, 254, 0.2);
  color: #4facfe;
}

/* ========== 中心主视图区 ========== */
.center-map {
  position: absolute;
  left: 100px;
  top: 60px;
  right: 380px;
  bottom: 280px;
  padding: 20px;
}

.map-container {
  width: 100%;
  height: 100%;
  position: relative;
  border: 2px solid #1e4d7b;
  border-radius: 8px;
  overflow: hidden;
  background: #0f2038;
}

/* Three.js 容器 */
.three-container {
  width: 100%;
  height: 100%;
  position: relative;
}

.three-container canvas {
  display: block;
  border-radius: 6px;
}

/* ========== 右侧信息面板 ========== */
.right-panel {
  position: absolute;
  right: 0;
  top: 60px;
  bottom: 280px;
  width: 380px;
  padding: 20px;
  background: rgba(15, 32, 56, 0.5);
  border-left: 1px solid #1e4d7b;
  overflow-y: auto;
}

.monitor-preview {
  background: rgba(26, 35, 50, 0.8);
  border: 1px solid #1e4d7b;
  border-radius: 8px;
  padding: 15px;
  margin-bottom: 20px;
}

.preview-image {
  width: 100%;
  height: 180px;
  background: linear-gradient(135deg, #1e4d7b 0%, #2a5f8f 100%);
  border-radius: 6px;
  margin: 10px 0;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
}

.preview-image::before {
  content: '📹';
  font-size: 48px;
  opacity: 0.3;
}

.preview-info h4 {
  color: #4facfe;
  margin: 10px 0 5px;
  font-size: 16px;
}

.preview-info p {
  color: #8b9bb3;
  font-size: 13px;
  line-height: 1.6;
}

.stats-panel {
  background: rgba(26, 35, 50, 0.8);
  border: 1px solid #1e4d7b;
  border-radius: 8px;
  padding: 15px;
}

.panel-title {
  font-size: 16px;
  font-weight: bold;
  color: #4facfe;
  margin-bottom: 15px;
  padding-bottom: 10px;
  border-bottom: 2px solid #1e4d7b;
}

.gauge-chart {
  width: 100%;
  height: 200px;
}

.stat-cards {
  display: flex;
  gap: 15px;
  margin-top: 15px;
}

.stat-card {
  flex: 1;
  background: rgba(15, 32, 56, 0.8);
  border: 1px solid #1e4d7b;
  border-radius: 6px;
  padding: 15px;
  text-align: center;
}

.stat-label {
  font-size: 12px;
  color: #8b9bb3;
  margin-bottom: 8px;
}

.stat-value {
  font-size: 24px;
  font-weight: bold;
}

.stat-value.warning {
  color: #E6A23C;
}

.stat-value.success {
  color: #67C23A;
}

/* ========== 底部数据与图表区域 ========== */
.bottom-stats {
  position: absolute;
  left: 100px;
  right: 0;
  bottom: 0;
  height: 280px;
  display: flex;
  gap: 20px;
  padding: 20px;
  background: rgba(15, 32, 56, 0.5);
  border-top: 1px solid #1e4d7b;
}

.production-stats,
.chart-panel,
.alert-list {
  flex: 1;
  background: rgba(26, 35, 50, 0.8);
  border: 1px solid #1e4d7b;
  border-radius: 8px;
  padding: 15px;
  overflow: hidden;
}

/* 生产指标 */
.stat-items {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin-top: 10px;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 15px;
  background: rgba(15, 32, 56, 0.6);
  padding: 12px;
  border-radius: 6px;
  border: 1px solid #1e4d7b;
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
}

.stat-content {
  flex: 1;
}

.stat-name {
  font-size: 13px;
  color: #8b9bb3;
  margin-bottom: 5px;
}

.stat-num {
  font-size: 22px;
  font-weight: bold;
  color: #4facfe;
}

.stat-num .unit {
  font-size: 14px;
  color: #8b9bb3;
  margin-left: 4px;
}

/* 柱状图 */
.bar-chart {
  width: 100%;
  height: calc(100% - 50px);
}

/* 告警列表 */
.alert-items {
  margin-top: 10px;
  max-height: calc(100% - 50px);
  overflow-y: auto;
}

.alert-item {
  padding: 10px;
  margin-bottom: 8px;
  background: rgba(15, 32, 56, 0.6);
  border-left: 3px solid #4facfe;
  border-radius: 4px;
  font-size: 12px;
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
  color: #8b9bb3;
  margin-bottom: 5px;
  font-size: 11px;
}

.alert-content {
  color: #e0e6ed;
  line-height: 1.5;
}

/* 滚动条样式 */
.right-panel::-webkit-scrollbar,
.alert-items::-webkit-scrollbar {
  width: 6px;
}

.right-panel::-webkit-scrollbar-thumb,
.alert-items::-webkit-scrollbar-thumb {
  background: #1e4d7b;
  border-radius: 3px;
}

.right-panel::-webkit-scrollbar-track,
.alert-items::-webkit-scrollbar-track {
  background: rgba(15, 32, 56, 0.5);
}
</style>
