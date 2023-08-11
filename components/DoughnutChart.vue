<template>
  <div class="container mt-5">
    <Doughnut v-if="loaded" :data="data" :options="options" />
  </div>
</template>
<script setup>
import { Chart as ChartJS, ArcElement, Tooltip, Legend } from "chart.js";
import { Doughnut } from "vue-chartjs";
import axios from "axios";
const labels = ref([]);
const dataApi = ref([]);
const loaded = ref(false);

ChartJS.register(ArcElement, Tooltip, Legend);

const data = ref({
  labels: labels,
  datasets: [
    {
      backgroundColor: ["#41B883", "#E46651"],
      data: dataApi,
    },
  ],
});
const options = ref({
  responsive: true,
  maintainAspectRatio: false,
});


function getCount() {
  loaded.value = false;
  axios
    .get("http://localhost:8000/api/count_customer_by_gender")
    .then((response) => {
      labels.value = response.data.data.labels;
      dataApi.value = response.data.data.count;
      loaded.value = true;
    });
}
onMounted(() => {
  getCount();
});
</script>
