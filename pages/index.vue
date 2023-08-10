<script setup>
import axios from "axios";
const { $swal } = useNuxtApp()
const provinces = ref([]);
const cities = ref([]);
const subdistricts = ref([]);
const villages = ref([]);
const config = useRuntimeConfig();

function deleteCustomer(id) {
  $swal.fire({
    title: 'Do you want to save the changes?',
    showDenyButton: true,
    showCancelButton: true,
    confirmButtonText: 'Save',
    denyButtonText: `Don't save`,
  }).then((result) => {
    if (result.isConfirmed) {
      doDelete(id).then(() => {
        $swal.fire('Data Terhapus!', '', 'success')
      }).catch((error) => {
        $swal.fire(error.response.data.data, '', 'error')
      }).finally(() => {
        getCustomer(`${baseUrl}/customer`);
      });
    } else if (result.isDenied) {
      $swal.fire('Changes are not saved', '', 'info')
    }
  })
}

function doDelete(id) {
  return axios
    .delete(`${baseUrl}/customer/${id}`)

}

const formErrors = ref({});
const formErrorsMessages = ref("");
const isSuccess = ref(false);

function create() {
  axios
    .post(`${baseUrl}/customer`, formData.value)
    .then((response) => {
      isSuccess.value = true;
    })
    .catch((error) => {
      formErrors.value = error.response.data.errors;
      formErrorsMessages.value = error.response.data.message;
    })
    .finally(() => {
      getCustomer(`${baseUrl}/customer`);
    });
}

const customersData = ref(null);

function getCustomer(url) {
  axios
    .get(url, formData.value)
    .then((response) => {
      customersData.value = response;
    })
    .catch((error) => {
      formErrorsMessages.value = error.response.data.message;
    })
    .finally();
}


function getProvince() {
  axios.get(`${baseUrlLocation}/provinsi.json`)
    .then(response => {
      provinces.value = response.data
    })
    .catch()
    .finally()
}

function getCity(provinceID, isChange = false) {
  axios.get(`${baseUrlLocation}/kabupaten/${provinceID.id}.json`)
    .then(response => {
      cities.value = response.data
    })
    .catch()
    .finally(() => {
      if (isChange) {
        formData.value.city = ""
        formData.value.subdistrict = ""
        formData.value.village = ""
      }
    })
}

function getSubdistrict(cityID, isChange = false) {
  axios.get(`${baseUrlLocation}/kecamatan/${cityID.id}.json`)
    .then(response => {
      subdistricts.value = response.data
    })
    .catch()
    .finally(() => {
      if (isChange) {
        formData.value.subdistrict = ""
        formData.value.village = ""
      }
    })
}

function getVillage(subdistrictID, isChange = false) {
  axios.get(`${baseUrlLocation}/kelurahan/${subdistrictID.id}.json`)
    .then(response => {
      villages.value = response.data
    })
    .catch()
    .finally(() => {
      if (isChange) {
        formData.value.village = ""
      }
    })
}


onMounted(() => {
  getCustomer(`${baseUrl}/customer`);

  getProvince();
  getCity(formData.value.province)
  getSubdistrict(formData.value.city)
  getVillage(formData.value.subdistrict)
});
const baseUrlLocation = "https://ibnux.github.io/data-indonesia/";

const baseUrl = config.public.API_BASE_URL;

const formData = ref({
  name: ref("Efo"),
  gender: ref("LAKI-LAKI"),
  birth_date: ref("1990-06-21"),  //yyyy-MM-dd
  occupation: ref("Programmer"),
  province: ref({ id: "11", nama: "ACEH" }),
  city: ref({ id: "1112", nama: "KAB. ACEH BARAT DAYA" }),
  subdistrict: ref({ id: "111203", nama: "Manggeng" }),
  village: ref({ id: "1112032026", nama: "Sejahtera" }),
});
</script>

