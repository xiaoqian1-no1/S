<template>
  <el-dialog 
    title="选择农场位置" 
    :visible.sync="visible" 
    width="90%"
    top="5vh"
    :close-on-click-modal="false"
    @close="handleClose"
  >
    <div class="location-selector">
      <!-- 搜索栏 -->
      <div class="search-bar">
        <el-input
          v-model="searchKeyword"
          placeholder="请输入地址搜索（如：张家界市武陵源区）"
          clearable
          @keyup.enter.native="searchLocation"
        >
          <el-button slot="append" icon="el-icon-search" @click="searchLocation">搜索</el-button>
        </el-input>
        
        <div class="tool-buttons">
          <el-button 
            :type="drawMode ? 'success' : 'primary'" 
            icon="el-icon-edit" 
            size="small"
            @click="toggleDrawMode"
          >
            {{ drawMode ? '完成绘制' : '开始绘制' }}
          </el-button>
          <el-button 
            type="warning" 
            icon="el-icon-delete" 
            size="small"
            @click="clearPolygon"
            v-if="polygon"
          >
            清除区域
          </el-button>
        </div>
      </div>

      <!-- 地图容器 -->
      <div id="farm-location-map" class="map-container"></div>

      <!-- 操作提示 -->
      <div class="tips" v-if="drawMode">
        <i class="el-icon-info"></i>
        <span>绘制模式：在地图上依次点击标记农场边界点（至少3个点），点击"完成绘制"结束</span>
        <span v-if="tempPath.length > 0" style="margin-left: 10px; color: #4CAF50; font-weight: bold;">
          已标记 {{ tempPath.length }} 个点
        </span>
      </div>
      <div class="tips" v-else style="background: #E3F2FD; border-left-color: #2196F3;">
        <i class="el-icon-info" style="color: #2196F3;"></i>
        <span>搜索提示：可以搜索地名、学校、景点等，如"吉首大学"、"天门山"、"张家界市武陵源区"</span>
      </div>

      <!-- 坐标信息 -->
      <div class="coord-info" v-if="center">
        <el-row :gutter="10">
          <el-col :span="8">
            <el-tag>中心经度: {{ center.lng.toFixed(6) }}</el-tag>
          </el-col>
          <el-col :span="8">
            <el-tag>中心纬度: {{ center.lat.toFixed(6) }}</el-tag>
          </el-col>
          <el-col :span="8">
            <el-tag type="success" v-if="areaSize">面积: {{ areaSize }} m²</el-tag>
          </el-col>
        </el-row>
      </div>
    </div>

    <span slot="footer" class="dialog-footer">
      <el-button @click="handleClose">取 消</el-button>
      <el-button type="primary" @click="handleConfirm" :disabled="!center">确 定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  name: 'FarmLocationSelector',
  props: {
    visible: {
      type: Boolean,
      default: false
    },
    // 初始位置数据
    initialData: {
      type: Object,
      default: () => ({})
    }
  },
  data() {
    return {
      map: null,
      marker: null,
      polygon: null,
      drawMode: false,
      searchKeyword: '',
      center: null,
      coordinates: [],
      areaSize: 0,
      amapKey: '',
      geocoder: null,
      tempPath: [], // 临时绘制路径
      mapInitialized: false, // 地图初始化状态
      selectedPoiIndex: 0, // 选中的POI索引
      tempMarkers: [] // 临时标记点（绘制模式）
    }
  },
  watch: {
    visible(val) {
      if (val && !this.mapInitialized) {
        this.$nextTick(() => {
          this.initMap()
        })
      }
    }
  },
  beforeDestroy() {
    // 组件销毁时清理地图
    this.destroyMap()
  },
  methods: {
    async initMap() {
      if (this.mapInitialized) {
        return
      }

      try {
        // 获取高德地图Key
        if (!this.amapKey) {
          const res = await this.request.get('/aether/weather/config')
          if (!res.data || !res.data.jsKey) {
            throw new Error('无法获取地图配置')
          }
          this.amapKey = res.data.jsKey
        }

        // 加载高德地图脚本
        await this.loadAmapScript()
        
        // 检查地图容器是否存在
        const container = document.getElementById('farm-location-map')
        if (!container) {
          throw new Error('地图容器不存在')
        }

        // 创建地图
        const defaultCenter = [110.479191, 29.117096] // 张家界默认中心点
        const initialCenter = this.initialData.centerLng && this.initialData.centerLat
          ? [Number(this.initialData.centerLng), Number(this.initialData.centerLat)]
          : defaultCenter

        this.map = new window.AMap.Map('farm-location-map', {
          zoom: 13,
          center: initialCenter,
          mapStyle: 'amap://styles/normal'
        })

        // 添加工具栏
        this.map.addControl(new window.AMap.ToolBar({
          position: 'RT'
        }))

        // 添加比例尺
        this.map.addControl(new window.AMap.Scale())

        // 创建地理编码器
        this.geocoder = new window.AMap.Geocoder()

        // 如果有初始数据，显示
        if (this.initialData.centerLng && this.initialData.centerLat) {
          this.center = {
            lng: Number(this.initialData.centerLng),
            lat: Number(this.initialData.centerLat)
          }
          this.addMarker(this.center)

          // 如果有区域数据，显示多边形
          if (this.initialData.coordinates) {
            try {
              const coords = typeof this.initialData.coordinates === 'string'
                ? JSON.parse(this.initialData.coordinates)
                : this.initialData.coordinates
              
              if (Array.isArray(coords) && coords.length >= 3) {
                this.coordinates = coords
                this.drawPolygon(coords)
              }
            } catch (e) {
              console.error('解析坐标数据失败', e)
              this.$message.warning('历史区域数据加载失败')
            }
          }
        }

        // 地图点击事件
        this.map.on('click', this.onMapClick)
        
        this.mapInitialized = true
      } catch (error) {
        console.error('地图初始化失败', error)
        this.$message.error('地图初始化失败：' + error.message)
      }
    },

    loadAmapScript() {
      return new Promise((resolve, reject) => {
        if (window.AMap) {
          resolve()
          return
        }

        const script = document.createElement('script')
        script.src = `https://webapi.amap.com/maps?v=2.0&key=${this.amapKey}&plugin=AMap.Geocoder,AMap.PlaceSearch,AMap.ToolBar,AMap.Scale,AMap.Polygon,AMap.Marker`
        script.onload = resolve
        script.onerror = reject
        document.head.appendChild(script)
      })
    },

    // 搜索地址 - 使用POI搜索或地理编码
    searchLocation() {
      if (!this.searchKeyword || !this.searchKeyword.trim()) {
        this.$message.warning('请输入搜索关键词')
        return
      }

      const keyword = this.searchKeyword.trim()
      
      // 使用 AMap.PlaceSearch 进行POI搜索
      window.AMap.plugin('AMap.PlaceSearch', () => {
        const placeSearch = new window.AMap.PlaceSearch({
          city: '张家界', // 默认城市
          citylimit: false, // 不限制城市范围
          pageSize: 10, // 返回10条结果
          extensions: 'all' // 返回详细信息
        })

        placeSearch.search(keyword, (status, result) => {
          if (status === 'complete' && result.poiList && result.poiList.pois && result.poiList.pois.length > 0) {
            // POI搜索成功
            const pois = result.poiList.pois
            
            if (pois.length === 1) {
              // 只有一个结果，直接定位
              this.handleSearchResult(pois[0])
            } else {
              // 多个结果，显示选择列表
              this.showSearchResults(pois)
            }
          } else {
            // POI搜索失败，尝试地理编码
            this.searchWithGeocode(keyword)
          }
        })
      })
    },

    // 处理搜索结果
    handleSearchResult(poi) {
      const location = poi.location
      this.center = {
        lng: Number(location.lng),
        lat: Number(location.lat)
      }
      
      this.map.setCenter([location.lng, location.lat])
      this.map.setZoom(16)
      this.addMarker(this.center)
      
      const addressInfo = poi.address || poi.pname + poi.cityname + poi.adname
      this.$message.success({
        message: `定位成功：${poi.name}`,
        description: addressInfo,
        duration: 3000
      })
    },

    // 显示多个搜索结果供用户选择
    showSearchResults(pois) {
      const items = pois.slice(0, 5).map((poi, index) => {
        const address = poi.address || `${poi.pname}${poi.cityname}${poi.adname}`
        return {
          label: `${index + 1}. ${poi.name} - ${address}`,
          value: index,
          poi: poi
        }
      })

      this.$msgbox({
        title: '搜索到多个结果，请选择：',
        message: this.$createElement('el-radio-group', {
          props: { value: 0 },
          on: {
            input: (val) => {
              this.selectedPoiIndex = val
            }
          }
        }, items.map(item => 
          this.$createElement('el-radio', {
            props: { label: item.value },
            style: { display: 'block', marginBottom: '10px' }
          }, item.label)
        )),
        showCancelButton: true,
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        beforeClose: (action, instance, done) => {
          if (action === 'confirm') {
            const selectedPoi = pois[this.selectedPoiIndex || 0]
            this.handleSearchResult(selectedPoi)
          }
          done()
        }
      }).catch(() => {})
    },

    // 使用地理编码搜索
    searchWithGeocode(keyword) {
      if (!this.geocoder) {
        this.$message.error('地理编码器未初始化，请稍后再试')
        return
      }

      this.geocoder.getLocation(keyword, (status, result) => {
        if (status === 'complete' && result.geocodes && result.geocodes.length) {
          const location = result.geocodes[0].location
          this.center = {
            lng: Number(location.lng),
            lat: Number(location.lat)
          }
          
          this.map.setCenter([location.lng, location.lat])
          this.map.setZoom(15)
          this.addMarker(this.center)
          
          this.$message.success(`定位成功：${result.geocodes[0].formattedAddress || keyword}`)
        } else {
          this.$message.error('未找到该地址，请尝试更详细的地址描述')
        }
      })
    },

    // 添加标记
    addMarker(position) {
      if (!position || typeof position.lng !== 'number' || typeof position.lat !== 'number') {
        console.error('无效的位置数据', position)
        return
      }

      // 移除旧标记
      if (this.marker) {
        this.map.remove(this.marker)
      }

      // 添加新标记
      this.marker = new window.AMap.Marker({
        position: [position.lng, position.lat],
        title: '农场中心',
        icon: new window.AMap.Icon({
          size: new window.AMap.Size(32, 32),
          image: '//a.amap.com/jsapi_demos/static/demo-center/icons/poi-marker-red.png',
          imageSize: new window.AMap.Size(32, 32)
        })
      })

      this.map.add(this.marker)
    },

    // 地图点击事件
    onMapClick(e) {
      if (!e || !e.lnglat) {
        return
      }

      if (this.drawMode) {
        // 绘制模式：添加多边形顶点
        const point = {
          lng: Number(e.lnglat.lng),
          lat: Number(e.lnglat.lat)
        }
        this.tempPath.push(point)

        // 添加顶点标记
        const vertexMarker = new window.AMap.Marker({
          position: [point.lng, point.lat],
          icon: new window.AMap.Icon({
            size: new window.AMap.Size(20, 20),
            image: '//a.amap.com/jsapi_demos/static/demo-center/icons/poi-marker-default.png',
            imageSize: new window.AMap.Size(20, 20)
          }),
          offset: new window.AMap.Pixel(-10, -10),
          label: {
            content: `<div style="background: #4CAF50; color: white; padding: 2px 6px; border-radius: 3px; font-size: 12px;">${this.tempPath.length}</div>`,
            offset: new window.AMap.Pixel(0, -30)
          }
        })
        this.map.add(vertexMarker)
        this.tempMarkers.push(vertexMarker)

        // 实时绘制多边形
        if (this.tempPath.length >= 2) {
          this.drawPolygon(this.tempPath, true)
        }

        // 提示用户进度
        if (this.tempPath.length === 1) {
          this.$message.info('已添加第1个点，继续点击添加边界点')
        } else if (this.tempPath.length === 2) {
          this.$message.info('已添加第2个点，至少需要3个点形成区域')
        } else {
          this.$message.success(`已添加第${this.tempPath.length}个点`)
        }
      } else {
        // 普通模式：设置中心点
        this.center = {
          lng: Number(e.lnglat.lng),
          lat: Number(e.lnglat.lat)
        }
        this.addMarker(this.center)
      }
    },

    // 绘制多边形
    drawPolygon(path, isTemp = false) {
      if (!path || !Array.isArray(path) || path.length < 2) {
        console.error('无效的路径数据', path)
        return
      }

      // 移除旧多边形
      if (this.polygon) {
        this.map.remove(this.polygon)
      }

      // 转换坐标格式并验证
      const amapPath = path.map(p => {
        if (!p || typeof p.lng !== 'number' || typeof p.lat !== 'number') {
          console.error('无效的坐标点', p)
          return null
        }
        return [p.lng, p.lat]
      }).filter(p => p !== null)

      if (amapPath.length < 2) {
        console.error('有效坐标点不足')
        return
      }

      // 创建多边形
      this.polygon = new window.AMap.Polygon({
        path: amapPath,
        strokeColor: '#4CAF50',
        strokeWeight: 3,
        strokeOpacity: 0.8,
        fillColor: isTemp ? '#81C784' : '#4CAF50',
        fillOpacity: isTemp ? 0.2 : 0.35,
        zIndex: 50
      })

      this.map.add(this.polygon)

      // 计算面积（仅在完成绘制时）
      if (!isTemp && path.length >= 3) {
        try {
          this.areaSize = Math.round(this.polygon.getArea())
          
          // 计算中心点
          const bounds = this.polygon.getBounds()
          const center = bounds.getCenter()
          this.center = {
            lng: Number(center.lng),
            lat: Number(center.lat)
          }
          this.addMarker(this.center)
        } catch (error) {
          console.error('计算面积失败', error)
        }
      }
    },

    // 切换绘制模式
    toggleDrawMode() {
      this.drawMode = !this.drawMode
      
      if (this.drawMode) {
        this.tempPath = []
        this.$message.info('请在地图上点击标记农场边界')
      } else {
        // 完成绘制
        if (this.tempPath.length < 3) {
          this.$message.warning('至少需要3个点才能形成区域')
          this.tempPath = []
          return
        }

        this.coordinates = [...this.tempPath]
        this.drawPolygon(this.coordinates, false)
        this.tempPath = []
        this.$message.success('区域绘制完成')
      }
    },

    // 清除多边形
    clearPolygon() {
      // 清除多边形
      if (this.polygon) {
        this.map.remove(this.polygon)
        this.polygon = null
      }
      
      // 清除所有临时顶点标记
      if (this.tempMarkers && this.tempMarkers.length > 0) {
        this.map.remove(this.tempMarkers)
        this.tempMarkers = []
      }
      
      this.coordinates = []
      this.areaSize = 0
      this.tempPath = []
      this.drawMode = false
      
      this.$message.success('区域已清除')
    },

    // 确认选择
    handleConfirm() {
      if (!this.center || typeof this.center.lng !== 'number' || typeof this.center.lat !== 'number') {
        this.$message.warning('请先选择或搜索农场位置')
        return
      }

      // 验证经纬度范围
      if (this.center.lng < -180 || this.center.lng > 180 || 
          this.center.lat < -90 || this.center.lat > 90) {
        this.$message.error('坐标数据异常，请重新选择')
        return
      }

      const data = {
        centerLng: this.center.lng,
        centerLat: this.center.lat,
        coordinates: this.coordinates.length >= 3 ? JSON.stringify(this.coordinates) : null,
        areaSize: this.areaSize
      }

      this.$emit('confirm', data)
      this.handleClose()
    },

    // 关闭对话框
    handleClose() {
      this.$emit('update:visible', false)
      this.destroyMap()
    },

    // 销毁地图
    destroyMap() {
      // 清理临时标记
      if (this.tempMarkers && this.tempMarkers.length > 0 && this.map) {
        this.map.remove(this.tempMarkers)
        this.tempMarkers = []
      }
      
      // 清理地图
      if (this.map) {
        this.map.destroy()
        this.map = null
      }
      
      // 重置状态
      this.marker = null
      this.polygon = null
      this.geocoder = null
      this.drawMode = false
      this.tempPath = []
      this.selectedPoiIndex = 0
      this.mapInitialized = false
    }
  }
}
</script>

