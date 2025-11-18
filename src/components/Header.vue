<template>
  <div class="green-header">
    <div class="header-left">
      <span :class="collapseBtnClass" class="collapse-btn" @click="collapse"></span>

      <el-breadcrumb separator="/" class="breadcrumb">
        <el-breadcrumb-item :to="'/'">
          <i class="el-icon-s-home"></i>
          首页
        </el-breadcrumb-item>
        <el-breadcrumb-item>{{ currentPathName }}</el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    
    <div class="header-center">
      <div class="system-title">
        <i class="el-icon-s-data"></i>
        智慧农田管理系统
      </div>
    </div>
    
    <el-dropdown class="user-dropdown">
      <div class="user-info">
        <img :src="user.avatarUrl" alt="" class="user-avatar">
        <span class="user-name">{{ user.nickname }}</span>
        <i class="el-icon-arrow-down"></i>
      </div>
      <el-dropdown-menu slot="dropdown" class="user-menu">
        <el-dropdown-item>
          <router-link to="/person" class="menu-link">
            <i class="el-icon-user"></i>
            个人信息
          </router-link>
        </el-dropdown-item>
        <el-dropdown-item>
          <router-link to="/password" class="menu-link">
            <i class="el-icon-key"></i>
            修改密码
          </router-link>
        </el-dropdown-item>
        <el-dropdown-item divided>
          <div class="menu-link logout-btn" @click="logout">
            <i class="el-icon-switch-button"></i>
            退出登录
          </div>
        </el-dropdown-item>
      </el-dropdown-menu>
    </el-dropdown>
  </div>
</template>

<script>
export default {
  name: "Header",
  props: {
    collapseBtnClass: String,
    user: Object
  },
  computed: {
    currentPathName () {
      return this.$store.state.currentPathName;　　//需要监听的数据
    }
  },
  data() {
    return {

    }
  },
  methods: {
    collapse() {
      // this.$parent.$parent.$parent.$parent.collapse()  // 通过4个 $parent 找到父组件，从而调用其折叠方法
      this.$emit("asideCollapse")
    },
    logout() {
      this.$store.commit("logout")
      this.$message.success("退出成功")
    }
  }
}
</script>

<style scoped>
/* 清新绿主题头部 */
.green-header {
  height: 60px;
  display: flex;
  align-items: center;
  background: linear-gradient(135deg, #e8f5e9 0%, #c8e6c9 100%);
  border-bottom: 3px solid #4caf50;
  box-shadow: 0 2px 12px rgba(76, 175, 80, 0.15);
  padding: 0 20px;
}

.header-left {
  display: flex;
  align-items: center;
  flex: 1;
}

.collapse-btn {
  cursor: pointer;
  font-size: 20px;
  color: #4caf50;
  padding: 8px;
  border-radius: 6px;
  transition: all 0.3s ease;
}

.collapse-btn:hover {
  background: rgba(76, 175, 80, 0.1);
  transform: scale(1.1);
}

.breadcrumb {
  margin-left: 15px;
}

:deep(.breadcrumb .el-breadcrumb__item) {
  font-weight: 500;
}

:deep(.breadcrumb .el-breadcrumb__inner) {
  color: #2c5530;
}

:deep(.breadcrumb .el-breadcrumb__inner:hover) {
  color: #4caf50;
}

.header-center {
  flex: 1;
  display: flex;
  justify-content: center;
}

.system-title {
  font-size: 20px;
  font-weight: 600;
  color: #2c5530;
  display: flex;
  align-items: center;
  gap: 8px;
}

.system-title i {
  font-size: 24px;
  color: #4caf50;
}

.user-dropdown {
  cursor: pointer;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.8);
  border: 2px solid #4caf50;
  transition: all 0.3s ease;
}

.user-info:hover {
  background: rgba(76, 175, 80, 0.1);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(76, 175, 80, 0.3);
}

.user-avatar {
  width: 35px;
  height: 35px;
  border-radius: 50%;
  border: 2px solid #4caf50;
  object-fit: cover;
}

.user-name {
  font-weight: 600;
  color: #2c5530;
  font-size: 14px;
}

.user-info i {
  color: #4caf50;
  font-size: 12px;
}

/* 下拉菜单 */
.user-menu {
  border: 2px solid #4caf50;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(76, 175, 80, 0.2);
}

.menu-link {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #2c5530;
  text-decoration: none;
  font-weight: 500;
  transition: all 0.3s ease;
}

.menu-link:hover {
  color: #4caf50;
}

.menu-link i {
  font-size: 16px;
}

.logout-btn {
  cursor: pointer;
}

.logout-btn:hover {
  color: #f56c6c !important;
}
</style>
