<script setup>
import { ref } from "vue";
import ProductCard from "../product/ProductCard.vue";

defineProps({
  activeCategory: {
    type: String,
    required: true,
  },
  filteredProducts: {
    type: Array,
    required: true,
  },
  favorites: {
    type: Array,
    default: () => [],
  },
  isLoggedIn: {
    type: Boolean,
    default: false,
  },
  formatPrice: {
    type: Function,
    required: true,
  },
});

const emit = defineEmits(["toggle-favorite", "add-to-cart", "order"]);
const productList = ref(null);

function scrollProducts(direction) {
  productList.value?.scrollBy({
    left: direction * 350,
    behavior: "smooth",
  });
}
</script>

<template>
  <main class="main-content">
    <section class="hero-banner">
      <div class="banner-illustration">
        <svg viewBox="0 0 150 150">
          <rect x="10" y="10" width="130" height="105" rx="12" />
          <path d="M10 95 H140" />
          <circle cx="75" cy="105" r="4" />
          <path d="M55 115 L50 135 H100 L95 115" />
          <path d="M45 48 H105 L98 83 H52 Z" />
          <path d="M58 48 V40 A17 17 0 0 1 92 40 V48" />
          <path d="M68 60 L72 74 M83 60 L79 74" />
        </svg>
      </div>

      <div class="banner-content">
        <h1>
          Selamat Datang di<br />
          Gan's Store!
        </h1>
        <p>
          Temukan berbagai produk pilihan dengan kualitas terbaik <br />
          dan harga terjangkau di gan's store!
        </p>
      </div>

      <div class="banner-illustration">
        <svg viewBox="0 0 150 150">
          <path
            d="M25 65 C5 45 20 20 45 25 C50 0 95 0 100 30 C130 15 150 50 125 70"
          />
          <path d="M75 40 V85 M55 68 L75 90 L95 68" />
          <path d="M35 90 L20 105 L30 135 H120 L130 105 L115 90" />
          <path d="M20 105 H130 M55 105 L60 135 M95 105 L90 135" />
        </svg>
      </div>
    </section>

    <section class="product-section">
      <div class="section-heading">
        <div>
          <h2>{{ activeCategory }}</h2>
          <p>Temukan produk pilihan dari Gan's Store</p>
        </div>

        <div class="slider-controls">
          <button @click="scrollProducts(-1)">‹</button>
          <button @click="scrollProducts(1)">›</button>
        </div>
      </div>

      <div
        v-if="filteredProducts.length"
        ref="productList"
        class="product-list"
        :class="{ 'logged-in': isLoggedIn }"
      >
        <ProductCard
          v-for="product in filteredProducts"
          :key="product.id"
          :product="product"
          :favorites="favorites"
          :is-logged-in="isLoggedIn"
          :format-price="formatPrice"
          @toggle-favorite="emit('toggle-favorite', $event)"
          @add-to-cart="emit('add-to-cart', $event)"
          @order="emit('order', $event)"
        />
      </div>

      <div v-else class="empty-products">
        <h3>Produk tidak ditemukan</h3>
        <p>Coba gunakan kata kunci atau kategori lainnya.</p>
      </div>
    </section>
  </main>
</template>
