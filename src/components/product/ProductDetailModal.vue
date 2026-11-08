
<script setup>
defineProps({
  show: {
    type: Boolean,
    default: false,
  },
  product: {
    type: Object,
    default: null,
  },
  quantity: {
    type: Number,
    default: 1,
  },
  subtotal: {
    type: Number,
    default: 0,
  },
  formatPrice: {
    type: Function,
    required: true,
  },
});

const emit = defineEmits([
  "close",
  "increase",
  "decrease",
  "checkout",
]);
</script>

<template>
  <div
    v-if="show && product"
    class="product-detail-overlay"
    @click.self="emit('close')"
  >
    <div class="product-detail-modal">
      <button
        class="product-detail-close"
        @click="emit('close')"
      >
        ×
      </button>

      <div class="product-detail-image">
        <img
          :src="product.image_url || product.image"
          :alt="product.name"
          @error="
            (event) =>
              (event.target.src =
                'https://placehold.co/500x400/f1f1f1/2438d2?text=Gans+Store')
          "
        />
      </div>

      <div class="product-detail-content">
        <div class="product-detail-main">
          <div class="product-detail-info">
            <h2>{{ product.name }}</h2>

            <p v-if="product.variant">
              {{ product.variant }}
            </p>

            <strong class="detail-price">
              {{ formatPrice(product.price) }}
            </strong>
          </div>

          <div class="product-detail-quantity">
            <div class="quantity-control">
              <button
                type="button"
                @click="emit('decrease')"
                :disabled="quantity <= 1"
              >
                −
              </button>

              <span>{{ quantity }}</span>

              <button
                type="button"
                @click="emit('increase')"
                :disabled="quantity >= product.stock"
              >
                +
              </button>
            </div>

            <span class="detail-stock">
              Stok: {{ product.stock }}
            </span>
          </div>
        </div>

        <div class="product-detail-divider"></div>

        <div class="product-detail-bottom">
          <div class="product-detail-description">
            <span>Deskripsi Produk</span>

            <p>
              {{ product.description || "Belum ada deskripsi produk." }}
            </p>
          </div>

          <div class="product-detail-checkout">
            <div class="detail-subtotal">
              <strong>Subtotal</strong>
              <strong>{{ formatPrice(subtotal) }}</strong>
            </div>

            <button
              class="detail-checkout-btn"
              @click="emit('checkout')"
              :disabled="product.stock < 1"
            >
              Check Out
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