<template>
  <div class="container mt-5">
    <div class="alert alert-danger" role="alert" v-if="formErrorsMessages">
      {{ formErrorsMessages }}
    </div>

    <div class="alert alert-success" role="alert" v-if="isSuccess">
      {{ "Sukses Input Customer" }}
    </div>

    <form @submit.prevent="create()">
      <div class="mb-3">
        <label for="name" class="form-label">Nama</label>
        <input type="text" class="form-control" :class="[formErrors.name ? 'is-invalid' : '']" id="name"
          v-model="formData.name" />
        <div :class="[formErrors.name ? 'invalid-feedback' : 'valid-feedback']">
          <span class="" v-for="error, i in formErrors.name" :key="i">
            {{ error }}
          </span>
        </div>
      </div>

      <div class="mb-3">
        <label for="gender" class="form-label">Jenis Kelamin</label>

        <select v-model="formData.gender" class="form-select" :class="[formErrors.gender ? 'is-invalid' : '']"
          id="gender">
          <option disabled value="">Pilih Jenis Kelamin</option>
          <option value="LAKI-LAKI">LAKI-LAKI</option>
          <option value="PEREMPUAN">PEREMPUAN</option>
        </select>

        <div :class="[formErrors.gender ? 'invalid-feedback' : 'valid-feedback']">
          <span class="" v-for="error, i in formErrors.gender" :key="i">
            {{ error }}
          </span>
        </div>
      </div>

      <div class="mb-3">
        <label for="birth_date" class="form-label">Tanggal Lahir</label>
        <input type="date" class="form-control" id="birth_date" v-model="formData.birth_date"
          :class="[formErrors.birth_date ? 'is-invalid' : '']" />

        <div :class="[
          formErrors.birth_date ? 'invalid-feedback' : 'valid-feedback',
        ]">
          <span class="" v-for="error, i in formErrors.birth_date" :key="i">
            {{ error }}
          </span>
        </div>
      </div>
      <div class="mb-3">
        <label for="occupation" class="form-label">Pekerjaan</label>
        <input type="text" class="form-control" id="occupation" v-model="formData.occupation"
          :class="[formErrors.occupation ? 'is-invalid' : '']" />

        <div :class="[
          formErrors.occupation ? 'invalid-feedback' : 'valid-feedback',
        ]">
          <span class="" v-for="error, i in formErrors.occupation" :key="i">
            {{ error }}
          </span>
        </div>
      </div>
      <div class="mb-3">
        <label for="province" class="form-label">Provinsi</label>

        <select v-model="formData.province" class="form-select" :class="[formErrors.province ? 'is-invalid' : '']"
          id="province" @change="getCity(formData.province, true)">
          <option disabled value="">Pilih Provinsi</option>
          <option v-for="item, i in  provinces" :key="i" :value="{ id: item.id, nama: item.nama }">{{ item.nama }}
          </option>
        </select>

        <div :class="[formErrors.province ? 'invalid-feedback' : 'valid-feedback']">
          <span class="" v-for="error, i in formErrors.province" :key="i">
            {{ error }}
          </span>
        </div>
      </div>

      <div class="mb-3">
        <label for="city" class="form-label">Kota</label>

        <select v-model="formData.city" class="form-select" :class="[formErrors.city ? 'is-invalid' : '']" id="city"
          @change="getSubdistrict(formData.city, true)">
          <option disabled value="">Pilih Kota</option>
          <option v-for="item, i in  cities" :key="i" :value="{ id: item.id, nama: item.nama }">{{ item.nama }}</option>
        </select>

        <div :class="[formErrors.city ? 'invalid-feedback' : 'valid-feedback']">
          <span class="" v-for="error, i in formErrors.city" :key="i">
            {{ error }}
          </span>
        </div>
      </div>
      <div class="mb-3">
        <label for="subdistrict" class="form-label">Kecamatan</label>

        <select v-model="formData.subdistrict" class="form-select" :class="[formErrors.subdistrict ? 'is-invalid' : '']"
          @change="getVillage(formData.subdistrict, true)" id="subdistrict">
          <option disabled value="">Pilih Kecamatan</option>
          <option v-for="item, i in  subdistricts" :key="i" :value="{ id: item.id, nama: item.nama }">{{ item.nama }}
          </option>
        </select>

        <div :class="[
          formErrors.subdistrict ? 'invalid-feedback' : 'valid-feedback',
        ]">
          <span class="" v-for="error, i in formErrors.subdistrict" :key="i">
            {{ error }}
          </span>
        </div>
      </div>
      <div class="mb-3">
        <label for="village" class="form-label">Desa</label>
        <select v-model="formData.village" class="form-select" :class="[formErrors.village ? 'is-invalid' : '']"
          id="village">
          <option disabled value="">Pilih Desa</option>
          <option v-for="item, i in  villages" :key="i" :value="{ id: item.id, nama: item.nama }">{{ item.nama }}</option>
        </select>
        <div :class="[formErrors.village ? 'invalid-feedback' : 'valid-feedback']">
          <span class="" v-for="error, i in formErrors.village" :key="i">
            {{ error }}
          </span>
        </div>
      </div>

      <button type="submit" class="btn btn-primary">Submit</button>
    </form>
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
        <tr v-if="customersData != null" v-for="item, i in customersData.data.data.data" :key="i">
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
            <NuxtLink role="button" :to="{ path: `/update`, query: { idCustomer: item.id } }"
              class="btn btn-primary btn-sm mb-2">
              Edit
            </NuxtLink>
            <button type="button" class="btn btn-danger btn-sm" @click="deleteCustomer(item.id)">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>

    <nav aria-label="Page navigation" v-if="customersData != null">
      <ul class="pagination">
        <li class="page-item" v-for="link in customersData.data.data.links" :key="link.label"
          :class="{ disabled: !link.url }">
          <button class="page-link" @click="getCustomer(link.url)" v-html="link.label"></button>
        </li>
      </ul>
    </nav>
  </div>
</template>
