<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const currentTime = ref('')
const unixTimestamp = ref('')
const convertedTime = ref('')
const inputDateTime = ref('')
const convertedTimestamp = ref('')
let timer = null

// 格式化时间显示
const formatTime = (date) => {
  return date.toLocaleString('zh-CN', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
    hour12: false
  })
}

// 更新当前时间
const updateCurrentTime = () => {
  const now = new Date()
  currentTime.value = formatTime(now)
}

// Unix 时间戳转可读时间
const convertTimestampToTime = () => {
  if (!unixTimestamp.value) {
    convertedTime.value = ''
    return
  }
  
  let timestamp = parseFloat(unixTimestamp.value)
  
  // 判断是秒还是毫秒（大于 10 位数字认为是毫秒）
  if (timestamp.toString().length <= 10) {
    timestamp = timestamp * 1000
  }
  
  const date = new Date(timestamp)
  if (isNaN(date.getTime())) {
    convertedTime.value = '无效的时间戳'
    return
  }
  
  convertedTime.value = formatTime(date)
}

// 可读时间转 Unix 时间戳
const convertTimeToTimestamp = () => {
  if (!inputDateTime.value) {
    convertedTimestamp.value = ''
    return
  }
  
  const date = new Date(inputDateTime.value)
  if (isNaN(date.getTime())) {
    convertedTimestamp.value = '无效的日期时间'
    return
  }
  
  // 返回秒级时间戳
  convertedTimestamp.value = Math.floor(date.getTime() / 1000).toString()
}

// 复制到剪贴板
const copyToClipboard = (text) => {
  navigator.clipboard.writeText(text).then(() => {
    alert('已复制到剪贴板')
  }).catch(() => {
    alert('复制失败')
  })
}

// 使用当前时间填充输入框
const useCurrentTime = () => {
  const now = new Date()
  inputDateTime.value = now.toISOString().slice(0, 16) // YYYY-MM-DDTHH:mm 格式
  convertTimeToTimestamp()
}

// 使用当前时间戳填充输入框
const useCurrentTimestamp = () => {
  unixTimestamp.value = Math.floor(Date.now() / 1000).toString()
  convertTimestampToTime()
}

onMounted(() => {
  updateCurrentTime()
  // 每秒更新一次当前时间
  timer = setInterval(updateCurrentTime, 1000)
})

onUnmounted(() => {
  if (timer) {
    clearInterval(timer)
  }
})
</script>

<template>
  <div class="container">
    <h1>Unix 时间转换工具</h1>
    
    <!-- 当前时间显示 -->
    <div class="current-time-section">
      <h2>当前时间</h2>
      <div class="time-display">{{ currentTime }}</div>
    </div>

    <!-- Unix 时间戳转可读时间 -->
    <div class="converter-section">
      <h2>Unix 时间戳 → 可读时间</h2>
      <div class="input-group">
        <label>Unix 时间戳（秒或毫秒）：</label>
        <div class="input-wrapper">
          <input 
            v-model="unixTimestamp" 
            @input="convertTimestampToTime"
            type="text" 
            placeholder="例如：1704067200 或 1704067200000"
          />
          <button @click="useCurrentTimestamp" class="btn-secondary">使用当前时间戳</button>
        </div>
      </div>
      <div v-if="convertedTime" class="result">
        <span>转换结果：</span>
        <span class="result-value">{{ convertedTime }}</span>
        <button @click="copyToClipboard(convertedTime)" class="btn-copy">复制</button>
      </div>
    </div>

    <!-- 可读时间转 Unix 时间戳 -->
    <div class="converter-section">
      <h2>可读时间 → Unix 时间戳</h2>
      <div class="input-group">
        <label>日期时间：</label>
        <div class="input-wrapper">
          <input 
            v-model="inputDateTime" 
            @input="convertTimeToTimestamp"
            type="datetime-local" 
          />
          <button @click="useCurrentTime" class="btn-secondary">使用当前时间</button>
        </div>
      </div>
      <div v-if="convertedTimestamp" class="result">
        <span>转换结果：</span>
        <span class="result-value">{{ convertedTimestamp }}</span>
        <button @click="copyToClipboard(convertedTimestamp)" class="btn-copy">复制</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

h1 {
  text-align: center;
  color: #2c3e50;
  margin-bottom: 30px;
}

h2 {
  color: #34495e;
  font-size: 1.2em;
  margin-bottom: 15px;
  border-bottom: 2px solid #3498db;
  padding-bottom: 8px;
}

.current-time-section {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 25px;
  border-radius: 10px;
  margin-bottom: 30px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.current-time-section h2 {
  color: white;
  border-bottom: 2px solid rgba(255, 255, 255, 0.3);
}

.time-display {
  font-size: 1.8em;
  font-weight: bold;
  text-align: center;
  margin-top: 10px;
  font-family: 'Courier New', monospace;
}

.converter-section {
  background: #f8f9fa;
  padding: 25px;
  border-radius: 10px;
  margin-bottom: 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.input-group {
  margin-bottom: 15px;
}

.input-group label {
  display: block;
  margin-bottom: 8px;
  color: #555;
  font-weight: 500;
}

.input-wrapper {
  display: flex;
  gap: 10px;
  align-items: center;
}

.input-wrapper input {
  flex: 1;
  padding: 12px;
  border: 2px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.input-wrapper input:focus {
  outline: none;
  border-color: #3498db;
}

.btn-secondary {
  padding: 12px 20px;
  background: #95a5a6;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  transition: background 0.3s;
  white-space: nowrap;
}

.btn-secondary:hover {
  background: #7f8c8d;
}

.result {
  margin-top: 15px;
  padding: 15px;
  background: white;
  border-radius: 6px;
  border-left: 4px solid #3498db;
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}

.result-value {
  font-family: 'Courier New', monospace;
  font-size: 1.1em;
  color: #2c3e50;
  font-weight: 500;
  flex: 1;
}

.btn-copy {
  padding: 8px 16px;
  background: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: background 0.3s;
}

.btn-copy:hover {
  background: #2980b9;
}

@media (max-width: 600px) {
  .input-wrapper {
    flex-direction: column;
  }
  
  .input-wrapper input,
  .btn-secondary {
    width: 100%;
  }
  
  .time-display {
    font-size: 1.4em;
  }
}
</style>
