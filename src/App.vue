<script setup>
import { ref, shallowRef, computed, watch, nextTick } from "vue";
import Chart from "chart.js/auto";

const weights = ref([]);

const weightChartE1 = ref(null);

const weightChart = shallowRef(null);

const weightInput = ref(null);

const currentWeight = computed(() => {
  return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 };
});

const addWeight = () => {
  weights.value.push({
    weight: weightInput.value,
    date: new Date().getTime(),
  });
};

watch(
  weights,
  (newWeights) => {
    const ws = [...newWeights];

    if (weightChart.value) {
      weightChart.value.data.labels = ws
        .sort((a, b) => a.date - b.date)
        .map((w) => new Date(w.date).toLocaleDateString())
        .slice(-7);

      weightChart.value.data.datasets[0].data = ws
        .sort((a, b) => a.date - b.date)
        .map((w) => w.weight)
        .slice(-7);

      weightChart.value.update();

      return;
    }

    nextTick(() => {
      weightChart.value = new Chart(weightChartE1.value.getContext("2d"), {
        type: "line",
        data: {
          labels: ws
            .sort((a, b) => a.date - b.date)
            .map((w) => new Date(w.date).toLocaleDateString()),
          datasets: [
            {
              label: "Weight (kg)",
              data: ws.sort((a, b) => a.date - b.date).map((w) => w.weight),
              borderColor: "rgba(75, 192, 192, 1)",
              backgroundColor: "rgba(75, 192, 192, 0.2)",
              borderWidth: 1,
              pointRadius: 3,
              pointBackgroundColor: "rgba(75, 192, 192, 1)",
              fill: true,
              tension: 0.1,
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
        },
      });
    });
  },
  { deep: true }
);
</script>

<template>
  <main>
    <h1>Weight Tracker</h1>

    <div class="current">
      <span>{{ currentWeight.weight }}</span>
      <small>Current weight (kg)</small>
    </div>

    <form @submit.prevent="addWeight">
      <input type="number" step="0.1" v-model="weightInput" placeholder="Enter weight (kg)"/>
      <input type="submit" value="Add Weight" />
    </form>
    
    <div class="history" v-if="weights && weights.length > 0">
      <h2>Last 7 Days</h2>

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
  </main>
</template>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: "montserrat", sans-serif;
}

body {
  background-color: #f4f4f4;
}
main {
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  align-items: center;
}

h1 {
  text-align: center;
  font-size: 2rem;
  margin-bottom: 2rem;
}

h2 {
  margin-bottom: 1rem;
  text-align: center;
  font-weight: 400;
}

.current {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

  width: 200px;
  height: 200px;

  text-align: center;
  background: white;
  border-radius: 100px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  border: 3px solid rgb(163, 255, 247);

  margin: 0 auto 2rem;
}

.current span {
  display: block;
  font-size: 2rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
}
.current small {
  font-style: italic;
  color: #666;
}

form {
  max-width: 720px;
  width: 100%;
  display: flex;
  flex-wrap: wrap; 
  margin-bottom: 2rem;
  border: 1px solid #aaa;
  border-radius: 0.5rem;
  overflow: hidden;
  transition: 200ms linear;
}

form:focus-within,
form:hover {
  border-color: #00bcd4;
  border-width: 2px;
}

form input[type="number"] {
  appearance: none;
  outline: none;
  border: none;
  background: #fff;

  flex: 1 1 auto;
  min-width: 200px;
  padding: 1rem 1.5rem;
  font-size: 1.25rem;
}

form input[type="submit"] {
  appearance: none;
  outline: none;
  border: none;
  background: #00bcd4;

  padding: 1rem 1.5rem;
  color: #fff;
  font-size: 1.25rem;
  font-weight: bold;
  cursor: pointer;
  transition: 200ms linear;
  border-left: 3px solid #00bcd4;
  flex-shrink: 0;
}

form input[type="submit"]:hover {
  background: #fff;
  color: #00bcd4;
  border-left-color: #00bcd4;
}

@media (max-width: 600px) {
    main {
        padding: 0.5rem;
    }
  form {
    flex-direction: row;
    gap: 0;
  }

  /* form input[type="number"],
  form input[type="submit"] {
    width: 100%;
    border-left: none;
    border-top: 1px solid #00bcd4;
  } */
  form input[type="number"] {
    width: 100%;
    border-left: none;
    border-top: none;
    align-items: center;
    text-align: center;
    font-size: 0.875rem;
    padding: 1rem;
  }

  form input[type="submit"] {
    width: 100%;
    border-top: none;
    border-radius: 0;
    font-size: 0.875rem;
  }
}

.history {
    width: 100%;
    max-width: 720px;
}
.canvas-box {
  width: 100%;
  max-width: 720px;
  background-color: white;
  padding: 1rem;
  border-radius: 0.5rem;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  margin-bottom: 2rem;
}

.weight-history ul {
  list-style: none;
  padding: 0;
  margin: 0;
}
.weight-history ul li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem;
  cursor: pointer;
}

.weight-history ul li:nth-child(even) {
  background-color: #c2c2c2;
}
.weight-history ul li:nth-child(odd) {
  background-color: #e4fff3;
}
.weight-history ul li:hover {
  background-color: #f0f0f0;
}
.weight-history ul li:last-of-type {
  border-bottom: none;
}
.weight-history ul li span {
  display: block;
  font-size: 1.25rem;
  font-weight: bold;
  margin-right: 1rem;
}

.weight-history ul li small {
  color: #666;
  font-style: italic;
}
</style>
