<script setup>
import { ref, computed, watch, provide, onMounted, onUnmounted } from 'vue'
import JsonTree from './JsonTree.vue'

const inputText = ref('')
const parseError = ref('')
const usedAutoUnescape = ref(false)
const previewWrapRef = ref(null)
const isFullscreen = ref(false)
const collapsedPaths = ref({})

const enterFullscreen = () => {
  if (!previewWrapRef.value) return
  previewWrapRef.value.requestFullscreen?.().catch(() => {})
}
const exitFullscreen = () => {
  if (document.fullscreenElement) document.exitFullscreen?.()
}
const onFullscreenChange = () => {
  isFullscreen.value = !!document.fullscreenElement
}
onMounted(() => {
  document.addEventListener('fullscreenchange', onFullscreenChange)
})
onUnmounted(() => {
  document.removeEventListener('fullscreenchange', onFullscreenChange)
})

// 与「去除转义」按钮同逻辑：\" → "，\\ → \，\' → '（JSON 不支持 \'，会报 Bad escaped character）
function unescapeJsonString(s) {
  return s
    .replace(/\\"/g, '"')
    .replace(/\\\\/g, '\\')
    .replace(/\\'/g, "'")
}

// 尝试解析 JSON：直接解析 → 双编码 → 先去除转义再解析
function parseJson(str) {
  const trimmed = str.trim()
  if (!trimmed) return { ok: false, error: '请输入 JSON', usedUnescape: false }

  // 1. 直接解析
  try {
    const parsed = JSON.parse(trimmed)
    return { ok: true, data: parsed, usedUnescape: false }
  } catch (e1) {
    // 2. 双编码：整体是 JSON 字符串
    if (trimmed.startsWith('"') && trimmed.endsWith('"')) {
      try {
        const inner = JSON.parse(trimmed)
        if (typeof inner === 'string') {
          const again = JSON.parse(inner)
          return { ok: true, data: again, usedUnescape: false }
        }
      } catch {}
    }
    // 3. 先去除转义再解析（与「去除转义」按钮一致）
    try {
      const unescaped = unescapeJsonString(trimmed)
      const parsed = JSON.parse(unescaped)
      return { ok: true, data: parsed, usedUnescape: true }
    } catch (e2) {
      const msg = (e2 instanceof SyntaxError && e2.message) ? e2.message : '无效的 JSON，请检查格式或转义'
      return { ok: false, error: msg, usedUnescape: false }
    }
  }
}

// 解析结果（单次解析，供格式化与树共用）
const parseResult = computed(() => {
  const result = parseJson(inputText.value)
  if (!result.ok) return { ok: false, error: result.error, formatted: '', data: null, usedUnescape: false }
  try {
    return {
      ok: true,
      error: '',
      formatted: JSON.stringify(result.data, null, 2),
      data: result.data,
      usedUnescape: result.usedUnescape === true
    }
  } catch {
    return { ok: false, error: '格式化失败', formatted: '', data: null, usedUnescape: false }
  }
})

watch(
  parseResult,
  (r) => {
    parseError.value = r.error || ''
    usedAutoUnescape.value = r.usedUnescape
  },
  { immediate: true }
)

const formattedJson = computed(() => parseResult.value.formatted)
const parsedData = computed(() => parseResult.value.data)

const toggleCollapse = (path) => {
  const cur = collapsedPaths.value[path]
  collapsedPaths.value = { ...collapsedPaths.value, [path]: !cur }
}

provide('collapsedPaths', collapsedPaths)
provide('toggleCollapse', toggleCollapse)

const copyToClipboard = (text) => {
  if (!text) return
  navigator.clipboard.writeText(text).catch(() => {})
}

const clearInput = () => {
  inputText.value = ''
  parseError.value = ''
}

// 去除转义：\" → "，\\ → \，\' → '（与 unescapeJsonString 一致）
const unescapeInput = () => {
  if (!inputText.value) return
  inputText.value = unescapeJsonString(inputText.value)
}

// 历史结果（仅内存，刷新即清空）
const MAX_HISTORY = 20
const historyList = ref([])
let historyId = 0

const saveToHistory = () => {
  if (!formattedJson.value) return
  historyList.value.unshift({
    id: ++historyId,
    input: inputText.value,
    formatted: formattedJson.value
  })
  if (historyList.value.length > MAX_HISTORY) {
    historyList.value = historyList.value.slice(0, MAX_HISTORY)
  }
}

const restoreFromHistory = (item) => {
  inputText.value = item.input
}

const removeFromHistory = (id) => {
  historyList.value = historyList.value.filter((h) => h.id !== id)
}

const historyPreview = (text) => {
  const lines = text.split('\n').slice(0, 2).join('\n')
  return lines.length > 80 ? lines.slice(0, 80) + '…' : lines
}
</script>

<template>
  <div class="container">
    <h1>JSON 去转义并格式化</h1>
    <p class="description">粘贴 JSON 文本，一键去掉多余转义并格式化为易读样式。</p>

    <div class="converter-section">
      <div class="input-group">
        <div class="input-label-row">
          <label>输入 JSON：</label>
          <div class="input-actions">
            <button type="button" @click="unescapeInput" class="btn-unescape" :disabled="!inputText">去除转义</button>
            <button type="button" @click="clearInput" class="btn-clear" :disabled="!inputText">一键清空</button>
          </div>
        </div>
        <textarea
          v-model="inputText"
          class="textarea-input"
          placeholder='例如：{"name":"test"} 或 双编码的 "{\"a\":1}"'
          rows="10"
        />
      </div>

      <p v-if="parseError" class="error-msg">{{ parseError }}</p>
      <p v-else-if="usedAutoUnescape" class="hint-msg">当前为带转义内容，已按去除转义后解析；可点击「去除转义」更新输入框。</p>

      <template v-if="formattedJson">
        <div ref="previewWrapRef" class="result-section-wrap">
          <div class="result-section">
            <div class="result-header">
              <label>格式化结果：</label>
              <div class="result-actions">
                <button @click="copyToClipboard(formattedJson)" class="btn-copy">复制</button>
                <button type="button" @click="isFullscreen ? exitFullscreen() : enterFullscreen()" class="btn-fullscreen">
                  {{ isFullscreen ? '退出全屏' : '全屏' }}
                </button>
              </div>
            </div>
            <pre v-show="!isFullscreen" class="result-pre">{{ formattedJson }}</pre>
            <div v-show="isFullscreen" class="result-tree-wrap">
              <JsonTree v-if="parsedData !== null" :value="parsedData" path="root" label="" />
            </div>
          </div>
        </div>
        <div class="save-history-row">
          <button type="button" @click="saveToHistory" class="btn-save-history">保存到历史</button>
        </div>
      </template>
    </div>

    <!-- 历史结果 -->
    <div v-if="historyList.length" class="history-section">
      <h2 class="history-title">历史结果</h2>
      <p class="history-hint">不刷新页面前会保留，最多 {{ MAX_HISTORY }} 条</p>
      <ul class="history-list">
        <li v-for="item in historyList" :key="item.id" class="history-item">
          <pre class="history-preview">{{ historyPreview(item.formatted) }}</pre>
          <div class="history-actions">
            <button type="button" @click="restoreFromHistory(item)" class="btn-history btn-restore">恢复</button>
            <button type="button" @click="copyToClipboard(item.formatted)" class="btn-history btn-copy-sm">复制</button>
            <button type="button" @click="removeFromHistory(item.id)" class="btn-history btn-remove">删除</button>
          </div>
        </li>
      </ul>
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
  margin-bottom: 10px;
}

.description {
  text-align: center;
  color: #7f8c8d;
  margin-bottom: 25px;
  font-size: 0.95em;
}

.converter-section {
  background: #f8f9fa;
  padding: 25px;
  border-radius: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.input-group {
  margin-bottom: 16px;
}

.input-label-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 8px;
  margin-bottom: 8px;
}

.input-group label {
  margin: 0;
  color: #555;
  font-weight: 500;
}

.input-actions {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.btn-unescape {
  padding: 6px 14px;
  background: #f093fb;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 13px;
  transition: background 0.3s;
}

.btn-unescape:hover:not(:disabled) {
  background: #e082ea;
}

.btn-unescape:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-clear {
  padding: 6px 14px;
  background: #95a5a6;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 13px;
  transition: background 0.3s;
}

.btn-clear:hover:not(:disabled) {
  background: #7f8c8d;
}

.btn-clear:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.textarea-input {
  width: 100%;
  padding: 12px;
  border: 2px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
  font-family: 'Courier New', monospace;
  line-height: 1.5;
  resize: vertical;
  transition: border-color 0.3s;
  box-sizing: border-box;
}

.textarea-input:focus {
  outline: none;
  border-color: #f093fb;
}

.error-msg {
  color: #e74c3c;
  font-size: 14px;
  margin: 0 0 12px 0;
}

.hint-msg {
  color: #7f8c8d;
  font-size: 13px;
  margin: 0 0 12px 0;
}

.result-section {
  margin-top: 20px;
  padding: 15px;
  background: white;
  border-radius: 6px;
  border-left: 4px solid #f093fb;
}

.result-section-wrap {
  margin-top: 20px;
}

.result-section-wrap .result-section {
  margin-top: 0;
}

.result-section-wrap:fullscreen,
.result-section-wrap:-webkit-full-screen {
  background: #f8f9fa;
  padding: 20px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.result-section-wrap:fullscreen .result-section,
.result-section-wrap:-webkit-full-screen .result-section {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 0;
  margin-top: 0;
}

.result-section-wrap:fullscreen .result-pre,
.result-section-wrap:-webkit-full-screen .result-pre,
.result-section-wrap:fullscreen .result-tree-wrap,
.result-section-wrap:-webkit-full-screen .result-tree-wrap {
  max-height: none;
  flex: 1;
  overflow: auto;
  min-height: 0;
}

.result-tree-wrap {
  padding: 8px 0;
  background: #fff;
  border-radius: 4px;
}

.result-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 10px;
}

.result-header label {
  color: #555;
  font-weight: 500;
  margin: 0;
}

.result-actions {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.btn-copy {
  padding: 8px 16px;
  background: #f093fb;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: background 0.3s;
}

.btn-copy:hover {
  background: #e082ea;
}

.btn-fullscreen {
  padding: 8px 16px;
  background: #95a5a6;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: background 0.3s;
}

.btn-fullscreen:hover {
  background: #7f8c8d;
}

.result-pre {
  margin: 0;
  padding: 12px;
  background: #f8f9fa;
  border-radius: 4px;
  font-family: 'Courier New', monospace;
  font-size: 14px;
  line-height: 1.6;
  white-space: pre-wrap;
  word-break: break-word;
  color: #2c3e50;
  max-height: 500px;
  overflow-y: auto;
}

.save-history-row {
  margin-top: 16px;
}

.btn-save-history {
  padding: 8px 16px;
  background: #f093fb;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: background 0.3s;
}

.btn-save-history:hover {
  background: #e082ea;
}

.history-section {
  margin-top: 32px;
  padding: 20px;
  background: #f8f9fa;
  border-radius: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.history-title {
  margin: 0 0 4px 0;
  font-size: 1.1em;
  color: #2c3e50;
  border-bottom: 2px solid #f093fb;
  padding-bottom: 8px;
}

.history-hint {
  margin: 0 0 12px 0;
  font-size: 12px;
  color: #7f8c8d;
}

.history-list {
  list-style: none;
  margin: 0;
  padding: 0;
}

.history-item {
  background: white;
  border-radius: 6px;
  padding: 12px;
  margin-bottom: 10px;
  border-left: 4px solid #f093fb;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
}

.history-item:last-child {
  margin-bottom: 0;
}

.history-preview {
  margin: 0 0 10px 0;
  padding: 0;
  font-family: 'Courier New', monospace;
  font-size: 13px;
  line-height: 1.5;
  white-space: pre-wrap;
  word-break: break-word;
  color: #555;
  max-height: 60px;
  overflow: hidden;
}

.history-actions {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.btn-history {
  padding: 4px 10px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
  transition: opacity 0.2s;
}

.btn-history:hover {
  opacity: 0.9;
}

.btn-restore {
  background: #3498db;
  color: white;
}

.btn-copy-sm {
  background: #95a5a6;
  color: white;
}

.btn-remove {
  background: #e74c3c;
  color: white;
}
</style>
