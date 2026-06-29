<template>
  <div class="login-page">
    <div class="login-card">
      <span class="login-icon">🏋️</span>
      <h2 class="login-title">GymTracker</h2>
      <p class="login-sub">Melde dich an um deine Workouts zu verwalten</p>
      <button class="btn-login" @click="login">Anmelden</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useAuth0 } from '@auth0/auth0-vue'

const { loginWithRedirect } = useAuth0()

function login() {
  loginWithRedirect({ appState: { target: '/' } })
}
</script>

<style scoped>
.login-page {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: calc(100vh - 60px);
}

.login-card {
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 2.5rem 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  width: 100%;
  max-width: 360px;
}

.login-icon { font-size: 2.5rem; }

.login-title {
  font-size: 1.4rem;
  font-weight: 700;
}

.login-sub {
  font-size: 0.88rem;
  color: var(--text-sub);
  text-align: center;
}

.btn-login {
  margin-top: 0.5rem;
  width: 100%;
  padding: 0.75rem;
  border: none;
  border-radius: var(--radius-sm);
  background: var(--accent);
  color: #fff;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
}

.btn-login:hover { background: var(--accent-hover); }
</style>
