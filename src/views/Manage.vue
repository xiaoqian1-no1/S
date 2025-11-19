<template>
  <el-container class="green-theme-container">

    <el-aside :width="sideWidth + 'px'" class="green-sidebar">
      <Aside :isCollapse="isCollapse" :logoTextShow="logoTextShow" />
    </el-aside>

    <el-container class="main-container">
      <el-header class="green-header-wrapper">
        <Header :collapseBtnClass="collapseBtnClass" @asideCollapse="collapse" :user="user" />
      </el-header>

      <el-main class="green-main">
        <router-view @refreshUser="getUser" />
      </el-main>

    </el-container>
  </el-container>
</template>

<script>

import Aside from "@/components/Aside";
import Header from "@/components/Header";

export default {
  name: 'Home',
  data() {
    return {
      collapseBtnClass: 'el-icon-s-fold',
      isCollapse: false,
      sideWidth: 200,
      logoTextShow: true,
      user: {}
    }
  },
  components: {
    Aside,
    Header
  },
  created() {
    // 从后台获取最新的User数据
    this.getUser()
  },
  methods: {
    collapse() {  // 点击收缩按钮触发
      this.isCollapse = !this.isCollapse
      if (this.isCollapse) {  // 收缩
        this.sideWidth = 64
        this.collapseBtnClass = 'el-icon-s-unfold'
        this.logoTextShow = false
      } else {   // 展开
        this.sideWidth = 200
        this.collapseBtnClass = 'el-icon-s-fold'
        this.logoTextShow = true
      }
    },
    getUser() {
      let username = "";
      try {
        const userStr = localStorage.getItem("user");
        const user = userStr ? JSON.parse(userStr) : null;
        username = user ? user.username : "";
      } catch (e) {
        console.error('解析用户信息失败:', e);
        username = "";
      }
      
      if (username) {
        // 从后台获取User数据
        this.request.get("/user/username/" + username).then(res => {
          // 重新赋值后台的最新User数据
          this.user = res.data
        })
      }
    }
  }
}
</script>

<style scoped>
/* 清新绿主题布局 */
.green-theme-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #f1f8e9 0%, #e8f5e9 50%, #dcedc8 100%);
}

.green-sidebar {
  box-shadow: 4px 0 20px rgba(76, 175, 80, 0.15);
  border-right: 3px solid #4caf50;
  background: transparent;
}

.main-container {
  background: transparent;
}

.green-header-wrapper {
  border-bottom: none;
  padding: 0;
}

.green-main {
  background: transparent;
  padding: 20px;
  min-height: calc(100vh - 60px);
}

/* 全局卡片样式 */
:deep(.el-card) {
  background: rgba(255, 255, 255, 0.85) !important;
  backdrop-filter: blur(10px);
  border: 2px solid rgba(76, 175, 80, 0.2);
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(76, 175, 80, 0.1);
  transition: all 0.3s ease;
}

:deep(.el-card:hover) {
  transform: translateY(-2px);
  box-shadow: 0 8px 30px rgba(76, 175, 80, 0.2);
  border-color: rgba(76, 175, 80, 0.4);
}

:deep(.el-card__header) {
  background: linear-gradient(135deg, rgba(76, 175, 80, 0.1) 0%, rgba(129, 199, 132, 0.1) 100%);
  border-bottom: 2px solid rgba(76, 175, 80, 0.2);
  font-weight: 600;
  color: #2c5530;
}

/* 按钮样式 */
:deep(.el-button--primary) {
  background: linear-gradient(135deg, #4caf50 0%, #66bb6a 100%);
  border: none;
  box-shadow: 0 2px 8px rgba(76, 175, 80, 0.3);
  transition: all 0.3s ease;
}

:deep(.el-button--primary:hover) {
  transform: translateY(-2px);
  box-shadow: 0 4px 15px rgba(76, 175, 80, 0.4);
}

:deep(.el-button--success) {
  background: linear-gradient(135deg, #66bb6a 0%, #81c784 100%);
  border: none;
}

:deep(.el-button--warning) {
  background: linear-gradient(135deg, #ff9800 0%, #ffb74d 100%);
  border: none;
}

/* 表格样式 */
:deep(.el-table) {
  background: rgba(255, 255, 255, 0.9);
  border-radius: 8px;
  overflow: hidden;
}

:deep(.el-table th) {
  background: linear-gradient(135deg, #e8f5e9 0%, #c8e6c9 100%);
  color: #2c5530;
  font-weight: 600;
}

:deep(.el-table--striped .el-table__body tr.el-table__row--striped td) {
  background: rgba(76, 175, 80, 0.05);
}

/* 输入框样式 */
:deep(.el-input__inner) {
  border: 2px solid #e0e0e0;
  border-radius: 6px;
  transition: all 0.3s ease;
}

:deep(.el-input__inner:focus) {
  border-color: #4caf50;
  box-shadow: 0 0 8px rgba(76, 175, 80, 0.3);
}

/* 标签样式 */
:deep(.el-tag--success) {
  background: rgba(76, 175, 80, 0.1);
  border-color: #4caf50;
  color: #2c5530;
}
</style>

