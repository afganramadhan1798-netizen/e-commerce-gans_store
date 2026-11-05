<script setup>
import { ref, computed } from "vue";
import { products as productData, categories } from "./data/products";
import ProductCard from "./components/product/ProductCard.vue";
import ProductDetailModal from "./components/product/ProductDetailModal.vue";

const isLoggedIn = ref(false);
const showLogin = ref(false);
const showRegister = ref(false);

const username = ref("");
const password = ref("");
const fullName = ref("");
const email = ref("");
const registerUsername = ref("");
const registerPassword = ref("");
const confirmPassword = ref("");

const searchQuery = ref("");
const activeCategory = ref("Best Seller");
const favorites = ref([]);
const cart = ref([]);
const activePage = ref("home");
const orders = ref([]);

const showProductDetail = ref(false);
const selectedProduct = ref(null);
const selectedQuantity = ref(1);

const products = ref(productData);
const directCheckoutProduct = ref(null);

// Data alamat pengguna
const userAddress = ref({
  recipient: "",
  phone: "",
  province: "",
  city: "",
  district: "",
  postalCode: "",
  detail: "",
});

// Form alamat
const addressForm = ref({
  recipient: "",
  phone: "",
  province: "",
  city: "",
  district: "",
  postalCode: "",
  detail: "",
});

// Status popup
const showSuccessModal = ref(false);

// Metode pembayaran sementara
const paymentMethod = ref("COD");

const selectedSubtotal = computed(() => {
  if (!selectedProduct.value) return 0;

  return selectedProduct.value.price * selectedQuantity.value;
});

const filteredProducts = computed(() => {
  return products.value.filter((product) => {
    const matchSearch =
      product.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      (product.variant || "")
        .toLowerCase()
        .includes(searchQuery.value.toLowerCase());

    const matchCategory =
      activeCategory.value === "Best Seller"
        ? product.bestseller
        : product.category === activeCategory.value;

    return matchSearch && matchCategory;
  });
});

const wishlistProducts = computed(() => {
  return products.value.filter((product) =>
    favorites.value.includes(product.id),
  );
});

const cartItemCount = computed(() => {
  return cart.value.reduce((total, item) => total + item.quantity, 0);
});

const cartProducts = computed(() => {
  return cart.value;
});

const cartTotal = computed(() => {
  return cart.value.reduce((total, product) => {
    return total + product.price * product.quantity;
  }, 0);
});

const formatPrice = (price) => {
  return new Intl.NumberFormat("id-ID", {
    style: "currency",
    currency: "IDR",
    maximumFractionDigits: 0,
  }).format(price);
};

const checkoutProducts = computed(() => {
  if (directCheckoutProduct.value) {
    return [directCheckoutProduct.value];
  }

  return cart.value;
});

const checkoutTotal = computed(() => {
  return checkoutProducts.value.reduce((total, item) => {
    return total + item.price * item.quantity;
  }, 0);
});

const checkoutItemCount = computed(() => {
  return checkoutProducts.value.reduce((total, item) => {
    return total + item.quantity;
  }, 0);
});

function orderProduct(product) {
  if (!isLoggedIn.value) {
    showLogin.value = true;
    return;
  }

  selectedProduct.value = product;
  selectedQuantity.value = 1;
  showProductDetail.value = true;
}

function increaseSelectedQuantity() {
  if (!selectedProduct.value) return;

  if (selectedQuantity.value < selectedProduct.value.stock) {
    selectedQuantity.value++;
  }
}

function decreaseSelectedQuantity() {
  if (selectedQuantity.value > 1) {
    selectedQuantity.value--;
  }
}

function closeProductDetail() {
  showProductDetail.value = false;
  selectedProduct.value = null;
  selectedQuantity.value = 1;
}

function checkoutSelectedProduct() {
  if (!selectedProduct.value) return;

  const product = selectedProduct.value;

  if (product.stock < 1) {
    alert("Maaf, stok produk sedang habis!");
    return;
  }

  if (selectedQuantity.value > product.stock) {
    alert("Jumlah produk melebihi stok yang tersedia!");
    return;
  }

  directCheckoutProduct.value = {
    ...product,
    quantity: selectedQuantity.value,
  };

  closeProductDetail();
  startCheckout();
}

function toggleFavorite(id) {
  if (!isLoggedIn.value) {
    showLogin.value = true;
    return;
  }

  if (favorites.value.includes(id)) {
    favorites.value = favorites.value.filter((item) => item !== id);
  } else {
    favorites.value.push(id);
  }
}

