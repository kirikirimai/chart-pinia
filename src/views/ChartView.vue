<template>
  <div class="chart">
    <!-- <button @click="updateData">データを更新</button>
    <button @click="addData">データを追加</button> -->
    <div class="selectWrap">
      <label for="selectStart">船名</label>
      <select v-model="selectedShip" id="selectedShip">
        <option v-for="(item, idx) in uniqueShip" :key="idx" :value="item">{{ item }}</option>
      </select>

      <label for="selectStart">出発地</label>
      <select v-model="selectedDepart" id="selectedDepart">
        <option v-for="(item, idx) in uniqueDepart" :key="idx" :value="item">{{ item }}</option>
      </select>

      <label for="selectStart">到着地</label>
      <select v-model="selectedArraival" id="selectedArraival">
        <option v-for="(item, idx) in uniqueArrival" :key="idx" :value="item">{{ item }}</option>
      </select>

      <label for="selectStart">Start</label>
      <select v-model="selectedStart" id="selectStart">
        <option v-for="(item,idx) in uniqueStart" :key="idx" :value="item">{{ item }}</option>
      </select>

      {{ selectedStart }}
      <label for="selectedEnd">End</label>
      <select v-model="selectedEnd" id="selectedEnd">
        <option v-for="(item,idx) in uniqueEnd" :key="idx" :value="item">{{ item }}</option>
      </select>

      {{ selectedEnd }}

      <button @click="filterData">検索</button>
    </div>

    <div class="canvasWrap">
      <canvas ref="chartRef"></canvas>
    </div>

  </div>
</template>
<script setup>
import { ref, onMounted, watch } from 'vue'
import Chart from 'chart.js/auto'

const dataRef = ref([])
const chartRef = ref(null)
let chartObj = null;

/* フォームのデータ */
const selectedStart = ref("")
const selectedEnd = ref("")
const selectedArraival = ref("")
const selectedDepart = ref("")
const selectedShip = ref("")

/* ユニークなデータ */
const uniqueDepart = ref([])
const uniqueArrival = ref([])
const uniqueShip = ref([])
const uniqueStart = ref([])
const uniqueEnd = ref([])

/* フィルタリングされたデータ */
const filteredData = ref([]);
const jsonData = ref([])

// const updateData = () => {
//   chartObj.data.datasets.forEach(element => {
//     element.data = element.data.map(() => Math.floor(Math.random() * 20));
//     chartObj.update()
//   });
// }

const filterData = () => {
  filteredData.value = jsonData.value.filter(item =>
    item.depart === selectedDepart.value &&
    item.arrival === selectedArraival.value &&
    item.start === selectedStart.value &&
    item.ship === selectedShip.value &&
    item.end === selectedEnd.value
  );

  addData();
};

const getRandomColor = () => {
  const r = Math.floor(Math.random() * 256);
  const g = Math.floor(Math.random() * 256);
  const b = Math.floor(Math.random() * 256);
  return `rgb(${r}, ${g}, ${b})`;
}


const createUniqueData = () => {
  uniqueShip.value = [...new Set(jsonData.value.map(item => item.ship))];
};

const updateuniqueDepartAndArrivals = () => {
  const filteredData = jsonData.value.filter(item => item.ship === selectedShip.value);

  uniqueDepart.value = [...new Set(filteredData.map(item => item.depart))];
  uniqueArrival.value = [...new Set(filteredData.map(item => item.arrival))];
  uniqueStart.value = [...new Set(filteredData.map(item => item.start))];
  uniqueEnd.value = [...new Set(filteredData.map(item => item.end))];
};

const fetchData = async () => {
  try {
    const response = await fetch("/jsonData.json");
    if (response.ok) {
      jsonData.value = await response.json();
    } else {
      console.error('Failed to load JSON data');
    }
  } catch (error) {
    console.error('An error occurred while loading JSON data:', error);
  }
}

const addData = () => {
  if (filteredData.value.length <= 0) {
    alert("no data");
    return false;
  }

  const newDataset = {
    label: `Dataset: ${selectedDepart.value} - ${selectedArraival.value}`,
    borderColor: getRandomColor(),
    data: filteredData.value[0].data
  };

  chartObj.data.datasets.push(newDataset);
  chartObj.options.plugins.legend.display = true;

  // チャートを更新
  chartObj.update();
};

watch(selectedShip, () => {
  updateuniqueDepartAndArrivals();
});


onMounted(async () => {

  await fetchData();
  await createUniqueData();
  const ctx = chartRef.value.getContext('2d')
  chartObj = new Chart(ctx, {
    type: 'line',
    data: {
      labels: ['1', '2', '3', '4', '5', '6'],
      datasets: [{
        label: "",
        data: dataRef.value,
        fill: false,
        borderColor: 'rgb(75, 192, 192)',
        tension: 0.1,
        pointStyle: "circle"
      }]
    },
    options: {
      responsive: true,
      plugins: {
        title: {
          display: true,
          text: "Chart.js Line Chart"
        },
        legend: {
          display: false, // 凡例を非表示
        }
      }
    }
  })
})

// Watch for changes in selectedOptionStart
// watch(selectedStart, (newVal, oldVal) => {
//   if (newVal && !oldVal) {
//     addData('Dataset - Start: ' + newVal);
//   }
// });

// // Watch for changes in selectedOptionEnd
// watch(selectedEnd, (newVal, oldVal) => {
//   if (newVal && !oldVal) {
//     console.log(newVal)
//     addData('Dataset - End: ' + newVal);
//   }
// });
</script>

<style>
@media (min-width: 1024px) {
  .chart {
    min-height: 100vh;
  }
}

.canvasWrap {
  width: 80%;
}

canvas {
  width: 100%;
  height: auto;
  max-height: 300px;
  margin: 0 auto;
  display: block;
}

.chart {
  display: flex;
}

.selectWrap {
  width: 20%;
}

.selectWrap * {
  display: block;
}
</style>
