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
import AppPopup from "./components/AppPopup.vue";

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
const wishlistRecords = ref([]);
const cart = ref([]);
const activePage = ref("home");
const addressReturnPage = ref("cart");
const addressWasMissing = ref(false);
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

// Custom popup
const showAppPopup = ref(false);
const popupType = ref("info");
const popupTitle = ref("Informasi");
const popupMessage = ref("");
const popupConfirmText = ref("Mengerti");
const popupCancelText = ref("Batal");
const popupShowCancel = ref(false);
const popupResolver = ref(null);

function showAlert(message, type = "info", title = "") {
  popupType.value = type;
  popupMessage.value = String(message);
  popupShowCancel.value = false;
  popupConfirmText.value = "Mengerti";

  const defaultTitles = {
    success: "Berhasil!",
    error: "Terjadi Kesalahan",
    warning: "Perhatian",
    info: "Informasi",
  };

  popupTitle.value = title || defaultTitles[type] || "Informasi";

  showAppPopup.value = true;
}

function showConfirm(message, title = "Konfirmasi") {
  return new Promise((resolve) => {
    popupType.value = "confirm";
    popupTitle.value = title;
    popupMessage.value = message;
    popupConfirmText.value = "Ya, Lanjutkan";
    popupCancelText.value = "Batal";
    popupShowCancel.value = true;

    popupResolver.value = resolve;
    showAppPopup.value = true;
  });
}

function handlePopupConfirm() {
  showAppPopup.value = false;

  if (popupResolver.value) {
    popupResolver.value(true);
    popupResolver.value = null;
  }
}

function handlePopupCancel() {
  showAppPopup.value = false;

  if (popupResolver.value) {
    popupResolver.value(false);
    popupResolver.value = null;
  }
}

function alert(message) {
  showAlert(message, "info");
}

function mapOrderFromApi(order) {
  return {
    id: order.id,
    orderNumber: order.order_number,
    date: order.created_at,

    customerName: order.user?.name || order.user?.username || order.recipient,
    customerEmail: order.user?.email || "",

    items: (order.items || []).map((item) => ({
      id: item.product_id ?? item.id,
      productId: item.product_id,
      name: item.product_name,
      variant: item.variant,
      image: item.image_url,
      price: Number(item.price),
      quantity: Number(item.quantity),
      subtotal: Number(item.subtotal),
    })),

    address: {
      recipient: order.recipient,
      phone: order.phone,
      province: order.province,
      city: order.city,
      district: order.district,
      postalCode: order.postal_code,
      detail: order.address_detail,
    },

    paymentMethod: order.payment_method,
    subtotal: Number(order.subtotal || 0),
    shippingCost: Number(order.shipping_cost || 0),
    total: Number(order.total),
    status: order.status,
  };
}

function mapCartFromApi(cartData) {
  if (!cartData || !cartData.items) return [];

  return cartData.items
    .filter((item) => item.product)
    .map((item) => {
      const product = item.product;

      const latestProduct = products.value.find(
        (p) => Number(p.id) === Number(product.id),
      );

      const imagePath =
        product.image_url || product.image || latestProduct?.image || "";

      return {
        id: product.id,
        productId: product.id,
        cartItemId: item.id,

        name: product.name,
        variant: product.variant || "",

        image: imagePath
          ? imagePath.startsWith("http")
            ? imagePath
            : `http://127.0.0.1:8000/storage/${imagePath}`
          : "",

        price: Number(product.price),
        stock: Number(product.stock),
        quantity: Number(item.quantity),

        bestseller: Boolean(product.bestseller),
        category: product.category?.name || latestProduct?.category || "",
      };
    });
}

async function fetchOrders() {
  if (!isLoggedIn.value || currentUserRole.value === "admin") return;

  const requestedUserId = currentUserId.value;

  try {
    const response = await api.get("/orders");

    if (
      currentUserId.value !== requestedUserId ||
      currentUserRole.value === "admin"
    ) {
      return;
    }

    orders.value = (response.data.orders || []).map(mapOrderFromApi);
  } catch (error) {
    console.error(
      "Gagal mengambil pesanan customer:",
      error.response?.data || error,
    );
  }
}

