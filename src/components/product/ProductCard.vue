<script setup>
defineProps({
  product: {
    type: Object,
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
</script>

<template>
  <article class="product-card">
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
        class="favorite-btn"
        :class="{ favorited: favorites.includes(product.id) }"
        @click="emit('toggle-favorite', product.id)"
      >
        {{ favorites.includes(product.id) ? "♥" : "♡" }}
      </button>

      <button
        v-if="isLoggedIn"
        class="quick-cart"
        title="Tambah ke keranjang"
        @click="emit('add-to-cart', product)"
      >
        🛒
      </button>
    </div>

    <div class="product-info">
      <div class="product-description">
        <h3>{{ product.name }}</h3>
        <p v-if="product.variant">{{ product.variant }}</p>
        <p class="product-price">
          {{ formatPrice(product.price) }}
        </p>
      </div>

      <button
        v-if="isLoggedIn"
        class="order-btn"
        @click="emit('order', product)"
      >
        Pesan
      </button>
    </div>
  </article>
</template>
