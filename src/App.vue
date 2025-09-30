<script setup lang="ts">
import { RouterView } from 'vue-router'
import HomeView from './views/HomeView.vue'
</script>

<template>
  <div class="app-container">
    <HomeView></HomeView>
  </div>
</template>

<style scoped>
.app-container {
  width: 100vw;
  height: 100vh;
  padding: 0;
  margin: 0;
  overflow: hidden;
}

.app-container :deep(*) {
  box-sizing: border-box;
}

.app-container :deep(#app) {
  width: 100%;
  height: 100%;
  padding: 0;
  margin: 0;
}
</style>