<template>
  <el-menu :default-openeds="opens" style="min-height: 100%; overflow-x: hidden"
           background-color="transparent"
           text-color="#2c5530"
           active-text-color="#ffffff"
           :collapse-transition="false"
           :collapse="isCollapse"
           router
           class="green-sidebar-menu"
  >
    <div class="logo-section">
      <img src="../assets/Logo.jpg" alt="" class="logo-img">
      <b class="logo-text" v-show="logoTextShow">🌿 智慧农田</b>
    </div>
    <div v-for="item in menus" :key="item.id">
      <div v-if="item.path">
        <el-menu-item :index="item.path">
          <i :class="item.icon"></i>
          <span slot="title">{{ item.name }}</span>
        </el-menu-item>
      </div>
      <div v-else>
        <el-submenu :index="item.id + ''">
          <template slot="title">
            <i :class="item.icon"></i>
            <span slot="title">{{ item.name }}</span>
          </template>
          <div  v-for="subItem in item.children" :key="subItem.id">
            <el-menu-item :index="subItem.path">
              <i :class="subItem.icon"></i>
              <span slot="title">{{ subItem.name }}</span>
            </el-menu-item>
          </div>
        </el-submenu>
      </div>
    </div>
  </el-menu>
</template>

<script>
export default {
  name: "Aside",
  props: {
    isCollapse: Boolean,
    logoTextShow: Boolean
  },
  data() {
    return {
      menus: localStorage.getItem("menus") ? JSON.parse(localStorage.getItem("menus")) : [],
      opens: localStorage.getItem("menus") ? JSON.parse(localStorage.getItem("menus")).map(v => v.id + '') : []
    }
  },
}
</script>

<style scoped>
/* 清新绿主题侧边栏 */
.green-sidebar-menu {
  background: linear-gradient(180deg, #e8f5e9 0%, #c8e6c9 50%, #a5d6a7 100%) !important;
  border-right: 3px solid #4caf50;
  box-shadow: 2px 0 12px rgba(76, 175, 80, 0.15);
}

/* Logo区域 */
.logo-section {
  height: 70px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #4caf50 0%, #66bb6a 100%);
  margin-bottom: 10px;
  box-shadow: 0 2px 8px rgba(76, 175, 80, 0.3);
}

.logo-img {
  width: 45px;
  height: 45px;
  border-radius: 50%;
  border: 3px solid white;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.logo-text {
  color: white;
  margin-left: 8px;
  font-size: 18px;
  font-weight: 600;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
}

/* 菜单项样式 */
:deep(.el-menu-item) {
  margin: 4px 8px;
  border-radius: 8px;
  transition: all 0.3s ease;
  color: #2c5530 !important;
  font-weight: 500;
}

:deep(.el-menu-item:hover) {
  background: rgba(76, 175, 80, 0.15) !important;
  transform: translateX(4px);
  box-shadow: 0 2px 8px rgba(76, 175, 80, 0.2);
}

:deep(.el-menu-item.is-active) {
  background: linear-gradient(135deg, #4caf50 0%, #66bb6a 100%) !important;
  color: white !important;
  box-shadow: 0 4px 12px rgba(76, 175, 80, 0.4);
  transform: translateX(6px);
}

:deep(.el-menu-item i) {
  font-size: 18px;
  margin-right: 8px;
}

/* 子菜单样式 */
:deep(.el-submenu) {
  margin: 4px 8px;
  border-radius: 8px;
}

:deep(.el-submenu__title) {
  color: #2c5530 !important;
  font-weight: 600;
  border-radius: 8px;
  transition: all 0.3s ease;
}

:deep(.el-submenu__title:hover) {
  background: rgba(76, 175, 80, 0.15) !important;
  transform: translateX(4px);
}

:deep(.el-submenu.is-active .el-submenu__title) {
  color: #4caf50 !important;
  background: rgba(76, 175, 80, 0.1) !important;
}

:deep(.el-submenu .el-menu-item) {
  margin: 2px 12px;
  padding-left: 35px !important;
  background: rgba(255, 255, 255, 0.5);
  border-left: 3px solid transparent;
}

:deep(.el-submenu .el-menu-item:hover) {
  border-left-color: #4caf50;
  background: rgba(76, 175, 80, 0.1) !important;
}

:deep(.el-submenu .el-menu-item.is-active) {
  border-left-color: #4caf50;
  background: rgba(76, 175, 80, 0.2) !important;
  color: #2c5530 !important;
  font-weight: 600;
}

/* 收缩状态 */
:deep(.el-menu--collapse) {
  width: 64px;
}

:deep(.el-menu--collapse .el-menu-item) {
  margin: 4px 4px;
  text-align: center;
}

:deep(.el-menu--collapse span) {
  visibility: hidden;
}

/* 图标美化 */
:deep(.el-menu-item i),
:deep(.el-submenu__title i) {
  color: #4caf50;
  transition: all 0.3s ease;
}

:deep(.el-menu-item.is-active i),
:deep(.el-submenu.is-active .el-submenu__title i) {
  color: white;
  transform: scale(1.1);
}
</style>
