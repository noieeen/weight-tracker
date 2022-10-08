<script  setup>
import { ref, shallowRef, computed, watch, nextTick } from "vue";
import Chart from "chart.js/auto";

// interface WeightData{
//   weight:number;
//   date:Date;
// }

const weights = ref([]);

const weightChartEl = ref(null);

const weightChart = shallowRef(null);

const weightInput = ref(60.0);

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
        .map((w) => new Date(w.date).toLocaleDateString("en-US"))
        .slice(-7);

      weightChart.value.data.datasets[0].data = ws
        .sort((a, b) => a.date - b.date)
        .map((w) => w.weight)
        .slice(-7);

      weightChart.value.update();

      return;
    }

    nextTick(() => {
      weightChart.value = new Chart(weightChartEl.value.getContext("2d"), {
        type: "line",
        data: {
          labels: ws
            .sort((a, b) => a.date - b.date)
            .map((w) => new Date(w.date).toLocaleDateString("en-US")),
          datasets: [
            {
              label: "Weight",
              data: ws.sort((a, b) => a.date - b.date).map((w) => w.weight),
              backgroundColor: "rgba(255,105,180,0.2)",
              borderColor: "rgba(255,105,180)",
              borderWidth: 1,
              fill: true,
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
      <small> Current weight (kg)</small>
    </div>

    <form @submit.prevent="addWeight">
      <input type="number" step="0.1" v-model="weightInput" />
      <input type="submit" value="Add Weight" />
    </form>

    <div v-if="weights && weights.length">
      <h2>Last 7 days</h2>

      <div class="canvas-box">
        <canvas ref="weightChartEl"></canvas>
      </div>

      <div class="weight-history">
        <h2>Weight history</h2>
        <ul>
          <li v-for="weight in weights" v-bind:key="weight.date">
            <span>{{ weight.weight }} kg</span>
            <small>{{
              new Date(weight.date).toLocaleDateString("en-US")
            }}</small>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>

<style scoped>

*{
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family:'montserrat', sans-serif;
}

.current{
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

  width: 200px;
  height: 200px;

  text-align: center;
  border-radius:50%;
  box-shadow: 0 4px 12px rgba(255,255,255,0.1);
  border:5px solid hwb(330 41% 0%);
  background-color: #313131;
  margin: 2rem auto 1rem;
}

.current > span{
  font-size: 3rem;
  font-weight: bold;
  color:  hwb(330 41% 0%);
  margin-bottom: 1rem;
}

.weight-history{
  margin-top: 1rem;
}
.weight-history ul{
  margin-top: .5rem;
  height: 100px;
  overflow-y:auto ;
}

.weight-history li{
  display: flex;
  justify-content:space-between;
  align-items:center;
}

.weight-history li >span{
  font-weight: bold;
}

</style>
