<script setup>
import { ref, computed, onMounted } from "vue";
import api from "./services/api";
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
const authToken = ref("");
const showLogin = ref(false);
const showRegister = ref(false);

const username = ref("");
const password = ref("");
const fullName = ref("");
const email = ref("");
const registerUsername = ref("");
const registerPassword = ref("");
const confirmPassword = ref("");
const isAuthChecking = ref(true);

const searchQuery = ref("");
const activeCategory = ref("Best Seller");
const favorites = ref([]);
const cart = ref([]);
const activePage = ref("home");
const orders = ref([]);
const users = ref([]);

const showProductDetail = ref(false);
const selectedProduct = ref(null);
const selectedQuantity = ref(1);

const products = ref([]);
const adminProducts = ref([]);
const categories = ref([]);
const adminCategories = ref([]);
const isProductsLoading = ref(false);

async function fetchProducts() {
  isProductsLoading.value = true;

  try {
    const response = await api.get("/products");

    products.value = response.data.products.map((product) => ({
      ...product,
      category: product.category?.name || "",
      bestseller: Boolean(product.bestseller),
      stock: Number(product.stock),
      price: Number(product.price),
    }));
  } catch (error) {
    console.error("Fetch products error:", error);
    alert("Gagal mengambil data produk.");
  } finally {
    isProductsLoading.value = false;
  }
}

async function fetchAdminProducts() {
  if (currentUserRole.value !== "admin") return;

  try {
    const response = await api.get("/admin/products");

    adminProducts.value = response.data.products.map((product) => ({
      ...product,
      category: product.category?.name || "",
      bestseller: Boolean(product.bestseller),
      stock: Number(product.stock),
      price: Number(product.price),
    }));
  } catch (error) {
    console.error("Fetch admin products error:", error);

    alert(
      error.response?.data?.message || "Gagal mengambil data produk admin.",
    );
  }
}

async function fetchCategories() {
  try {
    const response = await api.get("/categories");

    categories.value = [
      "Best Seller",
      ...response.data.categories.map((category) => category.name),
    ];
  } catch (error) {
    console.error("Fetch categories error:", error);
  }
}

async function fetchAdminCategories() {
  if (currentUserRole.value !== "admin") return;

  try {
    const response = await api.get("/admin/categories");

    adminCategories.value = response.data.categories;
  } catch (error) {
    console.error("Fetch admin categories error:", error);
  }
}

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

async function fetchAdminUsers() {
  if (currentUserRole.value !== "admin") return;

  try {
    const response = await api.get("/admin/users");

    users.value = response.data.users.map((user) => ({
      id: user.id,
      name: user.name,
      username: user.username,
      email: user.email,
      role: user.role,
      status: user.is_active ? "Aktif" : "Nonaktif",
      joinedAt: user.created_at
        ? new Date(user.created_at).toLocaleDateString("id-ID", {
            day: "numeric",
            month: "short",
            year: "numeric",
          })
        : "-",
    }));
  } catch (error) {
    console.error("Fetch admin users error:", error);
  }
}

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

