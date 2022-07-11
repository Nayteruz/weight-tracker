<template>
  <main>
    <h1>Отслеживание веса</h1>
    <div class="current">
      <span>{{ currentWeight.weight }}</span>
      <small>Текущий вес (кг)</small>
    </div>
    <form @submit.prevent="addWeight">
      <input type="number" step="0.1" v-model="weightInput">
      <button type="submit">Добавить вес</button>
    </form>
    <div v-if="weights && weights.length > 0">
      <h2>За последние 7 дней</h2>
      <div class="canvas-box">
        <canvas ref="weightChartEl"></canvas>
      </div>
      <div class="weight-history">
        <h2>История изменений веса</h2>
        <ul>
          <li v-for="weight in weights">
            <span>{{ weight.weight }}кг </span>
            <small>{{ new Date(weight.date).toLocaleDateString('ru-RU') }}</small>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>

<script setup>
import {ref, shallowRef, computed, watch, nextTick, onMounted} from 'vue'
import Chart from 'chart.js/auto'

const weights = ref([]);

const weightChartEl = ref(null);

const weightChart = shallowRef(null);

const weightInput = ref(100);

const currentWeight = computed(() => {
  return weights.value.sort((a, b) => b.date - a.date)[0] || {weight: 0}
})

const addWeight = () => {
  weights.value.push({
    weight: weightInput.value,
    date: new Date().getTime(),
  })
  localStorage.setItem('weight-list', JSON.stringify(weights.value));
}

onMounted(()=> {
  weights.value = JSON.parse(localStorage.getItem('weight-list')) || [];
})

watch(weights, newWeights => {
  const ws = [...newWeights]

  if (weightChart.value) {
    weightChart.value.data.labels = ws
        .sort((a, b) => a.date - b.date)
        .map(w => new Date(w.date).toLocaleDateString('ru-RU'))
        .slice(-7)

    weightChart.value.data.datasets[0].data = ws
        .sort((a, b) => a.date - b.date)
        .map(w => w.weight)
        .slice(-7)

    weightChart.value.update();

    return;
  }

  nextTick(() => {
    weightChart.value = new Chart(weightChartEl.value.getContext('2d'), {
      type: 'line',
      data: {
        labels: ws
            .sort((a, b) => a.date - b.date)
            .map(w => new Date(w.date).toLocaleDateString('ru-RU')),
        datasets: [
          {
            label: 'Вес',
            data: ws
                .sort((a, b) => a.date - b.date)
                .map(w => w.weight),
            backgroundColor: 'rgba(255,105,180,0.2)',
            borderColor: 'rgba(255,105,180,1)',
            borderWidth: 1,
            fill: true,
          }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false
      }
    })
  })
}, {deep: true})
</script>

