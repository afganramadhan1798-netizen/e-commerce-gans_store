<script setup>
defineProps({
  categories: {
    type: Array,
    required: true,
  },
  searchQuery: {
    type: String,
    default: "",
  },
  activeCategory: {
    type: String,
    required: true,
  },
  isLoggedIn: {
    type: Boolean,
    default: false,
  },
  userRole: {
    type: String,
    default: "customer",
  },
  fullName: {
    type: String,
    default: "",
  },
  username: {
    type: String,
    default: "",
  },
  favoriteCount: {
    type: Number,
    default: 0,
  },
  cartItemCount: {
    type: Number,
    default: 0,
  },
});

const emit = defineEmits([
  "update:searchQuery",
  "update:activeCategory",
  "show-login",
  "show-register",
  "navigate",
  "logout",
]);
</script>

<template>
  <header class="navbar">
    <div class="logo">
      <div class="logo-circle">G</div>
      <div class="logo-text">
        <span>gan's</span>
        <span>store</span>
      </div>
    </div>

    <div class="navbar-main">
      <div class="search-box">
        <svg viewBox="0 0 24 24">
          <circle cx="11" cy="11" r="7"></circle>
          <path d="m16 16 5 5"></path>
        </svg>

        <input
          :value="searchQuery"
          type="text"
          placeholder="Search"
          @input="emit('update:searchQuery', $event.target.value)"
        />
      </div>

      <nav class="category-nav">
        <button
          v-for="category in categories"
          :key="category"
          :class="{ active: activeCategory === category }"
          @click="emit('update:activeCategory', category)"
        >
          {{ category }}
        </button>
      </nav>
    </div>

    <div v-if="!isLoggedIn" class="guest-actions">
      <button class="register-btn" @click="emit('show-register')">
        ▣ Daftar
      </button>
      <span class="slash">/</span>
      <button class="login-btn" @click="emit('show-login')">➜ Login</button>
    </div>

    <div v-else class="user-actions">
      <button
        class="wishlist-icon-btn"
        :aria-label="`Wishlist, ${favoriteCount} produk`"
        title="Wishlist"
        @click="emit('navigate', 'wishlist')"
      >
        <span aria-hidden="true">♡</span>
      </button>

      <button
        v-if="userRole === 'admin'"
        class="user-btn admin-dashboard-btn"
        @click="emit('navigate', 'admin')"
      >
        Admin
      </button>

      <div class="user-action-stack">
        <button class="user-btn" @click="emit('navigate', 'orders')">
          📦 Orders
        </button>
        <button class="user-btn" @click="emit('navigate', 'cart')">
          🛒 Cart <span class="cart-count">{{ cartItemCount }}</span>
        </button>
      </div>

      <div class="user-account-stack">
        <div class="user-info">
          <div class="user-avatar">
            {{ (fullName || username || "U").charAt(0).toUpperCase() }}
          </div>
          <span :title="fullName || username || 'User'">
            {{ fullName || username || "User" }}
          </span>
        </div>
        <button class="logout-btn" @click="emit('logout')">Logout</button>
      </div>
    </div>
  </header>
</template>
