<script setup lang="ts">
import {ref} from 'vue'
type Workout = {
  name: string
  reps: number
  sets: number
}

const workouts = ref([])
const workoutNameField = ref('')
const nameField = ref('')
const repsField = ref(0)
const setField = ref(0)

function save() {
  workouts.value.push({
    name: nameField.value,
    reps: Number(repsField.value),
    sets: Number(setField.value)
  })
}
</script>

<template>
  <h1>Workout</h1>
  <h3>
    <input v-model="workoutNameField" placeholder="name of your workout" type="text">
  </h3>
  <div>
    <input v-model="nameField" placeholder="name of your exercise" type="text">
    <input v-model="repsField" placeholder="reps" type="text">
    <input v-model="setField" placeholder="sets" @keyup.enter="save()">
    <button @click="save()">Save</button>
  </div>
  <div v-for="workout in workouts" :key="workout.name">
    <h3>{{ workout.name }}</h3>
    <p>{{ workout.reps }} reps</p>
    <p>{{ workout.sets }} sets</p>
  </div>
</template>


<style scoped>

</style>