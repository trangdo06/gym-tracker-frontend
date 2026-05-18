<script setup lang="ts">

</script>

<template>
  <h1>Workout</h1>
  <h3>    <input v-model="workoutNameField" placeholder="name of your workout" type="text"></h3>
  <div>
    <input v-model="nameField" placeholder="name of your exercise" type="text">
    <input v-model="repsField" placeholder="reps" type="text">
    <input v-model="setField" placeholder="sets" @keyup.enter="save()">
    <button type="button" @click="save()">Save</button>
  </div>
  <div>
  </div>
</template>

const workoutNameField = ref(string[])
const nameField = ref('')
const repsField = ref(0)
const setField = ref(0)

<style scoped>

</style>