<script setup lang="ts">
import {ref, onMounted} from 'vue'

type Workout = {
  id:number
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


async function loadWorkout() {
  const endpoint = import.meta.env.VITE_BACKEND_BASE_URL + '/workout'
  const requestOptions = {
    method: 'GET'
  }

  fetch(endpoint, requestOptions)
      .then(res => res.json())
      .then(data => {
        workouts.value = data
      })
      .catch(error => console.log(error))
}


async function save() {
  const endpoint = import.meta.env.VITE_BACKEND_BASE_URL + '/workout'
  const data: Workout =Omit<Workout, 'id'> =  {
    workoutName: workoutNameField.value,
    name: nameField.value,
    reps: Number(repsField.value),
    sets: Number(setField.value)
  }
  try {
    const response = await fetch(endpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(data)
    })

    if (!response.ok) {
      throw new Error('Failed to save workout')
    }

    const savedWorkout = await response.json()

    workouts.value.push(savedWorkout)

    workoutNameField.value = ''
    nameField.value = ''
    repsField.value = 0
    setField.value = 0

  } catch (error) {
    console.error('Error:', error)
  }
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
  <div v-for="workout in workouts" :key="workout.id">
    <h3>{{workout.id}}</h3>
    <h3>{{ workout.workoutName }}</h3>
    <h3>{{ workout.name }}</h3>
    <p>{{ workout.reps }} reps</p>
    <p>{{ workout.sets }} sets</p>
  </div>
</template>


<style scoped>

</style>