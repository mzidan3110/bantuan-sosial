<template>
  <div class="form-container">
    <h1>Form Pendataan Bantuan Sosial</h1>
    <form @submit.prevent="submitForm">
      <div class="form-group">
        <label>Nama:</label>
        <input type="text" v-model="formData.nama" required />
      </div>

      <div class="form-group">
        <label>NIK:</label>
        <input type="number" v-model="formData.nik" required />
        <span v-if="nikError" class="error">{{ nikError }}</span>
      </div>

      <div class="form-group">
        <label>Nomor Kartu Keluarga:</label>
        <input type="number" v-model="formData.noKK" required />
        <span v-if="noKKError" class="error">{{ noKKError }}</span>
      </div>

      <div class="form-group">
        <label>Foto KTP:</label>
        <input type="file" @change="onFileChange('fotoKTP', $event)" required />
      </div>

      <div class="form-group">
        <label>Foto Kartu Keluarga:</label>
        <input type="file" @change="onFileChange('fotoKK', $event)" required />
      </div>

      <div class="form-group">
        <label>Umur:</label>
        <input type="number" v-model="formData.umur" min="25" required />
      </div>

      <div class="form-group">
        <label>Jenis Kelamin:</label>
        <select v-model="formData.jenisKelamin" required>
          <option value="Laki-laki">Laki-laki</option>
          <option value="Perempuan">Perempuan</option>
        </select>
      </div>

      <div class="form-group">
        <label>Provinsi:</label>
        <select v-model="provinceId" @change="fetchKota">
          <option value="" disabled selected>Pilih Provinsi</option>
          <option v-for="provinsi in provinsiList" :key="provinsi.id" :value="provinsi.id">
            {{ provinsi.name }}
          </option>
        </select>
      </div>

      <div class="form-group">
        <label>Kab/Kota:</label>
        <select v-model="regencyId" @change="fetchKecamatan">
          <option v-for="kota in kotaList" :key="kota.id" :value="kota.id"> {{ kota.name }}</option>
        </select>
      </div>

      <div class="form-group">
        <label>Kecamatan:</label>
        <select v-model="districtId" @change="fetchKelurahan" required>
          <option v-for="kecamatan in kecamatanList" :key="kecamatan.id" :value="kecamatan.id">{{ kecamatan.name }}
          </option>
        </select>
      </div>

      <div class="form-group">
        <label>Kelurahan:</label>
        <select v-model="villagesId" required>
          <option v-for="kelurahan in kelurahanList" :key="kelurahan.id" :value="kelurahan.id">{{ kelurahan.name }}
          </option>
        </select>
      </div>

      <div class="form-group">
        <label>Alamat:</label>
        <input type="text" v-model="formData.alamat" maxlength="255" required />
      </div>

      <div class="form-group">
        <label>RT:</label>
        <input type="text" v-model="formData.rt" required />
      </div>

      <div class="form-group">
        <label>RW:</label>
        <input type="text" v-model="formData.rw" required />
      </div>

      <div class="form-group">
        <label>Penghasilan Sebelum Pandemi:</label>
        <input type="number" v-model="formData.penghasilanSebelum" required />
      </div>

      <div class="form-group">
        <label>Penghasilan Setelah Pandemi:</label>
        <input type="number" v-model="formData.penghasilanSetelah" required />
      </div>

      <div class="form-group">
        <label>Alasan Memerlukan Bantuan:</label>
        <select v-model="formData.alasan" required>
          <option value="Kehilangan pekerjaan">Kehilangan pekerjaan</option>
          <option value="Kepala keluarga">Kepala keluarga</option>
          <option value="Fakir/miskin">Fakir/miskin</option>
          <option value="Lainnya">Lainnya</option>
        </select>
      </div>

      <div class="form-group">
        <input type="checkbox" v-model="formData.deklarasi" required />
        <label>Saya menyatakan bahwa data yang diisikan adalah benar.</label>
      </div>

      <button type="submit" class="submit-button">Submit</button>
    </form>

    <div v-if="formSubmitted">
      <h2>Preview Data</h2>
      <pre>{{ formData }}</pre>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      formData: {
        nama: "",
        nik: "",
        noKK: "",
        fotoKTP: null,
        fotoKK: null,
        umur: "",
        jenisKelamin: "",
        provinsi: "",
        kota: "",
        kecamatan: "",
        kelurahan: "",
        alamat: "",
        rt: "",
        rw: "",
        penghasilanSebelum: "",
        penghasilanSetelah: "",
        alasan: "",
        deklarasi: false,
      },
      provinsiList: [],
      kotaList: [],
      kecamatanList: [],
      kelurahanList: [],
      existingNik: [],
      existingNoKK: [],
      nikError: "",
      noKKError: "",
      formSubmitted: false,
    };
  },
  created() {
    this.fetchProvinsi();
  },
  methods: {
    fetchProvinsi() {
      axios
        .get("https://www.emsifa.com/api-wilayah-indonesia/api/provinces.json")
        .then((response) => {
          this.provinsiList = response.data;
        })
        .catch((error) => {
          console.error("Error fetching provinces:", error);
        });
    },

    fetchKota() {
      if (!this.provinceId) {
        console.error("Province ID is undefined. Please select a province.");
        return;
      }


      axios
        .get(`https://www.emsifa.com/api-wilayah-indonesia/api/regencies/${this.provinceId}.json`)
        .then((response) => {
          this.kotaList = response.data;
        })
        .catch((error) => {
          console.error("Error fetching cities:", error);
        });
    },

    fetchKecamatan() {
      if (!this.regencyId) {
        console.error("City is undefined. Please select a city.");
        return;
      }

      axios
        .get(`https://www.emsifa.com/api-wilayah-indonesia/api/districts/${this.regencyId}.json`)
        .then((response) => {
          this.kecamatanList = response.data;
        })
        .catch((error) => {
          console.error("Error fetching districts:", error);
        });
    },

    fetchKelurahan() {
      if (!this.districtId) {
        console.error("District is undefined. Please select a district.");
        return;
      }

      axios
        .get(`https://www.emsifa.com/api-wilayah-indonesia/api/villages/${this.districtId}.json`)
        .then((response) => {
          this.kelurahanList = response.data;
        })
        .catch((error) => {
          console.error("Error fetching villages:", error);
        });
    },

    onProvinceChange() {
      this.provinceId = this.formData.provinsi; // Set ID provinsi
      this.fetchKota(); // Ambil kota untuk provinsi yang dipilih
    },

    onCityChange() {
      this.formData.kota = this.selectedCity; // Set kota yang dipilih
      this.fetchKecamatan(); // Ambil kecamatan untuk kota yang dipilih
    },

    onFileChange(field, event) {
      const file = event.target.files[0];
      if (file.size <= 2097152) {
        this.formData[field] = file;
      } else {
        alert("File size exceeds 2MB.");
      }
    },
    validateNIK() {
      this.nikError = ""; // Reset error
      const nik = String(this.formData.nik).trim(); // Mengubah NIK menjadi string dan menghapus spasi

      // Validasi panjang NIK
      if (nik.length !== 16) {
        this.nikError = "NIK harus terdiri dari 16 digit.";
        return false; // Mengembalikan false jika validasi gagal
      }

      // Validasi apakah NIK sudah ada
      if (this.existingNik.includes(nik)) {
        this.nikError = "NIK sudah terdaftar.";
        return false; // Mengembalikan false jika validasi gagal
      }

      return true; // Mengembalikan true jika validasi berhasil
    },

    validateNoKK() {
      this.noKKError = ""; // Reset error
      const noKK = String(this.formData.noKK).trim(); // Mengubah No KK menjadi string dan menghapus spasi

      // Validasi panjang No KK
      if (noKK.length !== 16) {
        this.noKKError = "Nomor Kartu Keluarga harus terdiri dari 16 digit.";
        return false; // Mengembalikan false jika validasi gagal
      }

      // Validasi apakah No KK sudah ada
      if (this.existingNoKK.includes(noKK)) {
        this.noKKError = "Nomor Kartu Keluarga sudah terdaftar.";
        return false; // Mengembalikan false jika validasi gagal
      }

      return true; // Mengembalikan true jika validasi berhasil
    },

    submitForm() {
      // Validasi NIK dan No KK
      const isNikValid = this.validateNIK();
      const isNoKKValid = this.validateNoKK();

      // Jika ada error, tampilkan alert dan hentikan proses submit
      if (!isNikValid) {
        alert(this.nikError);
        return; // Hentikan proses jika ada error pada NIK
      }
      if (!isNoKKValid) {
        alert(this.noKKError);
        return; // Hentikan proses jika ada error pada No KK
      }

      // Simulasi pengiriman data dengan waktu respons
      const success = Math.random() > 0.2; // 80% chance of success
      setTimeout(() => {
        if (success) {
          this.formSubmitted = true;
          alert("Data submitted successfully.");

          // Simpan NIK dan No KK setelah pengiriman sukses
          this.existingNik.push(this.formData.nik);
          this.existingNoKK.push(this.formData.noKK);
         
        } else {
          alert("Submission failed. Please try again.");
        }
      }, 1500);
    },

    
  },
};
</script>

<style scoped>
.form-container {
  max-width: 600px;
  margin: 0 auto;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  padding: 20px;
}

h1 {
  text-align: center;
  color: #333;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

input,
select,
button {
  width: 100%;
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #ccc;
  box-shadow: inset 0 2px 5px rgba(0, 0, 0, 0.1);
  transition: border-color 0.3s;
}

input:focus,
select:focus {
  border-color: #007bff;
  outline: none;
}

.submit-button {
  background-color: #007bff;
  color: white;
  border: none;
  cursor: pointer;
  font-size: 16px;
}

.submit-button:hover {
  background-color: #0056b3;
}

.error {
  color: red;
  font-size: 0.9em;
  margin-top: 5px;
}
</style>