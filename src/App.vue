<script setup lang="ts">
import ImageUpload from './components/imageUpload.vue'
</script>

<template>
  <div>
    <ImageUpload />
  </div>
</template>

<style>
/* 重置默认样式 */
body,
html {
  margin: 0;
  padding: 0;
  height: 100%;
  font-family: Arial, sans-serif;
}

/* 设置页面的布局 */
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  background-color: #f0f0f0;
  /* 背景色 */
}

/* 设置图片样式 */
.image {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  /* 确保图片不被拉伸 */
}
</style>
