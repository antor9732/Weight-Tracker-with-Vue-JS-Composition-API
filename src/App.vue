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

watch(weights, newWeights => {
    const ws = [...newWeights]

    if (weightChart.value) {
        weightChart.value.data.labels = ws
            .sort((a, b) => a.date - b.date)
            .map(w => new Date(w.date).toLocaleDateString())
            .slice(-7)

        weightChart.value.data.datasets[0].data = ws
            .sort((a, b) => a.date - b.date)
            .map(w => w.weight)
            .slice(-7)

        weightChart.value.update()

        return
    }


    nextTick(() => {
        weightChart.value =new Chart(weightChartE1.value.getContext('2d'),{
            type: 'line',
            data: {
                labels: ws
                    .sort((a,b) => a.date - b.date)
                    .map(w => new Date(w.date).toLocaleDateString()),
                datasets: [{
                    label: 'Weight (kg)',
                    data: ws
                    .sort((a,b) => a.date - b.date)
                    .map(w => w.weight),
                    borderColor: 'rgba(75, 192, 192, 1)',
                    backgroundColor: 'rgba(75, 192, 192, 0.2)',
                    borderWidth: 1,
                    pointRadius: 3,
                    pointBackgroundColor: 'rgba(75, 192, 192, 1)',
                    fill: true,
                    tension: 0.1

                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,


            },
        })
    })
}, { deep: true })

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

        <div v-if="weights && weights.length > 0" >
          
            <h2> Last 7 Days</h2>

            <div class="canvas-box">
                <canvas ref="weightChartE1"></canvas>
            </div>

            <div class="weight-history">
                <h2>Weight History</h2>

                <ul>
                    <li v-for="weight in weights" :key="weight.date">

                        <span>{{ weight.weight }}kg</span>
                        <small>
                            {{ new Date(weight.date).toLocaleDateString() }}
                        </small>
                    </li>
                </ul>
            </div>

        </div>


    </form>


  </main>
</template>

<style scoped>

</style>
