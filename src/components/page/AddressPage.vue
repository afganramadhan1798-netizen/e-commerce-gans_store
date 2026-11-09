
<script setup>
defineProps({
  addressForm: {
    type: Object,
    required: true,
  },
  hasSavedAddress: {
    type: Boolean,
    default: false,
  },
});

const emit = defineEmits(["back", "cancel", "save"]);
</script>

<template>
  <main class="main-content address-page">
    <div class="checkout-heading">
      <div>
        <h2>
          {{ hasSavedAddress ? "Ubah Alamat Pengiriman" : "Tambah Alamat Pengiriman" }}
        </h2>
        <p>
          {{
            hasSavedAddress
              ? "Perbarui alamat utama untuk pengiriman pesananmu."
              : "Lengkapi alamat agar pesananmu dapat dikirim dengan benar."
          }}
        </p>
      </div>

      <button class="back-home-btn" @click="emit('back')">
        ← Kembali
      </button>
    </div>

    <section class="address-form-card">
      <h3>Informasi Penerima</h3>

      <p v-if="hasSavedAddress" class="address-saved-info">
        Alamat utama kamu sudah tersimpan. Perubahan yang disimpan akan
        menggantikan alamat sebelumnya.
      </p>

      <form @submit.prevent="emit('save')">
        <div class="address-form-grid">
          <div class="address-field">
            <label>Nama Penerima</label>
            <input
              v-model="addressForm.recipient"
              type="text"
              placeholder="Masukkan nama penerima"
              required
            />
          </div>

          <div class="address-field">
            <label>Nomor Telepon</label>
            <input
              v-model="addressForm.phone"
              type="tel"
              placeholder="Contoh: 081234567890"
              required
            />
          </div>

          <div class="address-field">
            <label>Provinsi</label>
            <input
              v-model="addressForm.province"
              type="text"
              placeholder="Masukkan provinsi"
              required
            />
          </div>

          <div class="address-field">
            <label>Kota / Kabupaten</label>
            <input
              v-model="addressForm.city"
              type="text"
              placeholder="Masukkan kota atau kabupaten"
              required
            />
          </div>

          <div class="address-field">
            <label>Kecamatan</label>
            <input
              v-model="addressForm.district"
              type="text"
              placeholder="Masukkan kecamatan"
              required
            />
          </div>

          <div class="address-field">
            <label>Kode Pos</label>
            <input
              v-model="addressForm.postalCode"
              type="text"
              placeholder="Masukkan kode pos"
              required
            />
          </div>

          <div class="address-field full-width">
            <label>Detail Alamat</label>
            <textarea
              v-model="addressForm.detail"
              rows="4"
              placeholder="Nama jalan, nomor rumah, RT/RW, patokan, dan sebagainya"
              required
            ></textarea>
          </div>
        </div>

        <div class="address-form-actions">
          <button
            type="button"
            class="address-cancel-btn"
            @click="emit('cancel')"
          >
            Batal
          </button>

          <button type="submit" class="address-save-btn">
            {{ hasSavedAddress ? "Simpan Perubahan" : "Simpan Alamat" }}
          </button>
        </div>
      </form>
    </section>
  </main>
</template>
