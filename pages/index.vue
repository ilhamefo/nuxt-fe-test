<script setup>
import axios from "axios";
const provinces = ref([]);
const cities = ref([]);
const subdistricts = ref([]);
const villages = ref([]);
const config = useRuntimeConfig();

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
    });
}

function getProvince() {
  axios
    .get(`${baseUrlLocation}/provinsi.json`)
    .then((response) => {
      provinces.value = response.data;
    })
    .catch()
    .finally();
}

function getCity(provinceID, isChange = false) {
  axios
    .get(`${baseUrlLocation}/kabupaten/${provinceID.id}.json`)
    .then((response) => {
      cities.value = response.data;
    })
    .catch()
    .finally(() => {
      if (isChange) {
        formData.value.city = "";
        formData.value.subdistrict = "";
        formData.value.village = "";
      }
    });
}

function getSubdistrict(cityID, isChange = false) {
  axios
    .get(`${baseUrlLocation}/kecamatan/${cityID.id}.json`)
    .then((response) => {
      subdistricts.value = response.data;
    })
    .catch()
    .finally(() => {
      if (isChange) {
        formData.value.subdistrict = "";
        formData.value.village = "";
      }
    });
}

function getVillage(subdistrictID, isChange = false) {
  axios
    .get(`${baseUrlLocation}/kelurahan/${subdistrictID.id}.json`)
    .then((response) => {
      villages.value = response.data;
    })
    .catch()
    .finally(() => {
      if (isChange) {
        formData.value.village = "";
      }
    });
}

onMounted(() => {

  getProvince();
  getCity(formData.value.province);
  getSubdistrict(formData.value.city);
  getVillage(formData.value.subdistrict);
});
const baseUrlLocation = "https://ibnux.github.io/data-indonesia/";

const baseUrl = config.public.API_BASE_URL;

const formData = ref({
  name: ref("Efo"),
  gender: ref("LAKI-LAKI"),
  birth_date: ref("1990-06-21"), //yyyy-MM-dd
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

    <NuxtLink
      role="button"
      :to="{ path: `/customers` }"
      class="btn btn-primary btn-sm mb-5"
    >
      Lihat Data Customer
    </NuxtLink>

    <form @submit.prevent="create()">
      <div class="mb-3">
        <label for="name" class="form-label">Nama</label>
        <input
          type="text"
          class="form-control"
          :class="[formErrors.name ? 'is-invalid' : '']"
          id="name"
          v-model="formData.name"
        />
        <div :class="[formErrors.name ? 'invalid-feedback' : 'valid-feedback']">
          <span class="" v-for="(error, i) in formErrors.name" :key="i">
            {{ error }}
          </span>
        </div>
      </div>

      <div class="mb-3">
        <label for="gender" class="form-label">Jenis Kelamin</label>

        <select
          v-model="formData.gender"
          class="form-select"
          :class="[formErrors.gender ? 'is-invalid' : '']"
          id="gender"
        >
          <option disabled value="">Pilih Jenis Kelamin</option>
          <option value="LAKI-LAKI">LAKI-LAKI</option>
          <option value="PEREMPUAN">PEREMPUAN</option>
        </select>

        <div
          :class="[formErrors.gender ? 'invalid-feedback' : 'valid-feedback']"
        >
          <span class="" v-for="(error, i) in formErrors.gender" :key="i">
            {{ error }}
          </span>
        </div>
      </div>

      <div class="mb-3">
        <label for="birth_date" class="form-label">Tanggal Lahir</label>
        <input
          type="date"
          class="form-control"
          id="birth_date"
          v-model="formData.birth_date"
          :class="[formErrors.birth_date ? 'is-invalid' : '']"
        />

        <div
          :class="[
            formErrors.birth_date ? 'invalid-feedback' : 'valid-feedback',
          ]"
        >
          <span class="" v-for="(error, i) in formErrors.birth_date" :key="i">
            {{ error }}
          </span>
        </div>
      </div>
      <div class="mb-3">
        <label for="occupation" class="form-label">Pekerjaan</label>
        <input
          type="text"
          class="form-control"
          id="occupation"
          v-model="formData.occupation"
          :class="[formErrors.occupation ? 'is-invalid' : '']"
        />

        <div
          :class="[
            formErrors.occupation ? 'invalid-feedback' : 'valid-feedback',
          ]"
        >
          <span class="" v-for="(error, i) in formErrors.occupation" :key="i">
            {{ error }}
          </span>
        </div>
      </div>
      <div class="mb-3">
        <label for="province" class="form-label">Provinsi</label>

        <select
          v-model="formData.province"
          class="form-select"
          :class="[formErrors.province ? 'is-invalid' : '']"
          id="province"
          @change="getCity(formData.province, true)"
        >
          <option disabled value="">Pilih Provinsi</option>
          <option
            v-for="(item, i) in provinces"
            :key="i"
            :value="{ id: item.id, nama: item.nama }"
          >
            {{ item.nama }}
          </option>
        </select>

        <div
          :class="[formErrors.province ? 'invalid-feedback' : 'valid-feedback']"
        >
          <span class="" v-for="(error, i) in formErrors.province" :key="i">
            {{ error }}
          </span>
        </div>
      </div>

      <div class="mb-3">
        <label for="city" class="form-label">Kota</label>

        <select
          v-model="formData.city"
          class="form-select"
          :class="[formErrors.city ? 'is-invalid' : '']"
          id="city"
          @change="getSubdistrict(formData.city, true)"
        >
          <option disabled value="">Pilih Kota</option>
          <option
            v-for="(item, i) in cities"
            :key="i"
            :value="{ id: item.id, nama: item.nama }"
          >
            {{ item.nama }}
          </option>
        </select>

        <div :class="[formErrors.city ? 'invalid-feedback' : 'valid-feedback']">
          <span class="" v-for="(error, i) in formErrors.city" :key="i">
            {{ error }}
          </span>
        </div>
      </div>
      <div class="mb-3">
        <label for="subdistrict" class="form-label">Kecamatan</label>

        <select
          v-model="formData.subdistrict"
          class="form-select"
          :class="[formErrors.subdistrict ? 'is-invalid' : '']"
          @change="getVillage(formData.subdistrict, true)"
          id="subdistrict"
        >
          <option disabled value="">Pilih Kecamatan</option>
          <option
            v-for="(item, i) in subdistricts"
            :key="i"
            :value="{ id: item.id, nama: item.nama }"
          >
            {{ item.nama }}
          </option>
        </select>

        <div
          :class="[
            formErrors.subdistrict ? 'invalid-feedback' : 'valid-feedback',
          ]"
        >
          <span class="" v-for="(error, i) in formErrors.subdistrict" :key="i">
            {{ error }}
          </span>
        </div>
      </div>
      <div class="mb-3">
        <label for="village" class="form-label">Desa</label>
        <select
          v-model="formData.village"
          class="form-select"
          :class="[formErrors.village ? 'is-invalid' : '']"
          id="village"
        >
          <option disabled value="">Pilih Desa</option>
          <option
            v-for="(item, i) in villages"
            :key="i"
            :value="{ id: item.id, nama: item.nama }"
          >
            {{ item.nama }}
          </option>
        </select>
        <div
          :class="[formErrors.village ? 'invalid-feedback' : 'valid-feedback']"
        >
          <span class="" v-for="(error, i) in formErrors.village" :key="i">
            {{ error }}
          </span>
        </div>
      </div>

      <button type="submit" class="btn btn-primary">Submit</button>
    </form>
  </div>
</template>
