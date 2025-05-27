<script setup>
import {ref, shallowRef, computed, watch, nextTick } from 'vue'
import Chart from 'chart.js/auto'

const weights =ref([])


const weightChartE1 = ref(null)

const weightChart = shallowRef(null)

const weightInput = ref(60.0)

const currentWeight = computed(() => {
  return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 }
})

const addWeight = () => {
    weights.value.push({
        weight: weightInput.value,
        date: new Date().getTime()
    })
}

</script>

<template>
  <main>

    <h1>Weight Tracker</h1>

    <div class="current">
        <span>{{ currentWeight.weight }}</span>
        <small>Current weight (kg)</small>
    </div>

    <form @submit.prevent="addWeight">
        <input type="number" step="0.1"  v-model="weightInput" placeholder="Enter weight (kg)" />
        <input type="submit" value="Add Weight" />


    </form>


  </main>
</template>

<style scoped>

</style>