<style scoped>
.location-selector {
  padding: 10px;
}

.search-bar {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
  align-items: center;
}

.search-bar .el-input {
  flex: 1;
}

.tool-buttons {
  display: flex;
  gap: 10px;
}

.map-container {
  width: 100%;
  height: 500px;
  border: 2px solid #4CAF50;
  border-radius: 8px;
  overflow: hidden;
}

.tips {
  margin-top: 10px;
  padding: 10px;
  background: #E8F5E9;
  border-left: 4px solid #4CAF50;
  border-radius: 4px;
  color: #2C5530;
  display: flex;
  align-items: center;
  gap: 8px;
}

.tips i {
  font-size: 18px;
  color: #4CAF50;
}

.coord-info {
  margin-top: 15px;
  padding: 10px;
  background: #F5F5F5;
  border-radius: 4px;
}

.coord-info .el-tag {
  font-size: 13px;
  padding: 8px 12px;
}

/* 对话框样式 */
::v-deep .el-dialog {
  border-radius: 12px;
}

::v-deep .el-dialog__header {
  background: linear-gradient(135deg, rgba(76, 175, 80, 0.1) 0%, rgba(129, 199, 132, 0.1) 100%);
  border-bottom: 2px solid rgba(76, 175, 80, 0.2);
  padding: 20px;
}

::v-deep .el-dialog__title {
  color: #2C5530;
  font-weight: 600;
  font-size: 18px;
}

::v-deep .el-dialog__body {
  padding: 20px;
}
</style>

