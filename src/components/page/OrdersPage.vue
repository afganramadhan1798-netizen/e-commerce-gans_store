<script setup>
defineProps({
  orders: {
    type: Array,
    required: true,
  },
  formatPrice: {
    type: Function,
    required: true,
  },
});

const emit = defineEmits(["home"]);
</script>

<template>
  <main class="main-content orders-page">
    <div class="orders-heading">
      <div>
        <h2>Pesanan Saya</h2>
        <p>Lihat dan pantau semua pesananmu di Gan's Store.</p>
      </div>
      <button class="back-home-btn" @click="emit('home')">
        ← Kembali Belanja
      </button>
    </div>

    <div v-if="orders.length" class="orders-list">
      <article v-for="order in orders" :key="order.id" class="order-card">
        <div class="order-card-header">
          <div>
            <span class="order-label">Nomor Pesanan</span>
            <strong>{{ order.id }}</strong>
          </div>
          <span class="order-status">{{ order.status }}</span>
        </div>

        <div class="order-date">{{ order.date }}</div>

        <div class="order-products">
          <div
            v-for="(product, index) in order.items"
            :key="product.id + '-' + index"
            class="order-product"
          >
            <img :src="product.image" :alt="product.name" />
            <div class="order-product-info">
              <strong>{{ product.name }}</strong>
              <p v-if="product.variant">{{ product.variant }}</p>
              <p>{{ product.quantity }} produk</p>
            </div>
            <strong class="order-product-price">
              {{ formatPrice(product.price * product.quantity) }}
            </strong>
          </div>
        </div>

        <div class="order-card-footer">
          <div>
            <span>Metode Pembayaran</span>
            <strong>{{ order.paymentMethod }}</strong>
          </div>
          <div class="order-total">
            <span>Total Pesanan</span>
            <strong>{{ formatPrice(order.total) }}</strong>
          </div>
        </div>

        <div class="order-address">
          <strong>Alamat Pengiriman</strong>
          <p>{{ order.address.recipient }} · {{ order.address.phone }}</p>
          <p>
            {{ order.address.detail }}, {{ order.address.district }},
            {{ order.address.city }}, {{ order.address.province }},
            {{ order.address.postalCode }}
          </p>
        </div>
      </article>
    </div>

    <div v-else class="orders-empty">
      <div class="orders-empty-icon">📦</div>
      <h3>Belum Ada Pesanan</h3>
      <p>
        Kamu belum memiliki riwayat pesanan. Yuk, temukan produk menarik di
        Gan's Store!
      </p>
      <button class="back-home-btn" @click="emit('home')">Mulai Belanja</button>
    </div>
  </main>
</template>
