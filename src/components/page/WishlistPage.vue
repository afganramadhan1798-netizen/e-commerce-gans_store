<script setup>
defineProps({
  wishlistProducts: {
    type: Array,
    required: true,
  },
  formatPrice: {
    type: Function,
    required: true,
  },
});

const emit = defineEmits(["home", "toggle-favorite", "add-to-cart"]);
</script>

<template>
  <main class="main-content wishlist-page">
    <div class="wishlist-heading">
      <div>
        <h2>My Wishlist</h2>
        <p>Produk pilihan yang kamu sukai di Gan's Store.</p>
      </div>
      <button class="back-home-btn" @click="emit('home')">
        ← Kembali Belanja
      </button>
    </div>

    <div v-if="wishlistProducts.length" class="wishlist-grid">
      <article
        v-for="product in wishlistProducts"
        :key="product.id"
        class="product-card wishlist-card"
      >
        <div class="product-image">
          <img
            :src="product.image"
            :alt="product.name"
            @error="
              (event) =>
                (event.target.src =
                  'https://placehold.co/500x400/f1f1f1/2438d2?text=Gans+Store')
            "
          />
          <button
            class="favorite-btn favorited"
            title="Hapus dari Wishlist"
            @click="emit('toggle-favorite', product.id)"
          >
            ♥
          </button>
        </div>

        <div class="product-info">
          <div class="product-description">
            <h3>{{ product.name }}</h3>
            <p v-if="product.variant">{{ product.variant }}</p>
            <p class="product-price">{{ formatPrice(product.price) }}</p>
          </div>

          <div class="wishlist-actions">
            <button
              class="wishlist-cart-btn"
              @click="emit('add-to-cart', product)"
            >
              🛒 Tambah ke Keranjang
            </button>
            <button
              class="wishlist-remove-btn"
              title="Hapus produk"
              @click="emit('toggle-favorite', product.id)"
            >
              Hapus
            </button>
          </div>
        </div>
      </article>
    </div>

    <div v-else class="wishlist-empty">
      <div class="wishlist-empty-icon">♡</div>
      <h3>Wishlist Kamu Masih Kosong</h3>
      <p>Belum ada produk yang kamu simpan. Yuk, temukan produk favoritmu!</p>
      <button class="back-home-btn" @click="emit('home')">Mulai Belanja</button>
    </div>
  </main>
</template>
