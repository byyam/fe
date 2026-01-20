<script setup>
const props = defineProps({
  toolName: {
    type: String,
    required: true
  }
})
</script>

<template>
  <div class="placeholder-container">
    <div class="placeholder-content">
      <div class="icon">🚧</div>
      <h1>{{ toolName }}</h1>
      <p>该工具正在开发中，敬请期待...</p>
    </div>
  </div>
</template>

<style scoped>
.placeholder-container {
  min-height: 60vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 40px 20px;
}

.placeholder-content {
  text-align: center;
  max-width: 500px;
}

.icon {
  font-size: 5em;
  margin-bottom: 20px;
}

h1 {
  color: #2c3e50;
  font-size: 2em;
  margin: 0 0 15px 0;
}

p {
  color: #7f8c8d;
  font-size: 1.2em;
  margin: 0;
}
</style>
