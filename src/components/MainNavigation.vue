<template>
  <nav class="side-nav">
    <div class="nav-header">
      <h2>埋点分析</h2>
    </div>
    <div class="nav-content">
      <router-link to="/" class="nav-item" active-class="active">
        <span class="nav-icon">📊</span>
        首页概览
      </router-link>
      <!-- 埋点管理 -->
      <div class="nav-group">
        <div class="nav-group-title">埋点管理</div>
        <router-link to="/track/guide" class="nav-item" active-class="active">
          <span class="nav-icon">📘</span>
          埋点意义
        </router-link>
        <router-link to="/track/overview" class="nav-item" active-class="active">
          <span class="nav-icon">📊</span>
          埋点概览
        </router-link>
        <router-link to="/track/list" class="nav-item" active-class="active">
          <span class="nav-icon">📋</span>
          事件列表
        </router-link>
        <router-link to="/track/create" class="nav-item" active-class="active">
          <span class="nav-icon">📝</span>
          创建埋点
        </router-link>

      </div>
      <!-- 用户管理 -->
      <div class="nav-group">
        <div class="nav-group-title">用户管理</div>
        <router-link to="/user/list" class="nav-item" active-class="active">
          <span class="nav-icon">👥</span>
          用户列表
        </router-link>
        <router-link to="/user/sessions" class="nav-item" active-class="active">
          <span class="nav-icon">⏱️</span>
          会话分析
        </router-link>
        <router-link to="/user/heatmap" class="nav-item" active-class="active">
          <span class="nav-icon">🔥</span>
          热力图
        </router-link>
      </div>
      <!-- 错误监控 -->
      <div class="nav-group">
        <div class="nav-group-title">错误监控</div>
        <router-link to="/monitor/errors" class="nav-item" active-class="active">
          <span class="nav-icon">🐛</span>
          错误分析
        </router-link>
      </div>
      <!-- 漏斗分析 -->
      <div class="nav-group">
        <div class="nav-group-title">漏斗分析</div>
        <router-link to="/funnel/list" class="nav-item" active-class="active">
          <span class="nav-icon">🔍</span>
          漏斗列表
        </router-link>
        <router-link to="/funnel/create" class="nav-item" active-class="active">
          <span class="nav-icon">➕</span>
          创建漏斗
        </router-link>
        <router-link to="/funnel/analysis" class="nav-item" active-class="active">
          <span class="nav-icon">📈</span>
          漏斗分析
        </router-link>
      </div>
      <!-- 应用管理 -->
      <div class="nav-group">
        <div class="nav-group-title">应用管理</div>
        <router-link to="/apps" class="nav-item" active-class="active">
          <span class="nav-icon">📱</span>
          应用列表
        </router-link>
        <router-link to="/apps/create" class="nav-item" active-class="active">
          <span class="nav-icon">➕</span>
          创建应用
        </router-link>
      </div>
      <!-- 分析工具 -->
      <div class="nav-group">
        <div class="nav-group-title">分析工具</div>
        <router-link to="/path-analysis" class="nav-item" active-class="active">
          <span class="nav-icon">🔍</span>
          路径分析
        </router-link>
      </div>
    </div>
  </nav>
</template>

<style scoped>
.side-nav {
  width: 200px;
  background-color: #f5f5f5;
  padding: 20px;
}

.nav-header {
  margin-bottom: 20px;
}

.nav-item {
  display: block;
  margin-bottom: 10px;
  color: #333;
  text-decoration: none;
}

.nav-item.active {
  font-weight: bold;
}

.nav-group {
  margin-top: 20px;
}

.nav-group-title {
  font-weight: bold;
  margin-bottom: 10px;
}

.nav-icon {
  margin-right: 8px;
}
</style>