<script setup lang="ts">
import {ref} from 'vue'

type Workout = {
  workoutName: string
  name: string
  reps: number
  sets: number
}
const workouts = ref<Workout[]>([])
const workoutNameField = ref('')
const nameField = ref('')
const repsField = ref(0)
const setField = ref(0)

function save() {
  workouts.value.push({
    workoutName: workoutNameField.value,
    name: nameField.value,
    reps: Number(repsField.value),
    sets: Number(setField.value)
  })

  nameField.value = ''
  repsField.value = 0
  setField.value = 0

}

function loadWorkout() {
  const endpoint = 'http://localhost:8080/exercise'

  fetch(endpoint)
      .then(res => res.json())
      .then(data => {
        workouts.value = data
      })

}

onMounted(() => {
  loadWorkout()

})
</script>

<template>
  <h1>Workout</h1>
  <h3>
    <input v-model="workoutNameField" placeholder="name of your workout" type="text">
  </h3>
  <div>
    <input v-model="nameField" placeholder="name of your exercise" type="text">
    <input v-model.number="repsField" placeholder="reps" type="number">
    <input v-model.number="setField" placeholder="sets" type="number" @keyup.enter="save()">
    <button @click="save()">Save</button>
  </div>
  <div v-for="workout in workouts" :key="workout.name">
    <h3>{{ workout.workoutName }}</h3>
    <h3>{{ workout.name }}</h3>
    <p>{{ workout.reps }} reps</p>
    <p>{{ workout.sets }} sets</p>
  </div>
</template>


<style scoped>

</style>