function startCheckout() {
  // Checkout langsung atau checkout dari keranjang
  if (!directCheckoutProduct.value && cart.value.length === 0) {
    alert("Keranjang kamu masih kosong!");
    return;
  }

  if (!userAddress.value.recipient) {
    openAddressForm();
    return;
  }

  activePage.value = "checkout";
}

function addToCart(product) {
  if (!isLoggedIn.value) {
    showLogin.value = true;
    return;
  }

  if (product.stock < 1) {
    alert("Maaf, stok produk sedang habis!");
    return;
  }

  const existingProduct = cart.value.find((item) => item.id === product.id);

  if (existingProduct) {
    if (existingProduct.quantity >= product.stock) {
      alert("Jumlah produk sudah mencapai batas stok!");
      return;
    }

    existingProduct.quantity++;
  } else {
    cart.value.push({
      ...product,
      quantity: 1,
    });
  }

  alert("Produk berhasil ditambahkan ke keranjang!");
}

function increaseQuantity(productId) {
  const index = cart.value.findIndex((item) => item.id === productId);

  if (index !== -1) {
    const item = cart.value[index];

    if (item.quantity >= item.stock) {
      alert("Jumlah produk sudah mencapai batas stok!");
      return;
    }

    cart.value[index] = {
      ...item,
      quantity: item.quantity + 1,
    };
  }
}

function decreaseQuantity(productId) {
  const index = cart.value.findIndex((item) => item.id === productId);

  if (index !== -1) {
    const currentQuantity = cart.value[index].quantity;

    if (currentQuantity > 1) {
      cart.value[index] = {
        ...cart.value[index],
        quantity: currentQuantity - 1,
      };
    }
  }
}

function login() {
  if (!username.value || !password.value) {
    alert("Silakan isi username/email dan password.");
    return;
  }

  // Simulasi login frontend
  isLoggedIn.value = true;
  showLogin.value = false;
  username.value = "";
  password.value = "";
}

function openAddressForm() {
  addressForm.value = { ...userAddress.value };
  activePage.value = "address";
}

function cancelCheckout() {
  directCheckoutProduct.value = null;
  activePage.value = "home";
}

function backToPreviousPage() {
  if (directCheckoutProduct.value) {
    cancelCheckout();
  } else {
    activePage.value = "cart";
  }
}

function saveAddress() {
  const address = addressForm.value;

  if (
    !address.recipient.trim() ||
    !address.phone.trim() ||
    !address.province.trim() ||
    !address.city.trim() ||
    !address.district.trim() ||
    !address.postalCode.trim() ||
    !address.detail.trim()
  ) {
    alert("Mohon lengkapi seluruh data alamat!");
    return;
  }

  userAddress.value = { ...address };
  activePage.value = "checkout";
}

function placeOrder() {
  if (!userAddress.value.recipient) {
    alert("Silakan isi alamat pengiriman terlebih dahulu!");
    activePage.value = "address";
    return;
  }

  if (checkoutProducts.value.length === 0) {
    alert("Tidak ada produk yang akan dipesan!");
    return;
  }

  // Validasi stok sebelum pesanan dibuat
  for (const item of checkoutProducts.value) {
    const latestProduct = products.value.find(
      (product) => product.id === item.id,
    );

    if (!latestProduct || item.quantity > latestProduct.stock) {
      alert(`Stok ${item.name} tidak mencukupi!`);
      return;
    }
  }

  const newOrder = {
    id: "GS-" + Date.now(),
    date: new Date().toLocaleString("id-ID"),
    items: checkoutProducts.value.map((product) => ({
      ...product,
    })),
    address: { ...userAddress.value },
    paymentMethod: paymentMethod.value,
    total: checkoutTotal.value,
    status: "Pesanan Dibuat",
  };

  orders.value.unshift(newOrder);

  showSuccessModal.value = true;
}

function finishOrder() {
  // Hanya kosongkan Cart jika checkout berasal dari Cart
  if (!directCheckoutProduct.value) {
    cart.value = [];
  }

  directCheckoutProduct.value = null;
  showSuccessModal.value = false;
  activePage.value = "home";
}

function register() {
  if (
    !fullName.value ||
    !email.value ||
    !registerUsername.value ||
    !registerPassword.value ||
    !confirmPassword.value
  ) {
    alert("Mohon lengkapi semua data pendaftaran.");
    return;
  }

  if (registerPassword.value !== confirmPassword.value) {
    alert("Konfirmasi password tidak sesuai.");
    return;
  }

  alert("Pendaftaran berhasil! Silakan login.");

  showRegister.value = false;
  showLogin.value = true;

  fullName.value = "";
  email.value = "";
  registerUsername.value = "";
  registerPassword.value = "";
  confirmPassword.value = "";
}

