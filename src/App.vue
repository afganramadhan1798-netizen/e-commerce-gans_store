<script setup>
import { ref, computed } from "vue";
import { products as productData, categories } from "./data/products";
import ProductCard from "./components/product/ProductCard.vue";
import ProductDetailModal from "./components/product/ProductDetailModal.vue";
import StoreHeader from "./components/StoreHeader.vue";
import HomePage from "./components/page/HomePage.vue";
import WishlistPage from "./components/page/WishlistPage.vue";
import CartPage from "./components/page/CartPage.vue";
import AddressPage from "./components/page/AddressPage.vue";
import CheckoutPage from "./components/page/CheckoutPage.vue";
import OrdersPage from "./components/page/OrdersPage.vue";
import AdminDashboard from "./components/admin/AdminDashboard.vue";

const isLoggedIn = ref(false);
const currentUserRole = ref(null);
const currentUserId = ref(null);
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
const users = ref([
  {
    id: "admin-001",
    name: "Admin Gan's Store",
    username: "admin",
    email: "admin@gansstore.test",
    role: "admin",
    status: "Aktif",
    joinedAt: "Admin",
  },
  {
    id: "customer-001",
    name: "Alya Putri",
    username: "alya",
    email: "alya@example.test",
    role: "customer",
    status: "Aktif",
    joinedAt: "Demo",
  },
  {
    id: "customer-002",
    name: "Bima Pratama",
    username: "bima",
    email: "bima@example.test",
    role: "customer",
    status: "Aktif",
    joinedAt: "Demo",
  },
]);

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

const adminUsers = computed(() =>
  users.value.map((user) => ({
    ...user,
    isCurrentUser: user.id === currentUserId.value,
  })),
);

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
  const identifier = username.value.trim().toLowerCase();

  if (!identifier || !password.value) {
    alert("Silakan isi username/email dan password.");
    return;
  }

  let account = users.value.find(
    (user) =>
      user.username.toLowerCase() === identifier ||
      user.email.toLowerCase() === identifier,
  );

  if (!account) {
    const newUsername = identifier.includes("@")
      ? identifier.split("@")[0]
      : identifier;
    account = {
      id: `customer-${Date.now()}`,
      name: newUsername,
      username: newUsername,
      email: identifier.includes("@") ? identifier : `${identifier}@demo.test`,
      role: "customer",
      status: "Aktif",
      joinedAt: "Baru",
    };
    users.value.push(account);
  }

  if (account.status !== "Aktif") {
    alert("Akun ini sedang dinonaktifkan. Hubungi admin toko.");
    return;
  }

  if (account.role === "admin" && password.value !== "admin123") {
    alert("Password demo admin tidak sesuai.");
    return;
  }

  isLoggedIn.value = true;
  currentUserId.value = account.id;
  currentUserRole.value = account.role;
  fullName.value = account.name;
  activePage.value = account.role === "admin" ? "admin" : "home";
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
    customerName: fullName.value || currentUserId.value,
    customerEmail:
      users.value.find((user) => user.id === currentUserId.value)?.email || "",
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

  const duplicateUser = users.value.some(
    (user) =>
      user.username.toLowerCase() === registerUsername.value.toLowerCase() ||
      user.email.toLowerCase() === email.value.toLowerCase(),
  );

  if (duplicateUser) {
    alert("Username atau email sudah terdaftar.");
    return;
  }

  users.value.push({
    id: `customer-${Date.now()}`,
    name: fullName.value.trim(),
    username: registerUsername.value.trim(),
    email: email.value.trim().toLowerCase(),
    role: "customer",
    status: "Aktif",
    joinedAt: "Baru",
  });

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
  currentUserRole.value = null;
  currentUserId.value = null;
  fullName.value = "";
  cart.value = [];
  favorites.value = [];
  activePage.value = "home";
}

function createProduct(product) {
  const nextId = Math.max(0, ...products.value.map((item) => item.id)) + 1;
  products.value.push({ ...product, id: nextId });
}

