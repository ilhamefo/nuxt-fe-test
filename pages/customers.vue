<script setup>
import axios from "axios";
import { Bar } from "vue-chartjs";
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale,
} from "chart.js";
import { ref } from "vue";
ChartJS.register(
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale
);

const { $swal } = useNuxtApp();

const customersData = ref(null);
const config = useRuntimeConfig();
const baseUrl = config.public.API_BASE_URL;
const pdfSection = ref(null);

const labels = ref([]);
const data = ref([]);
const loaded = ref(false);


const chartData = ref({
  labels: labels,
  datasets: [
    {
      label: "Data Berdasarkan Kota",
      backgroundColor: "#0d6efd",
      data: data,
    },
  ],
});

const chartOptions = ref({
  responsive: true,
  maintainAspectRatio: false,
});

function getCustomer(url) {
  axios
    .get(url)
    .then((response) => {
      customersData.value = response;
    })
    .catch((error) => {
      formErrorsMessages.value = error.response.data.message;
    })
    .finally();
}

function getCount() {
  loaded.value = false;
  axios.get(`${baseUrl}/count_customer`).then((response) => {
    labels.value = response.data.data.labels;
    data.value = response.data.data.count;
    loaded.value = true;
  });
}

function deleteCustomer(id) {
  $swal
    .fire({
      title: "Do you want to save the changes?",
      showDenyButton: true,
      showCancelButton: true,
      confirmButtonText: "Save",
      denyButtonText: `Don't save`,
    })
    .then((result) => {
      if (result.isConfirmed) {
        doDelete(id)
          .then(() => {
            $swal.fire("Data Terhapus!", "", "success");
          })
          .catch((error) => {
            $swal.fire(error.response.data.data, "", "error");
          })
          .finally(() => {
            getCustomer(`${baseUrl}/customer`);
          });
      } else if (result.isDenied) {
        $swal.fire("Changes are not saved", "", "info");
      }
    });
}

function doDelete(id) {
  return axios.delete(`${baseUrl}/customer/${id}`);
}

onMounted(() => {
  getCustomer(`${baseUrl}/customer`);

  getCount();
});
</script>
<template>
  <div class="container mt-5">
    <button class="btn btn-primary" onclick="window.print()">Print Page!</button>
  </div>
  <div ref="pdfSection">
    <div class="container mt-5"></div>
    <div class="container mt-5">
      <div class="container">
        <div class="row justify-center items-center">
          <div class="col">
            <div>
              <Bar v-if="loaded" :data="chartData" :options="chartOptions" />
            </div>
          </div>
          <div class="col">
            <div>
              <DoughnutChart />
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="container mt-5">
      <table class="table table-bordered table-hover">
        <thead>
          <tr>
            <th scope="col">#ID</th>
            <th scope="col">Nama</th>
            <th scope="col">Jenis Kelamin</th>
            <th scope="col">Tanggal Lahir</th>
            <th scope="col">Pekerjaan</th>
            <th scope="col">Provinsi</th>
            <th scope="col">Kota</th>
            <th scope="col">Kecamatan</th>
            <th scope="col">Desa</th>
            <th scope="col">#Action</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-if="customersData != null"
            v-for="(item, i) in customersData.data.data.data"
            :key="i"
          >
            <th scope="row">{{ item.id }}</th>
            <td>{{ item.name }}</td>
            <td>{{ item.gender }}</td>
            <td>{{ item.birth_date }}</td>
            <td>{{ item.occupation }}</td>
            <td>{{ item.province.nama }}</td>
            <td>{{ item.city.nama }}</td>
            <td>{{ item.subdistrict.nama }}</td>
            <td>{{ item.village.nama }}</td>
            <td class="row justify-content-md-center">
              <NuxtLink
                role="button"
                :to="{ path: `/update`, query: { idCustomer: item.id } }"
                class="btn btn-primary btn-sm mb-2"
              >
                Edit
              </NuxtLink>
              <button
                type="button"
                class="btn btn-danger btn-sm"
                @click="deleteCustomer(item.id)"
              >
                Delete
              </button>
            </td>
          </tr>
        </tbody>
      </table>

      <nav aria-label="Page navigation" v-if="customersData != null">
        <ul class="pagination">
          <li
            class="page-item"
            v-for="link in customersData.data.data.links"
            :key="link.label"
            :class="{ disabled: !link.url }"
          >
            <button
              class="page-link"
              @click="getCustomer(link.url)"
              v-html="link.label"
            ></button>
          </li>
        </ul>
      </nav>
    </div>
  </div>
</template>