function logout() {
  isLoggedIn.value = false;
  cart.value = [];
  favorites.value = [];
  activePage.value = "home";
}

function scrollProducts(direction) {
  const container = document.querySelector(".product-list");

  if (container) {
    container.scrollBy({
      left: direction * 350,
      behavior: "smooth",
    });
  }
}
</script>

<template>
  <div class="store">
    <!-- NAVBAR -->
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

          <input v-model="searchQuery" type="text" placeholder="Search" />
        </div>

        <nav class="category-nav">
          <button
            v-for="category in categories"
            :key="category"
            :class="{ active: activeCategory === category }"
            @click="activeCategory = category"
          >
            {{ category }}
          </button>
        </nav>
      </div>

      <!-- GUEST MENU -->
      <div v-if="!isLoggedIn" class="guest-actions">
        <button class="register-btn" @click="showRegister = true">
          ▣ Daftar
        </button>

        <span class="slash">/</span>

        <button class="login-btn" @click="showLogin = true">➜ Login</button>
      </div>

      <!-- USER MENU -->
      <div v-else class="user-actions">
        <div class="user-info">
          <div class="user-avatar">
            {{ (fullName || username || "U").charAt(0).toUpperCase() }}
          </div>
          <span>{{ fullName || username || "User" }}</span>
        </div>

        <button class="user-btn" @click="activePage = 'wishlist'">
          ♡ Wishlist ({{ favorites.length }})
        </button>

        <button class="user-btn" @click="activePage = 'orders'">
          📦Orders
        </button>

        <!-- <button class="user-btn" @click="activePage = 'cart'">
          🛒 Cart ({{ cart.length }})
        </button> -->

        <button class="user-btn" @click="activePage = 'cart'">
          🛒 Cart ({{ cartItemCount }})
        </button>

        <button class="logout-btn" @click="logout">Logout</button>
      </div>
    </header>

    <!-- MAIN CONTENT -->
    <main v-if="activePage === 'home'" class="main-content">
      <!-- HERO BANNER -->
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

      <!-- PRODUCT SECTION -->
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
            @toggle-favorite="toggleFavorite"
            @add-to-cart="addToCart"
            @order="orderProduct"
          />
        </div>

        <div v-else class="empty-products">
          <h3>Produk tidak ditemukan</h3>
          <p>Coba gunakan kata kunci atau kategori lainnya.</p>
        </div>
      </section>
    </main>

    <!-- WISHLIST PAGE -->
    <main v-if="activePage === 'wishlist'" class="main-content wishlist-page">
      <div class="wishlist-heading">
        <div>
          <h2>My Wishlist</h2>
          <p>Produk pilihan yang kamu sukai di Gan's Store.</p>
        </div>

        <button class="back-home-btn" @click="activePage = 'home'">
          ← Kembali Belanja
        </button>
      </div>

      <!-- Jika Wishlist memiliki produk -->
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
              @click="toggleFavorite(product.id)"
            >
              ♥
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

            <div class="wishlist-actions">
              <button class="wishlist-cart-btn" @click="addToCart(product)">
                🛒 Tambah ke Keranjang
              </button>

              <button
                class="wishlist-remove-btn"
                @click="toggleFavorite(product.id)"
                title="Hapus produk"
              >
                Hapus
              </button>
            </div>
          </div>
        </article>
      </div>

      <!-- Jika Wishlist kosong -->
      <div v-else class="wishlist-empty">
        <div class="wishlist-empty-icon">♡</div>

        <h3>Wishlist Kamu Masih Kosong</h3>

        <p>Belum ada produk yang kamu simpan. Yuk, temukan produk favoritmu!</p>

        <button class="back-home-btn" @click="activePage = 'home'">
          Mulai Belanja
        </button>
      </div>
    </main>

    <!-- CART PAGE -->
    <main v-if="activePage === 'cart'" class="main-content cart-page">
      <div class="cart-heading">
        <div>
          <h2>Keranjang Belanja</h2>
          <p>Periksa kembali produk yang ingin kamu beli.</p>
        </div>

        <button class="back-home-btn" @click="activePage = 'home'">
          ← Kembali Belanja
        </button>
      </div>

      <div v-if="cartProducts.length" class="cart-layout">
        <!-- DAFTAR PRODUK -->
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
                  <button type="button" @click="decreaseQuantity(product.id)">
                    −
                  </button>

                  <span>{{ product.quantity }}</span>

                  <button
                    type="button"
                    @click="increaseQuantity(product.id)"
                    :disabled="product.quantity >= product.stock"
                  >
                    +
                  </button>
                </div>
              </div>
            </div>

            <div class="cart-item-subtotal">
              {{ formatPrice(product.price * product.quantity) }}
            </div>
          </article>
        </section>

        <!-- RINGKASAN BELANJA -->
        <aside class="cart-summary">
          <h3>Ringkasan Belanja</h3>

          <div class="summary-row">
            <span>Total Produk</span>
            <span
              >{{
                cartProducts.reduce((total, item) => total + item.quantity, 0)
              }}
              item</span
            >
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

          <button class="checkout-btn" @click="startCheckout">
            Lanjut ke Checkout
          </button>
        </aside>
      </div>

      <!-- KERANJANG KOSONG -->
      <div v-else class="cart-empty">
        <div class="cart-empty-icon">🛒</div>
        <h3>Keranjang Kamu Masih Kosong</h3>
        <p>
          Sepertinya kamu belum menambahkan produk apa pun. Yuk, temukan produk
          menarik di Gan's Store!
        </p>

        <button class="back-home-btn" @click="activePage = 'home'">
          Mulai Belanja
        </button>
      </div>
    </main>

    <!-- ADDRESS PAGE -->
    <main v-if="activePage === 'address'" class="main-content address-page">
      <div class="checkout-heading">
        <div>
          <h2>Alamat Pengiriman</h2>
          <p>Lengkapi alamat agar pesananmu dapat dikirim dengan benar.</p>
        </div>

        <button class="back-home-btn" @click="activePage = 'cart'">
          ← Kembali
        </button>
      </div>

      <section class="address-form-card">
        <h3>Informasi Penerima</h3>

        <form @submit.prevent="saveAddress">
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
              @click="cancelCheckout"
            >
              Batal
            </button>

            <button type="submit" class="address-save-btn">
              Simpan Alamat
            </button>
          </div>
        </form>
      </section>
    </main>

    <!-- CHECKOUT PAGE -->
    <main v-if="activePage === 'checkout'" class="main-content checkout-page">
      <div class="checkout-heading">
        <div>
          <h2>Checkout</h2>
          <p>Periksa kembali pesananmu sebelum melanjutkan.</p>
        </div>

        <button class="back-home-btn" @click="backToPreviousPage">
          ← Kembali
        </button>
      </div>

      <div class="checkout-layout">
        <section class="checkout-left">
          <!-- ALAMAT -->
          <div class="checkout-card">
            <div class="checkout-card-heading">
              <h3>Alamat Pengiriman</h3>
              <button @click="openAddressForm">Ubah Alamat</button>
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

          <!-- PRODUK -->
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

          <!-- PEMBAYARAN -->
          <div class="checkout-card">
            <h3>Metode Pembayaran</h3>

            <label class="payment-option">
              <input v-model="paymentMethod" type="radio" value="COD" />
              <span>
                <strong>Cash on Delivery (COD)</strong>
                <small>Bayar ketika pesanan diterima</small>
              </span>
            </label>

            <label class="payment-option">
              <input
                v-model="paymentMethod"
                type="radio"
                value="Transfer Bank"
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

        <!-- RINGKASAN -->
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

          <button class="checkout-btn" @click="placeOrder">Buat Pesanan</button>

          <p class="checkout-note">
            Pastikan alamat dan produk pesanan sudah benar.
          </p>
        </aside>
      </div>
    </main>

    <!-- ORDER HISTORY PAGE -->
    <main v-if="activePage === 'orders'" class="main-content orders-page">
      <div class="orders-heading">
        <div>
          <h2>Pesanan Saya</h2>
          <p>Lihat dan pantau semua pesananmu di Gan's Store.</p>
        </div>

        <button class="back-home-btn" @click="activePage = 'home'">
          ← Kembali Belanja
        </button>
      </div>

      <!-- JIKA ADA PESANAN -->
      <div v-if="orders.length" class="orders-list">
        <article v-for="order in orders" :key="order.id" class="order-card">
          <div class="order-card-header">
            <div>
              <span class="order-label">Nomor Pesanan</span>
              <strong>{{ order.id }}</strong>
            </div>

            <span class="order-status">
              {{ order.status }}
            </span>
          </div>

          <div class="order-date">
            {{ order.date }}
          </div>

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
            <p>
              {{ order.address.recipient }} ·
              {{ order.address.phone }}
            </p>
            <p>
              {{ order.address.detail }}, {{ order.address.district }},
              {{ order.address.city }}, {{ order.address.province }},
              {{ order.address.postalCode }}
            </p>
          </div>
        </article>
      </div>

      <!-- JIKA BELUM ADA PESANAN -->
      <div v-else class="orders-empty">
        <div class="orders-empty-icon">📦</div>

        <h3>Belum Ada Pesanan</h3>

        <p>
          Kamu belum memiliki riwayat pesanan. Yuk, temukan produk menarik di
          Gan's Store!
        </p>

        <button class="back-home-btn" @click="activePage = 'home'">
          Mulai Belanja
        </button>
      </div>
    </main>

    <!-- SUCCESS MODAL -->
    <div v-if="showSuccessModal" class="success-overlay">
      <div class="success-modal">
        <div class="success-icon">✓</div>

        <h2>Pesanan Berhasil!</h2>

        <p>
          Terima kasih sudah berbelanja di Gan's Store. Pesananmu berhasil
          dibuat.
        </p>

        <div class="success-order-info">
          <span>Metode Pembayaran</span>
          <strong>{{ paymentMethod }}</strong>
        </div>

        <button class="success-done-btn" @click="finishOrder">
          Kembali ke Beranda
        </button>
      </div>
    </div>

    <!-- PRODUCT DETAIL MODAL -->
    <ProductDetailModal
      :show="showProductDetail"
      :product="selectedProduct"
      :quantity="selectedQuantity"
      :subtotal="selectedSubtotal"
      :format-price="formatPrice"
      @close="closeProductDetail"
      @increase="increaseSelectedQuantity"
      @decrease="decreaseSelectedQuantity"
      @checkout="checkoutSelectedProduct"
    />
  </div>

  <!-- LOGIN MODAL -->
  <div v-if="showLogin" class="modal-overlay" @click.self="showLogin = false">
    <div class="auth-modal">
      <button class="close-modal" @click="showLogin = false">×</button>

      <div class="auth-logo">
        <span>gan's</span>
        <div class="auth-circle">G'S</div>
        <span>store</span>
      </div>

      <h2>Login</h2>
      <p class="auth-subtitle">Silakan login untuk melanjutkan</p>

      <form @submit.prevent="login">
        <label>Email / Username</label>
        <input
          v-model="username"
          type="text"
          placeholder="Masukkan email/username"
        />

        <label>Password</label>
        <input
          v-model="password"
          type="password"
          placeholder="Masukkan password"
        />

        <button class="auth-submit" type="submit">Login</button>
      </form>

      <p class="switch-auth">
        Belum punya akun?
        <button
          @click="
            showLogin = false;
            showRegister = true;
          "
        >
          Daftar
        </button>
      </p>
    </div>
  </div>

  <!-- REGISTER MODAL -->
  <div
    v-if="showRegister"
    class="modal-overlay"
    @click.self="showRegister = false"
  >
    <div class="auth-modal register-modal">
      <button class="close-modal" @click="showRegister = false">×</button>

      <div class="auth-logo">
        <span>gan's</span>
        <div class="auth-circle">G'S</div>
        <span>store</span>
      </div>

      <h2>Register</h2>
      <p class="auth-subtitle">Buat akun baru Gan's Store</p>

      <form @submit.prevent="register">
        <label>Nama Lengkap</label>
        <input
          v-model="fullName"
          type="text"
          placeholder="Masukkan nama lengkap"
        />

        <label>Username</label>
        <input
          v-model="registerUsername"
          type="text"
          placeholder="Buat username"
        />

        <label>Email</label>
        <input v-model="email" type="email" placeholder="Masukkan email" />

        <label>Password</label>
        <input
          v-model="registerPassword"
          type="password"
          placeholder="Buat password"
        />

        <!-- <label>Konfirmasi Password</label>
          <input
            v-model="confirmPassword"
            type="password"
            placeholder="Ulangi password"
          /> -->

        <button class="auth-submit" type="submit">Daftar Sekarang</button>
      </form>

      <p class="switch-auth">
        Sudah punya akun?
        <button
          @click="
            showRegister = false;
            showLogin = true;
          "
        >
          Login
        </button>
      </p>
    </div>
  </div>
</template>
