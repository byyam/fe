<script setup>
import { ref, computed, watch } from 'vue'
import { marked } from 'marked'

const inputText = ref('')
const markdownHtml = ref('')

// 将字面量 \n（反斜杠+n）替换为真实换行
const convertedText = computed(() => {
  if (!inputText.value) return ''
  return inputText.value.replace(/\\n/g, '\n')
})

// Markdown 预览（基于第一个结果，异步解析）
watch(
  convertedText,
  async (text) => {
    if (!text) {
      markdownHtml.value = ''
      return
    }
    try {
      markdownHtml.value = await marked.parse(text)
    } catch {
      markdownHtml.value = ''
    }
  },
  { immediate: true }
)

// 复制到剪贴板，不弹窗
const copyToClipboard = (text) => {
  if (!text) return
  navigator.clipboard.writeText(text).catch(() => {})
}

// 一键清空输入
const clearInput = () => {
  inputText.value = ''
}

// 历史结果（仅内存，刷新即清空）
const MAX_HISTORY = 20
const historyList = ref([])
let historyId = 0

const saveToHistory = () => {
  if (!convertedText.value) return
  historyList.value.unshift({
    id: ++historyId,
    input: inputText.value,
    converted: convertedText.value
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

// 历史条目的预览文案（取前两行或前 80 字）
const historyPreview = (text) => {
  const lines = text.split('\n').slice(0, 2).join('\n')
  return lines.length > 80 ? lines.slice(0, 80) + '…' : lines
}

// Markdown 放大：点击后左侧消失，Markdown 占满整行
const markdownExpanded = ref(false)
const toggleMarkdownExpand = () => {
  markdownExpanded.value = !markdownExpanded.value
}
</script>

<template>
  <div class="container">
    <h1>Markdown Preview</h1>
    <p class="description">在输入框中粘贴含有字面量 <code>\n</code> 的文本，下方会实时显示换行结果与 Markdown 预览。</p>

    <div class="converter-section">
      <div class="input-group">
        <div class="input-label-row">
          <label>输入文本（可包含 \n）：</label>
          <button type="button" @click="clearInput" class="btn-clear" :disabled="!inputText">一键清空</button>
        </div>
        <textarea
          v-model="inputText"
          class="textarea-input"
          placeholder="例如：- Ask: function 测试\n- Display [opt1](button://...)"
          rows="8"
        />
      </div>

      <template v-if="convertedText">
        <div class="result-row" :class="{ 'markdown-expanded': markdownExpanded }">
          <!-- 左：仅换行结果（放大时隐藏） -->
          <div v-show="!markdownExpanded" class="result-section">
            <div class="result-header">
              <label>转换结果（仅换行）</label>
              <button @click="copyToClipboard(convertedText)" class="btn-copy">复制</button>
            </div>
            <pre class="result-pre">{{ convertedText }}</pre>
          </div>
          <!-- 右：Markdown 预览 -->
          <div class="result-section result-section-markdown">
            <div class="result-header">
              <label>Markdown 预览</label>
              <button type="button" @click="toggleMarkdownExpand" class="btn-expand">
                {{ markdownExpanded ? '收起' : '放大' }}
              </button>
            </div>
            <div class="markdown-preview" v-html="markdownHtml" />
          </div>
        </div>
      </template>

      <!-- 保存到历史 -->
      <div v-if="convertedText" class="save-history-row">
        <button type="button" @click="saveToHistory" class="btn-save-history">保存到历史</button>
      </div>
    </div>

    <!-- 历史结果 -->
    <div v-if="historyList.length" class="history-section">
      <h2 class="history-title">历史结果</h2>
      <p class="history-hint">不刷新页面前会保留，最多 {{ MAX_HISTORY }} 条</p>
      <ul class="history-list">
        <li v-for="item in historyList" :key="item.id" class="history-item">
          <pre class="history-preview">{{ historyPreview(item.converted) }}</pre>
          <div class="history-actions">
            <button type="button" @click="restoreFromHistory(item)" class="btn-history btn-restore">恢复</button>
            <button type="button" @click="copyToClipboard(item.converted)" class="btn-history btn-copy-sm">复制</button>
            <button type="button" @click="removeFromHistory(item.id)" class="btn-history btn-remove">删除</button>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.container {
  max-width: 1200px;
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

.description code {
  background: #ecf0f1;
  padding: 2px 6px;
  border-radius: 4px;
  font-family: 'Courier New', monospace;
}

.converter-section {
  background: #f8f9fa;
  padding: 25px;
  border-radius: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.input-group {
  margin-bottom: 20px;
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
  border-color: #3498db;
}

.result-row {
  display: flex;
  gap: 16px;
  margin-top: 20px;
  min-height: 0;
}

.result-row .result-section {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
}

.result-row.markdown-expanded .result-section-markdown {
  flex: 1 1 100%;
}

.result-section {
  margin-top: 0;
  padding: 15px;
  background: white;
  border-radius: 6px;
  border-left: 4px solid #3498db;
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
  flex: 1;
  min-height: 200px;
  max-height: 400px;
  overflow-y: auto;
}

.result-section-markdown {
  border-left-color: #27ae60;
}

.result-section-markdown .result-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 8px;
}

.btn-expand {
  padding: 6px 12px;
  background: #27ae60;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 13px;
  transition: background 0.3s;
}

.btn-expand:hover {
  background: #219a52;
}

.result-section-markdown .markdown-preview {
  flex: 1;
  min-height: 200px;
  max-height: 400px;
}

.markdown-preview {
  margin: 0;
  padding: 12px;
  background: #fff;
  border-radius: 4px;
  font-size: 14px;
  line-height: 1.6;
  color: #2c3e50;
  overflow-y: auto;
}

.markdown-preview :deep(ul) {
  margin: 0.5em 0;
  padding-left: 1.5em;
}

.markdown-preview :deep(li) {
  margin: 0.25em 0;
}

.markdown-preview :deep(strong) {
  font-weight: 600;
  color: #1a1a1a;
}

.markdown-preview :deep(a) {
  color: #3498db;
  text-decoration: none;
}

.markdown-preview :deep(a:hover) {
  text-decoration: underline;
}

.markdown-preview :deep(p) {
  margin: 0.5em 0;
}

.markdown-preview :deep(p:first-child) {
  margin-top: 0;
}

.save-history-row {
  margin-top: 16px;
}

.btn-save-history {
  padding: 8px 16px;
  background: #9b59b6;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: background 0.3s;
}

.btn-save-history:hover {
  background: #8e44ad;
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
  border-bottom: 2px solid #9b59b6;
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
  border-left: 4px solid #9b59b6;
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

@media (max-width: 768px) {
  .result-row {
    flex-direction: column;
  }
}
</style>