async function login() {
  const identifier = username.value.trim();

  if (!identifier || !password.value) {
    alert("Silakan isi username/email dan password.");
    return;
  }

  try {
    const response = await api.post("/login", {
      identifier: identifier,
      password: password.value,
    });

    const user = response.data.user;
    const token = response.data.token;

    // Simpan token autentikasi
    authToken.value = token;

    localStorage.setItem("gans_store_token", token);
    // Gunakan token untuk request berikutnya
    api.defaults.headers.common.Authorization = `Bearer ${token}`;

    // Simpan data user yang sedang login
    isLoggedIn.value = true;
    currentUserId.value = user.id;
    currentUserRole.value = user.role;
    fullName.value = user.name;

    // Masukkan user ke data lokal jika belum tersedia
    const existingUser = users.value.find((item) => item.id === user.id);

    if (!existingUser) {
      users.value.push({
        ...user,
        status: "Aktif",
        joinedAt: "Baru",
      });
    }

    // Arahkan berdasarkan role
    activePage.value = user.role === "admin" ? "admin" : "home";
    if (user.role === "admin") {
      await Promise.all([
        fetchAdminUsers(),
        fetchAdminProducts(),
        fetchAdminCategories(),
      ]);
    }
    showLogin.value = false;

    username.value = "";
    password.value = "";

    alert("Login berhasil! Selamat datang di Gan's Store.");
  } catch (error) {
    console.error("Login error:", error);

    const message =
      error.response?.data?.message ||
      "Login gagal. Periksa username/email dan password.";

    alert(message);
  }
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

async function register() {
  if (
    !fullName.value.trim() ||
    !email.value.trim() ||
    !registerUsername.value.trim() ||
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

  try {
    await api.post("/register", {
      name: fullName.value.trim(),
      username: registerUsername.value.trim(),
      email: email.value.trim().toLowerCase(),
      password: registerPassword.value,
      password_confirmation: confirmPassword.value,
    });

    alert("Pendaftaran berhasil! Silakan login.");

    showRegister.value = false;
    showLogin.value = true;

    // Kosongkan form registrasi
    fullName.value = "";
    email.value = "";
    registerUsername.value = "";
    registerPassword.value = "";
    confirmPassword.value = "";
  } catch (error) {
    console.error("Register error:", error);

    const errors = error.response?.data?.errors;

    if (errors) {
      const firstError = Object.values(errors).flat()[0];
      alert(firstError);
    } else {
      alert(
        error.response?.data?.message ||
          "Pendaftaran gagal. Silakan coba lagi.",
      );
    }
  }
}

async function logout() {
  try {
    if (authToken.value) {
      await api.post("/logout");
    }
  } catch (error) {
    console.error("Logout error:", error);
  } finally {
    // Hapus token autentikasi
    authToken.value = "";
    localStorage.removeItem("gans_store_token");
    delete api.defaults.headers.common.Authorization;

    // Reset data login
    isLoggedIn.value = false;
    currentUserRole.value = null;
    currentUserId.value = null;
    fullName.value = "";

    // Reset data sementara pengguna
    cart.value = [];
    favorites.value = [];

    activePage.value = "home";

    alert("Kamu berhasil logout.");
  }
}

async function restoreAuth() {
  const savedToken = localStorage.getItem("gans_store_token");

  if (!savedToken) {
    isAuthChecking.value = false;
    return;
  }

  try {
    authToken.value = savedToken;

    api.defaults.headers.common.Authorization = `Bearer ${savedToken}`;

    const response = await api.get("/user");
    const user = response.data;

    isLoggedIn.value = true;
    currentUserId.value = user.id;
    currentUserRole.value = user.role;
    fullName.value = user.name;

    activePage.value = user.role === "admin" ? "admin" : "home";

    // Ambil ulang data pengguna jika yang login adalah admin
    if (user.role === "admin") {
      await Promise.all([
        fetchAdminUsers(),
        fetchAdminProducts(),
        fetchAdminCategories(),
      ]);
    }
  } catch (error) {
    console.error("Restore authentication error:", error);

    authToken.value = "";
    localStorage.removeItem("gans_store_token");

    delete api.defaults.headers.common.Authorization;

    isLoggedIn.value = false;
    currentUserRole.value = null;
    currentUserId.value = null;
    fullName.value = "";
  } finally {
    isAuthChecking.value = false;
  }
}

onMounted(async () => {
  await Promise.all([fetchProducts(), fetchCategories()]);

  await restoreAuth();
});

function createProduct(product) {
  saveProductToBackend(product);
}

function updateProduct(product) {
  saveProductToBackend(product, true);
}

async function saveProductToBackend(product, isUpdate = false) {
  try {
    const selectedCategory = adminCategories.value.find(
      (category) => category.name === product.category,
    );

    if (!selectedCategory) {
      alert("Kategori produk tidak ditemukan.");
      return;
    }

    const formData = new FormData();

    formData.append("name", product.name);
    formData.append("variant", product.variant || "");
    formData.append("category_id", selectedCategory.id);
    formData.append("price", product.price);
    formData.append("stock", product.stock);
    formData.append("bestseller", product.bestseller ? "1" : "0");
    formData.append("description", product.description || "");

    if (product.imageFile) {
      console.log("Detail file gambar:", {
        file: product.imageFile,
        name: product.imageFile.name,
        type: product.imageFile.type,
        size: product.imageFile.size,
        isFile: product.imageFile instanceof File,
      });

      formData.append("image", product.imageFile);
    }

    console.log("FormData image:", formData.get("image"));

    if (isUpdate) {
      formData.append("_method", "PUT");

      await api.post(`/admin/products/${product.id}`, formData);
    } else {
      await api.post("/admin/products", formData);
    }

    await Promise.all([fetchAdminProducts(), fetchProducts()]);

    alert(
      isUpdate ? "Produk berhasil diperbarui." : "Produk berhasil ditambahkan.",
    );
  } catch (error) {
    console.error("Save product error:", error);

    const errors = error.response?.data?.errors;

    if (errors) {
      alert(Object.values(errors).flat()[0]);
    } else {
      alert(error.response?.data?.message || "Gagal menyimpan produk.");
    }
  }
}

async function deleteProduct(productId) {
  if (!window.confirm("Yakin ingin menghapus produk ini?")) {
    return;
  }

  try {
    await api.delete(`/admin/products/${productId}`);

    await Promise.all([fetchAdminProducts(), fetchProducts()]);

    cart.value = cart.value.filter((product) => product.id !== productId);

    favorites.value = favorites.value.filter((id) => id !== productId);

    alert("Produk berhasil dihapus.");
  } catch (error) {
    console.error("Delete product error:", error);

    alert(error.response?.data?.message || "Gagal menghapus produk.");
  }
}

async function updateUserRole({ userId, role }) {
  const user = users.value.find((item) => item.id === userId);

  if (!user || user.id === currentUserId.value) return;

  try {
    const response = await api.patch(`/admin/users/${userId}/role`, { role });

    user.role = response.data.user.role;

    alert("Role pengguna berhasil diperbarui.");
  } catch (error) {
    console.error("Update user role error:", error);

    alert(error.response?.data?.message || "Gagal memperbarui role pengguna.");
  }
}

async function toggleUserStatus(userId) {
  const user = users.value.find((item) => item.id === userId);

  if (!user || user.id === currentUserId.value) return;

  const newStatus = user.status !== "Aktif";

  try {
    const response = await api.patch(`/admin/users/${userId}/status`, {
      is_active: newStatus,
    });

    user.status = response.data.user.is_active ? "Aktif" : "Nonaktif";

    alert("Status pengguna berhasil diperbarui.");
  } catch (error) {
    console.error("Toggle user status error:", error);

    alert(
      error.response?.data?.message || "Gagal memperbarui status pengguna.",
    );
  }
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
  <!-- AUTHENTICATION LOADING -->
  <div v-if="isAuthChecking" class="auth-loading">
    <div class="auth-loading-content">
      <div class="auth-spinner"></div>
      <p>Memeriksa sesi akun...</p>
    </div>
  </div>

  <div v-else class="store">
    <StoreHeader
      v-if="activePage !== 'admin'"
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
      :products="adminProducts"
      :users="adminUsers"
      :orders="orders"
      :categories="categories"
      :format-price="formatPrice"
      :user-name="fullName || username"
      @home="activePage = 'home'"
      @logout="logout"
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