function updateProduct(updatedProduct) {
  products.value = products.value.map((product) =>
    product.id === updatedProduct.id
      ? { ...updatedProduct }
      : { ...product, stock: product.stock ?? 10 },
  );
}

function deleteProduct(productId) {
  products.value = products.value.filter((product) => product.id !== productId);
  cart.value = cart.value.filter((product) => product.id !== productId);
  favorites.value = favorites.value.filter((id) => id !== productId);
}

function updateUserRole({ userId, role }) {
  const user = users.value.find((item) => item.id === userId);
  if (!user || user.id === currentUserId.value) return;
  user.role = role;
}

function toggleUserStatus(userId) {
  const user = users.value.find((item) => item.id === userId);
  if (!user || user.id === currentUserId.value) return;
  user.status = user.status === "Aktif" ? "Nonaktif" : "Aktif";
}

function updateOrderStatus({ orderId, status }) {
  const order = orders.value.find((item) => item.id === orderId);
  if (order) order.status = status;
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
    <StoreHeader
      :categories="categories"
      :search-query="searchQuery"
      :active-category="activeCategory"
      :is-logged-in="isLoggedIn"
      :user-role="currentUserRole"
      :full-name="fullName"
      :username="username"
      :favorite-count="favorites.length"
      :cart-item-count="cartItemCount"
      @update:search-query="searchQuery = $event"
      @update:active-category="activeCategory = $event"
      @show-login="showLogin = true"
      @show-register="showRegister = true"
      @navigate="activePage = $event"
      @logout="logout"
    />

    <AdminDashboard
      v-if="activePage === 'admin' && currentUserRole === 'admin'"
      :products="products"
      :users="adminUsers"
      :orders="orders"
      :categories="categories"
      :format-price="formatPrice"
      @home="activePage = 'home'"
      @create-product="createProduct"
      @update-product="updateProduct"
      @delete-product="deleteProduct"
      @update-user-role="updateUserRole"
      @toggle-user-status="toggleUserStatus"
      @update-order-status="updateOrderStatus"
    />

    <HomePage
      v-if="activePage === 'home'"
      :active-category="activeCategory"
      :filtered-products="filteredProducts"
      :favorites="favorites"
      :is-logged-in="isLoggedIn"
      :format-price="formatPrice"
      @toggle-favorite="toggleFavorite"
      @add-to-cart="addToCart"
      @order="orderProduct"
    />

    <WishlistPage
      v-if="activePage === 'wishlist'"
      :wishlist-products="wishlistProducts"
      :format-price="formatPrice"
      @home="activePage = 'home'"
      @toggle-favorite="toggleFavorite"
      @add-to-cart="addToCart"
    />

    <CartPage
      v-if="activePage === 'cart'"
      :cart-products="cartProducts"
      :cart-total="cartTotal"
      :format-price="formatPrice"
      @home="activePage = 'home'"
      @decrease="decreaseQuantity"
      @increase="increaseQuantity"
      @checkout="startCheckout"
    />

    <AddressPage
      v-if="activePage === 'address'"
      :address-form="addressForm"
      @back="activePage = 'cart'"
      @cancel="cancelCheckout"
      @save="saveAddress"
    />

    <CheckoutPage
      v-if="activePage === 'checkout'"
      :user-address="userAddress"
      :checkout-products="checkoutProducts"
      :checkout-item-count="checkoutItemCount"
      :checkout-total="checkoutTotal"
      :payment-method="paymentMethod"
      :format-price="formatPrice"
      @back="backToPreviousPage"
      @edit-address="openAddressForm"
      @update:payment-method="paymentMethod = $event"
      @place-order="placeOrder"
    />

    <OrdersPage
      v-if="activePage === 'orders'"
      :orders="orders"
      :format-price="formatPrice"
      @home="activePage = 'home'"
    />

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
      <p class="demo-admin-hint">
        Demo admin: <strong>admin</strong> / <strong>admin123</strong>
      </p>

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

        <label>Konfirmasi Password</label>
        <input
          v-model="confirmPassword"
          type="password"
          placeholder="Ulangi password"
        />

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
