<script setup>
defineProps({
  userAddress: {
    type: Object,
    required: true,
  },
  checkoutProducts: {
    type: Array,
    required: true,
  },
  checkoutItemCount: {
    type: Number,
    required: true,
  },
  checkoutTotal: {
    type: Number,
    required: true,
  },
  paymentMethod: {
    type: String,
    required: true,
  },
  formatPrice: {
    type: Function,
    required: true,
  },
});

const emit = defineEmits([
  "back",
  "edit-address",
  "update:paymentMethod",
  "place-order",
]);
</script>

<template>
  <main class="main-content checkout-page">
    <div class="checkout-heading">
      <div>
        <h2>Checkout</h2>
        <p>Periksa kembali pesananmu sebelum melanjutkan.</p>
      </div>
      <button class="back-home-btn" @click="emit('back')">← Kembali</button>
    </div>

    <div class="checkout-layout">
      <section class="checkout-left">
        <div class="checkout-card">
          <div class="checkout-card-heading">
            <h3>Alamat Pengiriman</h3>
            <button @click="emit('edit-address')">Ubah Alamat</button>
          </div>
          <div class="saved-address">
            <strong>{{ userAddress.recipient }}</strong>
            <p>{{ userAddress.phone }}</p>
            <p>
              {{ userAddress.detail }}, {{ userAddress.district }},
              {{ userAddress.city }}, {{ userAddress.province }},
              {{ userAddress.postalCode }}
            </p>
          </div>
        </div>

        <div class="checkout-card">
          <h3>Produk yang Dipesan</h3>
          <div
            v-for="product in checkoutProducts"
            :key="product.id"
            class="checkout-product"
          >
            <img :src="product.image" :alt="product.name" />
            <div class="checkout-product-info">
              <strong>{{ product.name }}</strong>
              <p v-if="product.variant">{{ product.variant }}</p>
              <p>{{ product.quantity }} produk</p>
            </div>
            <strong class="checkout-product-price">
              {{ formatPrice(product.price * product.quantity) }}
            </strong>
          </div>
        </div>

        <div class="checkout-card">
          <h3>Metode Pembayaran</h3>
          <label class="payment-option">
            <input
              :checked="paymentMethod === 'COD'"
              type="radio"
              value="COD"
              @change="emit('update:paymentMethod', $event.target.value)"
            />
            <span>
              <strong>Cash on Delivery (COD)</strong>
              <small>Bayar ketika pesanan diterima</small>
            </span>
          </label>
          <label class="payment-option">
            <input
              :checked="paymentMethod === 'Transfer Bank'"
              type="radio"
              value="Transfer Bank"
              @change="emit('update:paymentMethod', $event.target.value)"
            />
            <span>
              <strong>Transfer Bank</strong>
              <small
                >Informasi pembayaran akan diberikan setelah pemesanan</small
              >
            </span>
          </label>
        </div>
      </section>

      <aside class="checkout-summary">
        <h3>Ringkasan Pesanan</h3>
        <div class="summary-row">
          <span>Total Produk</span>
          <span>{{ checkoutItemCount }} item</span>
        </div>
        <div class="summary-row">
          <span>Subtotal</span>
          <strong>{{ formatPrice(checkoutTotal) }}</strong>
        </div>
        <div class="summary-row">
          <span>Pengiriman</span>
          <span>Dikonfirmasi kemudian</span>
        </div>
        <div class="summary-divider"></div>
        <div class="summary-row summary-total">
          <strong>Total Belanja</strong>
          <strong>{{ formatPrice(checkoutTotal) }}</strong>
        </div>
        <button class="checkout-btn" @click="emit('place-order')">
          Buat Pesanan
        </button>
        <p class="checkout-note">
          Pastikan alamat dan produk pesanan sudah benar.
        </p>
      </aside>
    </div>
  </main>
</template>
