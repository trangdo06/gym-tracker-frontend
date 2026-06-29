<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useAuth0 } from '@auth0/auth0-vue'

const BASE = import.meta.env.VITE_BACKEND_BASE_URL
const { getAccessTokenSilently } = useAuth0()

async function authFetch(url: string, options: RequestInit = {}) {
  const token = await getAccessTokenSilently()
  return fetch(url, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`,
      ...options.headers,
    },
  })
}

const workouts = ref<any[]>([])
const activeWorkout = ref<any>(null)
const newWorkoutName = ref('')

const searchQuery = ref('')
const nameQuery = ref('')
const searchResults = ref<any[]>([])
const selectedExercise = ref<any>(null)
const sets = ref(3)
const reps = ref(10)

const muscleGroups = ['chest', 'lats', 'middle_back', 'lower_back', 'traps', 'shoulders', 'biceps', 'triceps', 'quadriceps', 'hamstrings', 'glutes', 'calves', 'abdominals']
const difficultyColor: Record<string, string> = { beginner: '#4caf50', intermediate: '#ff9800', expert: '#f44336' }

async function loadWorkouts() {
  const res = await authFetch(BASE + '/workout')
  workouts.value = await res.json()
}

async function createWorkout() {
  if (!newWorkoutName.value.trim()) return
  const res = await authFetch(BASE + '/workout', {
    method: 'POST',
    body: JSON.stringify({ workoutName: newWorkoutName.value.trim() })
  })
  const workout = await res.json()
  workouts.value.unshift(workout)
  activeWorkout.value = workout
  newWorkoutName.value = ''
}

async function searchByMuscle(muscle: string) {
  searchQuery.value = muscle
  nameQuery.value = ''
  const res = await authFetch(`${BASE}/exercises/search?muscle=${encodeURIComponent(muscle)}`)
  searchResults.value = await res.json()
  selectedExercise.value = null
}

async function searchByName() {
  const name = nameQuery.value.trim()
  if (!name) return
  searchQuery.value = ''
  const res = await authFetch(`${BASE}/exercises/search?name=${encodeURIComponent(name)}`)
  searchResults.value = await res.json()
  selectedExercise.value = null
}

async function addExercise() {
  if (!activeWorkout.value || !selectedExercise.value) return
  const res = await authFetch(`${BASE}/workout/${activeWorkout.value.id}/exercises`, {
    method: 'POST',
    body: JSON.stringify({
      exerciseName: selectedExercise.value.name,
      muscle: selectedExercise.value.muscle,
      sets: sets.value,
      reps: reps.value
    })
  })
  const updated = await res.json()
  activeWorkout.value = updated
  workouts.value = workouts.value.map(w => w.id === updated.id ? updated : w)
  selectedExercise.value = null
  searchResults.value = []
  searchQuery.value = ''
  sets.value = 3
  reps.value = 10
}

onMounted(loadWorkouts)
</script>

<template>
  <div class="page">

    <!-- Workout erstellen -->
    <section class="section">
      <h2 class="section-title">Neues Workout</h2>
      <div class="row">
        <input v-model="newWorkoutName" class="input" placeholder="z.B. Leg Day, Push, Pull..." @keyup.enter="createWorkout" />
        <button class="btn-primary" @click="createWorkout">Erstellen</button>
      </div>
    </section>

    <!-- Workout Liste -->
    <section class="section" v-if="workouts.length">
      <h2 class="section-title">Meine Workouts</h2>
      <div class="workout-list">
        <div
          v-for="w in workouts"
          :key="w.id"
          class="workout-card"
          :class="{ 'workout-card--active': activeWorkout?.id === w.id }"
          @click="activeWorkout = w; searchResults = []; selectedExercise = null"
        >
          <div class="workout-card__left">
            <div class="workout-icon">💪</div>
            <div>
              <p class="workout-name">{{ w.workoutName }}</p>
              <p class="workout-meta">{{ w.exercises.length }} Übungen</p>
            </div>
          </div>
          <span class="chevron">›</span>
        </div>
      </div>
    </section>

    <!-- Aktives Workout: Übungen + Suche -->
    <section class="section" v-if="activeWorkout">
      <div class="section-header">
        <h2 class="section-title">{{ activeWorkout.workoutName }}</h2>
        <span class="tag--orange">Aktiv</span>
      </div>

      <!-- Bereits hinzugefügte Übungen -->
      <div v-if="activeWorkout.exercises.length" class="exercise-list">
        <div v-for="ex in activeWorkout.exercises" :key="ex.id" class="exercise-entry">
          <div class="entry-bar" />
          <div class="entry-info">
            <p class="exercise-name">{{ ex.exerciseName }}</p>
            <p class="exercise-meta">{{ ex.muscle }}</p>
          </div>
          <div class="entry-stats">
            <span class="stat-chip">{{ ex.sets }} Sets</span>
            <span class="stat-chip">{{ ex.reps }} Reps</span>
          </div>
        </div>
      </div>
      <p class="empty-hint" v-else>Noch keine Übungen</p>

      <!-- Muskelgruppen-Chips -->
      <div class="search-box">
        <p class="search-label">Übung suchen</p>
        <div class="row">
          <input v-model="nameQuery" class="input" placeholder="Name eingeben..." @keyup.enter="searchByName" />
          <button class="btn-primary" @click="searchByName">Suchen</button>
        </div>
        <div class="muscle-chips">
          <button
            v-for="m in muscleGroups"
            :key="m"
            class="chip"
            :class="{ 'chip--active': searchQuery === m }"
            @click="searchByMuscle(m)"
          >{{ m }}</button>
        </div>

        <!-- Suchergebnisse -->
        <div v-if="searchResults.length" class="results-list">
          <div
            v-for="ex in searchResults"
            :key="ex.name"
            class="result-card"
            :class="{ 'result-card--selected': selectedExercise?.name === ex.name }"
            @click="selectedExercise = ex"
          >
            <div class="result-top">
              <span class="result-name">{{ ex.name }}</span>
              <span class="diff-dot" :style="{ background: difficultyColor[ex.difficulty] ?? '#888' }" />
            </div>
            <div class="result-tags">
              <span class="tag">{{ ex.muscle }}</span>
              <span class="tag">{{ ex.equipment }}</span>
            </div>
          </div>
        </div>

        <!-- Sets & Reps + Hinzufügen -->
        <div v-if="selectedExercise" class="add-panel">
          <p class="add-panel__title">{{ selectedExercise.name }}</p>
          <div class="counter-row">
            <div class="counter">
              <button class="counter-btn" @click="sets = Math.max(1, sets - 1)">−</button>
              <div class="counter-val">
                <span class="counter-num">{{ sets }}</span>
                <span class="counter-label">Sets</span>
              </div>
              <button class="counter-btn" @click="sets++">＋</button>
            </div>
            <div class="counter-divider" />
            <div class="counter">
              <button class="counter-btn" @click="reps = Math.max(1, reps - 1)">−</button>
              <div class="counter-val">
                <span class="counter-num">{{ reps }}</span>
                <span class="counter-label">Reps</span>
              </div>
              <button class="counter-btn" @click="reps++">＋</button>
            </div>
          </div>
          <button class="btn-accent" @click="addExercise">+ Hinzufügen</button>
        </div>
      </div>
    </section>

  </div>
</template>

<style scoped>
.page { padding: 1rem 1rem 3rem; display: flex; flex-direction: column; gap: 1.5rem; }

.section { display: flex; flex-direction: column; gap: 0.75rem; }
.section-header { display: flex; justify-content: space-between; align-items: center; }
.section-title { font-size: 1rem; font-weight: 700; }

.row { display: flex; gap: 0.5rem; }
.input {
  flex: 1; background: var(--bg-input); border: 1px solid var(--border);
  border-radius: var(--radius-sm); color: var(--text); padding: 0.55rem 0.8rem;
  font-size: 0.9rem; outline: none; min-width: 0;
}
.input::placeholder { color: var(--text-sub); }
.input:focus { border-color: var(--accent); }

.btn-primary {
  padding: 0.55rem 1rem; border: none; border-radius: var(--radius-sm);
  background: var(--accent); color: #fff; font-size: 0.9rem; font-weight: 600; cursor: pointer;
}
.btn-primary:hover { background: var(--accent-hover); }

.btn-accent {
  width: 100%; padding: 0.85rem; border: none; border-radius: var(--radius-sm);
  background: var(--accent); color: #fff; font-size: 0.95rem; font-weight: 700; cursor: pointer;
}
.btn-accent:hover { background: var(--accent-hover); }

.workout-list { display: flex; flex-direction: column; gap: 0.5rem; }
.workout-card {
  display: flex; justify-content: space-between; align-items: center;
  background: var(--bg-card); border-radius: var(--radius); padding: 0.9rem 1rem;
  cursor: pointer; border: 1.5px solid transparent; transition: border-color 0.15s;
}
.workout-card:hover { border-color: var(--border); }
.workout-card--active { border-color: var(--accent); background: var(--accent-soft); }
.workout-card__left { display: flex; align-items: center; gap: 0.75rem; }
.workout-icon { font-size: 1.3rem; }
.workout-name { font-weight: 600; font-size: 0.95rem; }
.workout-meta { font-size: 0.78rem; color: var(--text-sub); margin-top: 0.1rem; }
.chevron { font-size: 1.3rem; color: var(--text-sub); }

.exercise-list { display: flex; flex-direction: column; gap: 0.5rem; }
.exercise-entry {
  display: flex; align-items: center; gap: 0.75rem;
  background: var(--bg-card); border-radius: var(--radius-sm); padding: 0.75rem 0.9rem;
}
.entry-bar { width: 3px; height: 2rem; border-radius: 2px; background: var(--accent); flex-shrink: 0; }
.entry-info { flex: 1; min-width: 0; }
.exercise-name { font-weight: 600; font-size: 0.9rem; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.exercise-meta { font-size: 0.75rem; color: var(--text-sub); margin-top: 0.1rem; }
.entry-stats { display: flex; gap: 0.4rem; }
.stat-chip { font-size: 0.75rem; background: var(--bg-input); padding: 0.2rem 0.5rem; border-radius: 20px; color: var(--text-sub); }

.search-box {
  background: var(--bg-card); border-radius: var(--radius); padding: 1rem;
  display: flex; flex-direction: column; gap: 0.75rem;
}
.search-label { font-size: 0.78rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.8px; color: var(--text-sub); }
.muscle-chips { display: flex; flex-wrap: wrap; gap: 0.4rem; }
.chip {
  padding: 0.3rem 0.7rem; border-radius: 20px; border: 1px solid var(--border);
  background: var(--bg-input); color: var(--text-sub); font-size: 0.78rem;
  cursor: pointer; text-transform: capitalize; transition: all 0.15s;
}
.chip:hover { border-color: var(--accent); color: var(--accent); }
.chip--active { border-color: var(--accent); background: var(--accent-soft); color: var(--accent); font-weight: 600; }

.results-list {
  display: flex; flex-direction: column; gap: 0.4rem;
  max-height: 220px; overflow-y: auto;
}
.result-card {
  background: var(--bg-input); border-radius: var(--radius-sm); padding: 0.65rem 0.8rem;
  cursor: pointer; border: 1.5px solid transparent; transition: border-color 0.12s;
}
.result-card:hover { border-color: var(--border); }
.result-card--selected { border-color: var(--accent); background: var(--accent-soft); }
.result-top { display: flex; justify-content: space-between; align-items: center; gap: 0.5rem; }
.result-name { font-weight: 600; font-size: 0.88rem; }
.diff-dot { width: 8px; height: 8px; border-radius: 50%; flex-shrink: 0; }
.result-tags { display: flex; gap: 0.35rem; margin-top: 0.35rem; flex-wrap: wrap; }
.tag { font-size: 0.72rem; padding: 0.15rem 0.5rem; border-radius: 20px; background: var(--bg-hover); color: var(--text-sub); text-transform: capitalize; }
.tag--orange { background: var(--accent-soft); color: var(--accent); font-size: 0.72rem; padding: 0.15rem 0.5rem; border-radius: 20px; font-weight: 600; }

.add-panel {
  background: var(--bg-input); border-radius: var(--radius-sm); padding: 1rem;
  display: flex; flex-direction: column; gap: 1rem; border: 1.5px solid var(--accent);
}
.add-panel__title { font-weight: 700; font-size: 0.95rem; }
.counter-row { display: flex; align-items: center; justify-content: center; gap: 2rem; }
.counter { display: flex; align-items: center; gap: 1rem; }
.counter-btn {
  width: 36px; height: 36px; border-radius: 50%; border: 1.5px solid var(--border);
  background: var(--bg-hover); color: var(--text); font-size: 1.1rem; cursor: pointer;
  display: flex; align-items: center; justify-content: center;
}
.counter-btn:hover { border-color: var(--accent); color: var(--accent); }
.counter-val { text-align: center; }
.counter-num { display: block; font-size: 1.6rem; font-weight: 700; line-height: 1; }
.counter-label { display: block; font-size: 0.72rem; color: var(--text-sub); text-transform: uppercase; letter-spacing: 0.5px; }
.counter-divider { width: 1px; height: 3rem; background: var(--border); }

.empty-hint { font-size: 0.82rem; color: var(--text-sub); }
</style>
