<script setup lang="ts">
import { RouterView } from 'vue-router'
</script>

<template>
  <header class="app-header">
    <span class="logo-icon">🏋️</span>
    <h1 class="logo-text">GymTracker</h1>
  </header>
  <RouterView />
</template>

<style scoped>
.app-header {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  padding: 1.2rem 1.2rem 0.8rem;
  border-bottom: 1px solid var(--border);
}

.logo-icon {
  font-size: 1.4rem;
}

.logo-text {
  font-size: 1.2rem;
  font-weight: 700;
  color: var(--text);
  letter-spacing: -0.3px;
}
</style>