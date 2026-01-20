<script setup>
import { ref } from 'vue'

const emit = defineEmits(['navigate'])

const tools = [
  {
    id: 'time-converter',
    name: 'Unix 时间转换',
    description: 'Unix 时间戳与可读时间相互转换',
    icon: '⏰',
    color: '#667eea'
  },
  {
    id: 'json-formatter',
    name: 'JSON 格式化',
    description: '格式化、验证和美化 JSON 数据',
    icon: '📝',
    color: '#f093fb'
  },
  {
    id: 'url-encoder',
    name: 'URL 编解码',
    description: 'URL 编码和解码工具',
    icon: '🔗',
    color: '#4facfe'
  },
  {
    id: 'base64-converter',
    name: 'Base64 编解码',
    description: 'Base64 编码和解码工具',
    icon: '🔐',
    color: '#43e97b'
  },
  {
    id: 'hash-generator',
    name: '哈希生成器',
    description: '生成 MD5、SHA1、SHA256 等哈希值',
    icon: '🔑',
    color: '#fa709a'
  },
  {
    id: 'qr-generator',
    name: '二维码生成器',
    description: '生成文本或 URL 的二维码',
    icon: '📱',
    color: '#30cfd0'
  }
]

const navigateToTool = (toolId) => {
  emit('navigate', toolId)
}
</script>

<template>
  <div class="home-container">
    <header class="header">
      <h1>🛠️ 实用工具集合</h1>
      <p class="subtitle">便捷的在线工具，提高工作效率</p>
    </header>

    <div class="tools-grid">
      <div
        v-for="tool in tools"
        :key="tool.id"
        class="tool-card"
        @click="navigateToTool(tool.id)"
        :style="{ '--card-color': tool.color }"
      >
        <div class="tool-icon">{{ tool.icon }}</div>
        <h3 class="tool-name">{{ tool.name }}</h3>
        <p class="tool-description">{{ tool.description }}</p>
        <div class="tool-arrow">→</div>
      </div>
    </div>

    <footer class="footer">
      <p>更多工具正在开发中...</p>
    </footer>
  </div>
</template>

<style scoped>
.home-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 40px 20px;
}

.header {
  text-align: center;
  color: white;
  margin-bottom: 50px;
}

.header h1 {
  font-size: 3em;
  margin: 0 0 10px 0;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
}

.subtitle {
  font-size: 1.2em;
  opacity: 0.9;
  margin: 0;
}

.tools-grid {
  max-width: 1200px;
  margin: 0 auto;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 25px;
  padding: 20px;
}

.tool-card {
  background: white;
  border-radius: 15px;
  padding: 30px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  position: relative;
  overflow: hidden;
}

.tool-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: var(--card-color);
  transform: scaleX(0);
  transition: transform 0.3s ease;
}

.tool-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.2);
}

.tool-card:hover::before {
  transform: scaleX(1);
}

.tool-icon {
  font-size: 3em;
  margin-bottom: 15px;
  text-align: center;
}

.tool-name {
  color: #2c3e50;
  font-size: 1.4em;
  margin: 0 0 10px 0;
  text-align: center;
}

.tool-description {
  color: #7f8c8d;
  font-size: 0.95em;
  text-align: center;
  margin: 0 0 15px 0;
  line-height: 1.5;
}

.tool-arrow {
  text-align: center;
  color: var(--card-color);
  font-size: 1.5em;
  font-weight: bold;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.tool-card:hover .tool-arrow {
  opacity: 1;
}

.footer {
  text-align: center;
  color: white;
  margin-top: 50px;
  opacity: 0.8;
}

@media (max-width: 768px) {
  .header h1 {
    font-size: 2em;
  }

  .tools-grid {
    grid-template-columns: 1fr;
    gap: 20px;
  }
}
</style>
