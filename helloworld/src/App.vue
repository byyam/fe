<script setup>
import { ref } from 'vue'
import Home from './components/Home.vue'
import TimeConverter from './components/TimeConverter.vue'
import NewlineUnescape from './components/NewlineUnescape.vue'
import Placeholder from './components/Placeholder.vue'

const currentPage = ref('home')

const toolNames = {
  'time-converter': 'Unix 时间转换',
  'newline-unescape': '\\n 转换行',
  'json-formatter': 'JSON 格式化',
  'url-encoder': 'URL 编解码',
  'base64-converter': 'Base64 编解码',
  'hash-generator': '哈希生成器',
  'qr-generator': '二维码生成器'
}

const navigateTo = (page) => {
  currentPage.value = page
}

const goHome = () => {
  currentPage.value = 'home'
}
</script>

<template>
  <div class="app-container">
    <!-- 导航栏 -->
    <nav v-if="currentPage !== 'home'" class="navbar">
      <button @click="goHome" class="nav-home-btn">← 返回首页</button>
      <h2 class="nav-title">{{ toolNames[currentPage] || '工具' }}</h2>
    </nav>

    <!-- 页面内容 -->
    <Home v-if="currentPage === 'home'" @navigate="navigateTo" />
    <TimeConverter v-else-if="currentPage === 'time-converter'" />
    <NewlineUnescape v-else-if="currentPage === 'newline-unescape'" />
    <Placeholder v-else :tool-name="toolNames[currentPage] || '工具'" />
  </div>
</template>

<style scoped>
.app-container {
  min-height: 100vh;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.navbar {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 15px 20px;
  display: flex;
  align-items: center;
  gap: 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  position: sticky;
  top: 0;
  z-index: 100;
}

.nav-home-btn {
  background: rgba(255, 255, 255, 0.2);
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.3);
  padding: 8px 16px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.3s;
}

.nav-home-btn:hover {
  background: rgba(255, 255, 255, 0.3);
}

.nav-title {
  margin: 0;
  font-size: 1.3em;
  font-weight: 500;
}

@media (max-width: 600px) {
  .navbar {
    padding: 12px 15px;
  }

  .nav-title {
    font-size: 1.1em;
  }

  .nav-home-btn {
    padding: 6px 12px;
    font-size: 12px;
  }
}
</style>
