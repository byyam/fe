<script setup>
import { inject, computed } from 'vue'

const props = defineProps({
  value: { type: [Object, Array, String, Number, Boolean, null], default: null },
  path: { type: String, required: true },
  label: { type: String, default: '' }
})

const collapsedPathsRef = inject('collapsedPaths', { value: {} })
const toggleCollapse = inject('toggleCollapse', () => {})

const isObject = computed(() => props.value !== null && typeof props.value === 'object' && !Array.isArray(props.value))
const isArray = computed(() => Array.isArray(props.value))
const isCollapsible = computed(() => isObject.value || isArray.value)
const collapsedObj = computed(() => collapsedPathsRef?.value ?? collapsedPathsRef ?? {})
const isCollapsed = computed(() => !!collapsedObj.value[props.path])

const childEntries = computed(() => {
  if (isObject.value) {
    return Object.entries(props.value).map(([k, v]) => ({ key: k, value: v }))
  }
  if (isArray.value) {
    return props.value.map((v, i) => ({ key: String(i), value: v }))
  }
  return []
})

const childPath = (key) => (props.path === 'root' ? key : `${props.path}.${key}`)

const bracketOpen = computed(() => (isArray.value ? '[' : '{'))
const bracketClose = computed(() => (isArray.value ? ']' : '}'))
const size = computed(() => childEntries.value.length)

function formatPrimitive(v) {
  if (v === null) return 'null'
  if (typeof v === 'boolean') return v ? 'true' : 'false'
  if (typeof v === 'string') return JSON.stringify(v)
  return String(v)
}
</script>

<template>
  <div class="json-tree-node">
    <!-- 可折叠对象/数组 -->
    <template v-if="isCollapsible">
      <div class="tree-line" @click="toggleCollapse(path)">
        <span class="tree-toggle" :class="{ collapsed: isCollapsed }">{{ isCollapsed ? '▶' : '▼' }}</span>
        <span v-if="label !== ''" class="tree-key">{{ label }}:</span>
        <span class="tree-bracket">{{ bracketOpen }}</span>
        <span v-if="isCollapsed" class="tree-preview"> {{ size }} {{ isArray ? '项' : '键' }} </span>
        <span v-if="isCollapsed" class="tree-bracket">{{ bracketClose }}</span>
      </div>
      <div v-if="!isCollapsed" class="tree-children">
        <template v-for="(entry, i) in childEntries" :key="entry.key">
          <div class="tree-child">
            <JsonTree
              v-if="entry.value !== null && typeof entry.value === 'object'"
              :value="entry.value"
              :path="childPath(entry.key)"
              :label="isArray ? String(entry.key) : JSON.stringify(entry.key)"
            />
            <div v-else class="tree-line tree-primitive">
              <span v-if="!isArray" class="tree-key">{{ JSON.stringify(entry.key) }}:</span>
              <span v-else class="tree-index">{{ entry.key }}:</span>
              <span class="tree-value">{{ formatPrimitive(entry.value) }}</span>
            </div>
          </div>
        </template>
        <div class="tree-line tree-close">
          <span class="tree-bracket">{{ bracketClose }}</span>
        </div>
      </div>
    </template>
    <!-- 根为原始值（不应出现） -->
    <template v-else-if="path === 'root'">
      <span class="tree-value">{{ formatPrimitive(value) }}</span>
    </template>
  </div>
</template>

<style scoped>
.json-tree-node {
  font-family: 'Courier New', monospace;
  font-size: 13px;
  line-height: 1.5;
  color: #2c3e50;
}

.tree-line {
  display: flex;
  align-items: baseline;
  gap: 4px;
  padding: 1px 0;
  cursor: default;
  flex-wrap: wrap;
}

.tree-line:has(.tree-toggle) {
  cursor: pointer;
  user-select: none;
}

.tree-line:has(.tree-toggle):hover {
  background: rgba(0, 0, 0, 0.04);
}

.tree-toggle {
  display: inline-block;
  width: 16px;
  color: #7f8c8d;
  font-size: 10px;
}

.tree-toggle.collapsed {
  transform: rotate(-90deg);
}

.tree-key {
  color: #881391;
  margin-right: 4px;
}

.tree-index {
  color: #7f8c8d;
  margin-right: 4px;
}

.tree-value {
  color: #0e639c;
}

.tree-value.boolean {
  color: #0e639c;
}

.tree-bracket {
  color: #2c3e50;
  font-weight: 500;
}

.tree-preview {
  color: #7f8c8d;
  font-size: 12px;
}

.tree-children {
  padding-left: 20px;
  border-left: 1px solid #eee;
  margin-left: 8px;
}

.tree-child {
  margin: 0;
}

.tree-primitive {
  cursor: default;
}

.tree-close {
  margin-top: 0;
}
</style>