async function fetchAdminOrders() {
  if (!isLoggedIn.value || currentUserRole.value !== "admin") return;

  try {
    const response = await api.get("/admin/orders");

    orders.value = (response.data.orders || []).map(mapOrderFromApi);

    console.log("Pesanan seluruh customer:", orders.value);
  } catch (error) {
    console.error(
      "Gagal mengambil pesanan customer untuk admin:",
      error.response?.data || error,
    );
  }
}

async function fetchCart() {
  if (!isLoggedIn.value || currentUserRole.value !== "customer") {
    return;
  }

  const requestedUserId = currentUserId.value;

  try {
    const response = await api.get("/cart");

    // Pastikan respons masih milik akun yang sedang login.
    if (
      currentUserId.value !== requestedUserId ||
      currentUserRole.value !== "customer"
    ) {
      return;
    }

    // Perbarui isi keranjang tanpa mengosongkan data sebelumnya.
    cart.value = mapCartFromApi(response.data.cart);
  } catch (error) {
    console.error("Gagal mengambil keranjang:", error.response?.data || error);
  }
}

async function fetchWishlist() {
  if (!isLoggedIn.value || currentUserRole.value !== "customer") {
    return;
  }

  const requestedUserId = currentUserId.value;

  try {
    const response = await api.get("/wishlist");

    // Pastikan respons masih milik akun yang sedang login.
    if (
      currentUserId.value !== requestedUserId ||
      currentUserRole.value !== "customer"
    ) {
      return;
    }

    wishlistRecords.value = response.data.wishlists || [];

    favorites.value = wishlistRecords.value.map((item) =>
      Number(item.product_id ?? item.product?.id),
    );
  } catch (error) {
    console.error("Gagal mengambil Wishlist:", error.response?.data || error);
  }
}

async function fetchAddress() {
  if (!isLoggedIn.value || currentUserRole.value !== "customer") {
    return;
  }

  const requestedUserId = currentUserId.value;

  try {
    const response = await api.get("/address");

    // Pastikan respons masih milik akun yang sedang login.
    if (
      currentUserId.value !== requestedUserId ||
      currentUserRole.value !== "customer"
    ) {
      return;
    }

    const address = response.data.address;

    if (address) {
      userAddress.value = {
        recipient: address.recipient || "",
        phone: address.phone || "",
        province: address.province || "",
        city: address.city || "",
        district: address.district || "",
        postalCode: address.postal_code || "",
        detail: address.address_detail || "",
      };

      addressForm.value = { ...userAddress.value };
    } else {
      userAddress.value = {
        recipient: "",
        phone: "",
        province: "",
        city: "",
        district: "",
        postalCode: "",
        detail: "",
      };

      addressForm.value = { ...userAddress.value };
    }
  } catch (error) {
    console.error(
      "Gagal mengambil alamat pengguna:",
      error.response?.data || error,
    );
  }
}

async function cancelOrder(orderId) {
  const confirmed = await showConfirm(
    "Apakah kamu yakin ingin membatalkan pesanan ini?",
    "Batalkan Pesanan?",
  );

  if (!confirmed) return;

  try {
    const response = await api.patch(`/orders/${orderId}/cancel`);

    alert(response.data.message || "Pesanan berhasil dibatalkan.");

    // Perbarui daftar pesanan.
    await fetchOrders();

    // Perbarui stok produk dari backend.
    await fetchProducts();
  } catch (error) {
    console.error("Gagal membatalkan pesanan:", error.response?.data || error);

    alert(
      error.response?.data?.message ||
        "Gagal membatalkan pesanan. Silakan coba lagi.",
    );
  }
}

