<script setup lang="ts">
import { RouterView } from 'vue-router'
import { useAuth0 } from '@auth0/auth0-vue'

const { isAuthenticated, logout, user } = useAuth0()

function handleLogout() {
  logout({ logoutParams: { returnTo: window.location.origin + '/login' } })
}
</script>

<template>
  <header class="app-header">
    <span class="logo-icon">🏋️</span>
    <h1 class="logo-text">GymTracker</h1>
    <div class="header-actions" v-if="isAuthenticated">
      <span class="user-name">{{ user?.name }}</span>
      <button class="btn-logout" @click="handleLogout">Logout</button>
    </div>
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

.logo-icon { font-size: 1.4rem; }

.logo-text {
  font-size: 1.2rem;
  font-weight: 700;
  color: var(--text);
  letter-spacing: -0.3px;
}

.header-actions {
  margin-left: auto;
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.user-name {
  font-size: 0.85rem;
  color: var(--text-sub);
}

.btn-logout {
  padding: 0.4rem 0.9rem;
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  background: transparent;
  color: var(--text-sub);
  font-size: 0.85rem;
  cursor: pointer;
}

.btn-logout:hover {
  border-color: var(--accent);
  color: var(--accent);
}
</style>
