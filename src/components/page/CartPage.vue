<script setup>
defineProps({
  cartProducts: {
    type: Array,
    required: true,
  },
  cartTotal: {
    type: Number,
    required: true,
  },
  formatPrice: {
    type: Function,
    required: true,
  },
});

const emit = defineEmits([
  "home",
  "decrease",
  "increase",
  "remove",
  "checkout",
]);
</script>

<template>
  <main class="main-content cart-page">
    <div class="cart-heading">
      <div>
        <h2>Keranjang Belanja</h2>
        <p>Periksa kembali produk yang ingin kamu beli.</p>
      </div>
      <button class="back-home-btn" @click="emit('home')">
        ← Kembali Belanja
      </button>
    </div>

    <div v-if="cartProducts.length" class="cart-layout">
      <section class="cart-items">
        <article
          v-for="product in cartProducts"
          :key="product.id"
          class="cart-item"
        >
          <img
            class="cart-item-image"
            :src="product.image"
            :alt="product.name"
          />

          <div class="cart-item-info">
            <h3>{{ product.name }}</h3>
            <p v-if="product.variant">{{ product.variant }}</p>
            <strong>{{ formatPrice(product.price) }}</strong>

            <div class="cart-item-actions">
              <div class="quantity-control">
                <button type="button" @click="emit('decrease', product.id)">
                  −
                </button>
                <span>{{ product.quantity }}</span>
                <button
                  type="button"
                  :disabled="product.quantity >= product.stock"
                  @click="emit('increase', product.id)"
                >
                  +
                </button>
              </div>
            </div>
          </div>

          <div class="cart-item-subtotal">
            <strong>
              {{ formatPrice(product.price * product.quantity) }}
            </strong>

            <button
              type="button"
              class="remove-cart-btn"
              @click="emit('remove', product.id)"
            >
              Hapus
            </button>
          </div>
        </article>
      </section>

      <aside class="cart-summary">
        <h3>Ringkasan Belanja</h3>
        <div class="summary-row">
          <span>Total Produk</span>
          <span>
            {{ cartProducts.reduce((total, item) => total + item.quantity, 0) }}
            item
          </span>
        </div>
        <div class="summary-row">
          <span>Subtotal</span>
          <strong>{{ formatPrice(cartTotal) }}</strong>
        </div>
        <div class="summary-divider"></div>
        <div class="summary-row summary-total">
          <strong>Total Belanja</strong>
          <strong>{{ formatPrice(cartTotal) }}</strong>
        </div>
        <button class="checkout-btn" @click="emit('checkout')">
          Lanjut ke Checkout
        </button>
      </aside>
    </div>

    <div v-else class="cart-empty">
      <div class="cart-empty-icon">🛒</div>
      <h3>Keranjang Kamu Masih Kosong</h3>
      <p>
        Sepertinya kamu belum menambahkan produk apa pun. Yuk, temukan produk
        menarik di Gan's Store!
      </p>
      <button class="back-home-btn" @click="emit('home')">Mulai Belanja</button>
    </div>
  </main>
</template>