async function fetchProducts() {
  isProductsLoading.value = true;

  try {
    const response = await api.get("/products");

    products.value = response.data.products.map((product) => ({
      ...product,

      // Samakan properti gambar untuk seluruh frontend
      image: product.image_url || product.image || "",

      category: product.category?.name || "",
      bestseller: Boolean(product.bestseller),
      stock: Number(product.stock),
      price: Number(product.price),
    }));

    return true;
  } catch (error) {
    console.error("Fetch products error:", error);
    alert("Gagal mengambil data produk.");

    return false;
  } finally {
    isProductsLoading.value = false;
  }
}

async function fetchAdminProducts() {
  if (currentUserRole.value !== "admin") return;

  try {
    const response = await api.get("/admin/products");

    adminProducts.value = (response.data.products || []).map((product) => ({
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

    adminCategories.value = response.data.categories || [];
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
    favorites.value.some(
      (favoriteId) => Number(favoriteId) === Number(product.id),
    ),
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

    users.value = (response.data.users || []).map((user) => ({
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

  const latestProduct = products.value.find(
    (item) => item.id === selectedProduct.value.id,
  );

  if (!latestProduct || latestProduct.stock < 1) {
    alert("Maaf, stok produk sedang habis!");
    return;
  }

  if (selectedQuantity.value > latestProduct.stock) {
    alert("Jumlah produk melebihi stok yang tersedia!");
    return;
  }

  directCheckoutProduct.value = {
    ...latestProduct,
    quantity: selectedQuantity.value,
  };

  closeProductDetail();
  startCheckout();
}

async function toggleFavorite(id) {
  if (!isLoggedIn.value) {
    showLogin.value = true;
    return;
  }

  if (currentUserRole.value !== "customer") {
    return;
  }

  const productId = Number(id);

  const existingWishlist = wishlistRecords.value.find(
    (item) => Number(item.product_id ?? item.product?.id) === productId,
  );

  try {
    if (existingWishlist) {
      // Hapus Wishlist dari database.
      await api.delete(`/wishlist/${existingWishlist.id}`);
    } else {
      // Tambahkan Wishlist ke database.
      await api.post("/wishlist", {
        product_id: productId,
      });
    }

    // Sinkronkan kembali data Wishlist dari backend.
    await fetchWishlist();
  } catch (error) {
    console.error("Gagal memperbarui Wishlist:", error.response?.data || error);

    alert(
      error.response?.data?.message ||
        "Gagal memperbarui Wishlist. Silakan coba lagi.",
    );
  }
}

async function startCheckout() {
  // Checkout langsung atau checkout dari keranjang
  if (!directCheckoutProduct.value && cart.value.length === 0) {
    alert("Keranjang kamu masih kosong!");
    return;
  }

  try {
    // Ambil data produk terbaru dari backend
    const isSuccess = await fetchProducts();

    if (!isSuccess) return;

    const checkoutItems = directCheckoutProduct.value
      ? [directCheckoutProduct.value]
      : cart.value;

    // Periksa stok setiap produk
    for (const item of checkoutItems) {
      const latestProduct = products.value.find(
        (product) => product.id === item.id,
      );

      if (!latestProduct || latestProduct.stock < 1) {
        alert(`Maaf, stok ${item.name} sedang habis!`);
        return;
      }

      if (item.quantity > latestProduct.stock) {
        alert(
          `Stok ${item.name} hanya tersedia ${latestProduct.stock} item. Silakan sesuaikan jumlahnya.`,
        );

        return;
      }

      // Sinkronkan stok dan data produk terbaru
      item.stock = latestProduct.stock;
      item.price = latestProduct.price;
      item.image = latestProduct.image;
    }

    if (!userAddress.value.recipient) {
      openAddressForm();
      return;
    }

    activePage.value = "checkout";
  } catch (error) {
    console.error("Checkout validation error:", error);
    alert("Gagal memeriksa stok produk. Silakan coba lagi.");
  }
}

async function addToCart(product) {
  if (!isLoggedIn.value) {
    showLogin.value = true;
    return;
  }

  const latestProduct = products.value.find(
    (item) => Number(item.id) === Number(product.id),
  );

  if (!latestProduct || latestProduct.stock < 1) {
    alert("Maaf, stok produk sedang habis!");
    return;
  }

  const existingProduct = cart.value.find(
    (item) => Number(item.id) === Number(product.id),
  );

  if (existingProduct && existingProduct.quantity >= latestProduct.stock) {
    alert("Jumlah produk sudah mencapai batas stok!");
    return;
  }

  try {
    const response = await api.post("/cart/items", {
      product_id: latestProduct.id,
      quantity: 1,
    });

    if (response.data.cart) {
      cart.value = mapCartFromApi(response.data.cart);
    } else {
      await fetchCart();
    }

    alert("Produk berhasil ditambahkan ke keranjang!");
  } catch (error) {
    console.error(
      "Gagal menambahkan produk ke keranjang:",
      error.response?.data || error,
    );

    alert(
      error.response?.data?.message || "Gagal menambahkan produk ke keranjang.",
    );
  }
}

async function increaseQuantity(productId) {
  const item = cart.value.find(
    (product) => Number(product.id) === Number(productId),
  );

  if (!item) return;

  const latestProduct = products.value.find(
    (product) => Number(product.id) === Number(productId),
  );

  if (!latestProduct || latestProduct.stock < 1) {
    alert("Maaf, stok produk sudah habis!");
    return;
  }

  if (item.quantity >= latestProduct.stock) {
    alert("Jumlah produk sudah mencapai batas stok!");
    return;
  }

  try {
    const response = await api.patch(`/cart/items/${item.cartItemId}`, {
      quantity: item.quantity + 1,
    });

    if (response.data.cart) {
      cart.value = mapCartFromApi(response.data.cart);
    } else {
      await fetchCart();
    }
  } catch (error) {
    console.error(
      "Gagal menambah jumlah produk:",
      error.response?.data || error,
    );

    alert(error.response?.data?.message || "Gagal memperbarui jumlah produk.");
  }
}

async function decreaseQuantity(productId) {
  const item = cart.value.find(
    (product) => Number(product.id) === Number(productId),
  );

  if (!item || item.quantity <= 1) return;

  try {
    const response = await api.patch(`/cart/items/${item.cartItemId}`, {
      quantity: item.quantity - 1,
    });

    if (response.data.cart) {
      cart.value = mapCartFromApi(response.data.cart);
    } else {
      await fetchCart();
    }
  } catch (error) {
    console.error(
      "Gagal mengurangi jumlah produk:",
      error.response?.data || error,
    );

    alert(error.response?.data?.message || "Gagal memperbarui jumlah produk.");
  }
}

async function removeFromCart(productId) {
  const item = cart.value.find(
    (product) => Number(product.id) === Number(productId),
  );

  if (!item) {
    alert("Produk tidak ditemukan di keranjang.");
    return;
  }

  const confirmed = await showConfirm(
    `Apakah kamu yakin ingin menghapus "${item.name}" dari keranjang?`,
    "Hapus Produk?",
  );

  if (!confirmed) return;

  try {
    const response = await api.delete(`/cart/items/${item.cartItemId}`);

    if (response.data.cart) {
      cart.value = mapCartFromApi(response.data.cart);
    } else {
      await fetchCart();
    }

    showAlert("Produk berhasil dihapus dari keranjang.", "success");
  } catch (error) {
    console.error(
      "Gagal menghapus produk dari keranjang:",
      error.response?.data || error,
    );

    showAlert(
      error.response?.data?.message ||
        "Gagal menghapus produk dari keranjang. Silakan coba lagi.",
      "error",
    );
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

    api.defaults.headers.common.Authorization = `Bearer ${token}`;

    // Simpan data user
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

    // Bersihkan form dan langsung tutup login
    username.value = "";
    password.value = "";
    showLogin.value = false;

    // Tampilkan popup keberhasilan
    alert("Login berhasil! Selamat datang di Gan's Store.");

    // Arahkan berdasarkan role
    if (user.role === "admin") {
      activePage.value = "admin";

      // Muat data admin di belakang layar
      Promise.all([
        fetchAdminUsers(),
        fetchAdminProducts(),
        fetchAdminCategories(),
        fetchAdminOrders(),
      ]).catch((error) => {
        console.error("Gagal memuat data admin:", error);
      });
    } else {
      activePage.value = "home";

      // Muat data customer di belakang layar
      Promise.all([
        fetchOrders(),
        fetchCart(),
        fetchWishlist(),
        fetchAddress(),
      ]).catch((error) => {
        console.error("Gagal memuat data customer:", error);
      });
    }
  } catch (error) {
    console.error("Login error:", error);

    const message =
      error.response?.data?.message ||
      "Login gagal. Periksa username/email dan password.";

    alert(message);
  }
}

function openAddressForm() {
  addressReturnPage.value = activePage.value;
  addressWasMissing.value = !userAddress.value.recipient;

  addressForm.value = { ...userAddress.value };
  activePage.value = "address";
}

function leaveAddressPage() {
  // Jika sedang mengisi alamat pertama kali dari direct checkout,
  // bersihkan produk checkout agar tidak terbawa ke transaksi berikutnya.
  if (addressWasMissing.value && directCheckoutProduct.value) {
    directCheckoutProduct.value = null;
  }

  activePage.value = addressReturnPage.value;
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

async function saveAddress() {
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

  try {
    const response = await api.put("/address", {
      recipient: address.recipient.trim(),
      phone: address.phone.trim(),
      province: address.province.trim(),
      city: address.city.trim(),
      district: address.district.trim(),
      postal_code: address.postalCode.trim(),
      address_detail: address.detail.trim(),
    });

    const savedAddress = response.data.address;

    // Sinkronkan data alamat dari backend.
    userAddress.value = {
      recipient: savedAddress.recipient,
      phone: savedAddress.phone,
      province: savedAddress.province,
      city: savedAddress.city,
      district: savedAddress.district,
      postalCode: savedAddress.postal_code,
      detail: savedAddress.address_detail,
    };

    addressForm.value = { ...userAddress.value };

    alert("Alamat berhasil disimpan!");

    activePage.value = "checkout";
  } catch (error) {
    console.error("Gagal menyimpan alamat:", error.response?.data || error);

    const errors = error.response?.data?.errors;

    if (errors) {
      alert(Object.values(errors).flat()[0]);
    } else {
      alert(
        error.response?.data?.message ||
          "Gagal menyimpan alamat. Silakan coba lagi.",
      );
    }
  }
}

async function placeOrder() {
  if (!isLoggedIn.value) {
    alert("Silakan login terlebih dahulu!");
    return;
  }

  if (!userAddress.value.recipient) {
    alert("Silakan isi alamat pengiriman terlebih dahulu!");
    activePage.value = "address";
    return;
  }

  if (checkoutProducts.value.length === 0) {
    alert("Tidak ada produk yang akan dipesan!");
    return;
  }

  try {
    // Ambil data produk terbaru dari backend
    const isSuccess = await fetchProducts();

    if (!isSuccess) return;

    // Validasi stok terbaru
    for (const item of checkoutProducts.value) {
      const latestProduct = products.value.find(
        (product) => Number(product.id) === Number(item.id),
      );

      if (!latestProduct || latestProduct.stock < 1) {
        alert(`Maaf, stok ${item.name} sedang habis!`);
        return;
      }

      if (item.quantity > latestProduct.stock) {
        alert(
          `Stok ${item.name} hanya tersedia ${latestProduct.stock} item. Silakan sesuaikan jumlahnya.`,
        );
        return;
      }

      if (!latestProduct.is_active) {
        alert(`Maaf, produk ${item.name} sudah tidak tersedia!`);
        return;
      }
    }

    // Siapkan data yang sesuai dengan validasi Laravel
    const orderData = {
      address: {
        recipient: userAddress.value.recipient,
        phone: userAddress.value.phone,
        province: userAddress.value.province,
        city: userAddress.value.city,
        district: userAddress.value.district,
        postalCode: userAddress.value.postalCode,
        detail: userAddress.value.detail,
      },
      payment_method: paymentMethod.value,
      items: checkoutProducts.value.map((product) => ({
        product_id: product.id,
        quantity: product.quantity,
      })),
    };

    console.log("Checkout Products:", checkoutProducts.value);

    // Kirim pesanan ke backend
    const response = await api.post("/orders", orderData);

    console.log("Pesanan berhasil dibuat:", response.data);

    // Ambil ulang pesanan dari database
    await fetchOrders();

    // Ambil ulang produk agar stok terbaru ditampilkan
    await fetchProducts();

    // Tampilkan modal keberhasilan
    showSuccessModal.value = true;
  } catch (error) {
    console.error("Place order error:", error.response?.data || error);

    const errors = error.response?.data?.errors;

    if (errors) {
      const firstError = Object.values(errors).flat()[0];
      alert(firstError);
    } else {
      alert(
        error.response?.data?.message ||
          "Pesanan gagal dibuat. Silakan coba lagi.",
      );
    }
  }
}

async function finishOrder() {
  try {
    // Kosongkan keranjang backend hanya jika checkout
    // berasal dari halaman keranjang.
    if (!directCheckoutProduct.value) {
      await api.delete("/cart");
      cart.value = [];
    }
  } catch (error) {
    console.error(
      "Gagal mengosongkan keranjang:",
      error.response?.data || error,
    );
    alert(
      "Pesanan berhasil dibuat, tetapi keranjang gagal dikosongkan. Silakan coba lagi.",
    );
    return;
  }
  directCheckoutProduct.value = null;
  showSuccessModal.value = false;
  activePage.value = "home";
  await fetchCart();
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
    wishlistRecords.value = [];
    orders.value = [];
    userAddress.value = {
      recipient: "",
      phone: "",
      province: "",
      city: "",
      district: "",
      postalCode: "",
      detail: "",
    };

    addressForm.value = { ...userAddress.value };
    users.value = [];
    adminProducts.value = [];
    adminCategories.value = [];

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

  // Bersihkan data sebelum memulihkan sesi.
  orders.value = [];
  users.value = [];
  adminProducts.value = [];
  adminCategories.value = [];

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

    if (user.role === "admin") {
      await Promise.all([
        fetchAdminUsers(),
        fetchAdminProducts(),
        fetchAdminCategories(),
        fetchAdminOrders(),
      ]);
    } else {
      await Promise.all([
        fetchOrders(),
        fetchCart(),
        fetchWishlist(),
        fetchAddress(),
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

    orders.value = [];
    users.value = [];
    adminProducts.value = [];
    adminCategories.value = [];
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
  const confirmed = await showConfirm(
    "Apakah kamu yakin ingin menghapus produk ini? Produk yang dihapus tidak dapat dikembalikan.",
    "Hapus Produk?",
  );

  if (!confirmed) return;

  const id = Number(productId);

  try {
    await api.delete(`/admin/products/${id}`);

    // Perbarui data produk admin dan halaman utama.
    await Promise.all([fetchAdminProducts(), fetchProducts()]);

    // Hapus produk dari keranjang lokal.
    cart.value = cart.value.filter((product) => Number(product.id) !== id);

    // Hapus produk dari daftar favorit.
    favorites.value = favorites.value.filter(
      (favoriteId) => Number(favoriteId) !== id,
    );

    // Hapus data Wishlist lokal.
    wishlistRecords.value = wishlistRecords.value.filter(
      (item) => Number(item.product_id) !== id,
    );

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

async function createCategory(category) {
  await saveCategoryToBackend(category, false);
}

async function updateCategory(category) {
  await saveCategoryToBackend(category, true);
}

async function saveCategoryToBackend(category, isUpdate = false) {
  try {
    const payload = {
      name: category.name,
      description: category.description || "",
      is_active: Boolean(category.is_active),
    };

    if (isUpdate) {
      await api.put(`/admin/categories/${category.id}`, payload);
    } else {
      await api.post("/admin/categories", payload);
    }

    await Promise.all([fetchAdminCategories(), fetchCategories()]);

    alert(
      isUpdate
        ? "Kategori berhasil diperbarui."
        : "Kategori berhasil ditambahkan.",
    );
  } catch (error) {
    console.error("Gagal menyimpan kategori:", error.response?.data || error);

    alert(error.response?.data?.message || "Kategori gagal disimpan.");
  }
}

async function deleteCategory(id) {
  try {
    await api.delete(`/admin/categories/${id}`);

    await Promise.all([fetchAdminCategories(), fetchCategories()]);

    alert("Kategori berhasil dihapus.");
  } catch (error) {
    console.error("Gagal menghapus kategori:", error.response?.data || error);

    alert(
      error.response?.data?.message ||
        "Kategori gagal dihapus. Pastikan tidak ada produk yang menggunakan kategori ini.",
    );
  }
}

async function updateOrderStatus({ orderId, status }) {
  const order = orders.value.find(
    (item) => Number(item.id) === Number(orderId),
  );

  if (!order) {
    alert("Pesanan tidak ditemukan.");
    return;
  }

  const previousStatus = order.status;

  try {
    const response = await api.patch(`/admin/orders/${orderId}/status`, {
      status,
    });

    order.status = response.data.order?.status || status;

    if (status === "Dibatalkan" && previousStatus !== "Dibatalkan") {
      await Promise.all([fetchProducts(), fetchAdminProducts()]);
    }

    await fetchAdminOrders();

    alert("Status pesanan berhasil diperbarui.");
  } catch (error) {
    console.error("Update order status error:", error.response?.data || error);

    order.status = previousStatus;

    alert(error.response?.data?.message || "Gagal memperbarui status pesanan.");
  }
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
      :category-records="adminCategories"
      @create-category="createCategory"
      @update-category="updateCategory"
      @delete-category="deleteCategory"
      @logout="logout"
      @create-product="createProduct"
      @update-product="updateProduct"
      @delete-product="deleteProduct"
      @update-user-role="updateUserRole"
      @toggle-user-status="toggleUserStatus"
      @update-order-status="updateOrderStatus"
    />

    <HomePage
      v-if="activePage === 'home' && currentUserRole !== 'admin'"
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
      v-if="activePage === 'wishlist' && currentUserRole === 'customer'"
      :wishlist-products="wishlistProducts"
      :format-price="formatPrice"
      @home="activePage = 'home'"
      @toggle-favorite="toggleFavorite"
      @add-to-cart="addToCart"
    />

    <CartPage
      v-if="activePage === 'cart' && currentUserRole === 'customer'"
      :cart-products="cartProducts"
      :cart-total="cartTotal"
      :format-price="formatPrice"
      @home="activePage = 'home'"
      @decrease="decreaseQuantity"
      @increase="increaseQuantity"
      @remove="removeFromCart"
      @checkout="startCheckout"
    />

    <AddressPage
      v-if="activePage === 'address' && currentUserRole === 'customer'"
      :address-form="addressForm"
      :has-saved-address="Boolean(userAddress.recipient)"
      @back="leaveAddressPage"
      @cancel="leaveAddressPage"
      @save="saveAddress"
    />

    <CheckoutPage
      v-if="activePage === 'checkout' && currentUserRole === 'customer'"
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
      v-if="activePage === 'orders' && currentUserRole === 'customer'"
      :orders="orders"
      :format-price="formatPrice"
      @home="activePage = 'home'"
      @cancel="cancelOrder"
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

  <!-- CUSTOM APP POPUP -->
  <AppPopup
    v-model="showAppPopup"
    :type="popupType"
    :title="popupTitle"
    :message="popupMessage"
    :confirm-text="popupConfirmText"
    :cancel-text="popupCancelText"
    :show-cancel="popupShowCancel"
    @confirm="handlePopupConfirm"
    @cancel="handlePopupCancel"
  />

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
      <!-- <p class="demo-admin-hint">
        Demo admin: <strong>admin</strong> / <strong>admin123</strong>
      </p> -->

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
