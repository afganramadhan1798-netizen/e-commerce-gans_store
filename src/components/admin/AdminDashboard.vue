<script setup>
import { computed, ref } from "vue";

const props = defineProps({
  products: {
    type: Array,
    default: () => [],
  },
  users: {
    type: Array,
    default: () => [],
  },
  orders: {
    type: Array,
    default: () => [],
  },
  categories: {
    type: Array,
    default: () => [],
  },
  categoryRecords: {
    type: Array,
    default: () => [],
  },
  formatPrice: {
    type: Function,
    required: true,
  },
  userName: {
    type: String,
    default: "Admin",
  },
});

const emit = defineEmits([
  "home",
  "logout",
  "create-product",
  "update-product",
  "delete-product",
  "update-user-role",
  "toggle-user-status",
  "update-order-status",

  // Manajemen kategori
  "create-category",
  "update-category",
  "delete-category",
]);

const adminDialog = ref({
  open: false,
  type: "alert",
  title: "",
  message: "",
  confirmText: "Ya, Lanjutkan",
  onConfirm: null,
});

function showAdminAlert(message, title = "Informasi") {
  adminDialog.value = {
    open: true,
    type: "alert",
    title,
    message,
    confirmText: "Mengerti",
    onConfirm: null,
  };
}

function showAdminConfirm(message, onConfirm, title = "Konfirmasi") {
  adminDialog.value = {
    open: true,
    type: "confirm",
    title,
    message,
    confirmText: "Ya, Lanjutkan",
    onConfirm,
  };
}

function closeAdminDialog() {
  adminDialog.value.open = false;
}

function confirmAdminDialog() {
  const action = adminDialog.value.onConfirm;

  closeAdminDialog();
  action?.();
}

// STATE KATEGORI
const categorySearch = ref("");
const categoryDialogOpen = ref(false);
const editingCategoryId = ref(null);

const categoryForm = ref({
  name: "",
  description: "",
  is_active: true,
});

// FILTER KATEGORI
const filteredCategories = computed(() => {
  const keyword = categorySearch.value.trim().toLowerCase();

  if (!keyword) {
    return props.categoryRecords;
  }

  return props.categoryRecords.filter((category) =>
    `${category.name} ${category.description || ""}`
      .toLowerCase()
      .includes(keyword),
  );
});

// MENGHITUNG KATEGORI AKTIF
const activeCategoryCount = computed(() => {
  return props.categoryRecords.filter((category) => category.is_active).length;
});

// FORM KATEGORI KOSONG
function createEmptyCategory() {
  return {
    name: "",
    description: "",
    is_active: true,
  };
}

// MEMBUKA FORM TAMBAH KATEGORI
function openCreateCategory() {
  editingCategoryId.value = null;
  categoryForm.value = createEmptyCategory();
  categoryDialogOpen.value = true;
}

// MEMBUKA FORM EDIT KATEGORI
function openEditCategory(category) {
  editingCategoryId.value = category.id;

  categoryForm.value = {
    name: category.name,
    description: category.description || "",
    is_active: Boolean(category.is_active),
  };

  categoryDialogOpen.value = true;
}

// MENYIMPAN KATEGORI
function saveCategory() {
  const name = categoryForm.value.name.trim();

  if (!name) {
    showAdminAlert("Nama kategori wajib diisi.", "Data Belum Lengkap");
    return;
  }

  const categoryData = {
    name,
    description: categoryForm.value.description.trim(),
    is_active: categoryForm.value.is_active,
  };

  if (editingCategoryId.value !== null) {
    emit("update-category", {
      id: editingCategoryId.value,
      ...categoryData,
    });
  } else {
    emit("create-category", categoryData);
  }

  categoryDialogOpen.value = false;
}

// MENGUBAH STATUS KATEGORI
function toggleCategoryStatus(category) {
  emit("update-category", {
    id: category.id,
    name: category.name,
    description: category.description || "",
    is_active: !category.is_active,
  });
}

// MENGHAPUS KATEGORI
function deleteCategory(category) {
  showAdminConfirm(
    `Apakah kamu yakin ingin menghapus kategori "${category.name}"?`,
    () => emit("delete-category", category.id),
    "Hapus Kategori",
  );
}

const activeTab = ref("products");
const selectedOrder = ref(null);

function openOrderDetail(order) {
  selectedOrder.value = order;
}

function closeOrderDetail() {
  selectedOrder.value = null;
}

const productSearch = ref("");
const productDialogOpen = ref(false);
const editingProductId = ref(null);
const productForm = ref(createEmptyProduct());

const selectedImage = ref(null);
const imagePreview = ref("");

const categoryOptions = computed(() => {
  if (props.categoryRecords.length > 0) {
    return props.categoryRecords
      .filter((category) => category.is_active)
      .map((category) => category.name);
  }

  return props.categories.filter((category) => category !== "Best Seller");
});

const uniqueProducts = computed(() =>
  [
    ...new Map(props.products.map((product) => [product.id, product])).values(),
  ].map((product) => ({ ...product, stock: product.stock ?? 10 })),
);

const filteredProducts = computed(() => {
  const query = productSearch.value.trim().toLowerCase();
  if (!query) return uniqueProducts.value;

  return uniqueProducts.value.filter((product) =>
    `${product.name} ${product.variant || ""} ${product.category || ""}`
      .toLowerCase()
      .includes(query),
  );
});

const activeUserCount = computed(
  () => props.users.filter((user) => user.status === "Aktif").length,
);

const pendingOrderCount = computed(
  () =>
    props.orders.filter(
      (order) => !["Selesai", "Dibatalkan"].includes(order.status),
    ).length,
);

const lowStockCount = computed(
  () => uniqueProducts.value.filter((product) => product.stock <= 5).length,
);

function createEmptyProduct() {
  return {
    name: "",
    variant: "",
    price: 0,
    category: "",
    image: "",
    imageFile: null,
    bestseller: false,
    stock: 0,
    description: "",
  };
}

function openCreateProduct() {
  editingProductId.value = null;
  productForm.value = {
    ...createEmptyProduct(),
    category: categoryOptions.value[0] || "",
  };

  selectedImage.value = null;
  imagePreview.value = "";
  productDialogOpen.value = true;
}

function openEditProduct(product) {
  editingProductId.value = product.id;
  productForm.value = {
    ...createEmptyProduct(),
    ...product,
    category: product.category || product.category?.name || "",
    is_active: Boolean(product.is_active),
  };

  selectedImage.value = null;
  imagePreview.value = product.image_url || product.image || "";
  productDialogOpen.value = true;
}

const allowedTypes = ["image/jpeg", "image/png", "image/jpg", "image/webp"];
function handleImageChange(event) {
  const file = event.target.files?.[0] || null;

  if (!file) {
    productForm.value.imageFile = null;
    selectedImage.value = null;
    return;
  }

  if (!allowedTypes.includes(file.type)) {
    showAdminAlert(
      "Format gambar harus JPG, PNG, atau WEBP.",
      "Format Gambar Tidak Valid",
    );

    event.target.value = "";
    productForm.value.imageFile = null;
    return;
  }

  if (file.size > 2 * 1024 * 1024) {
    showAdminAlert(
      "Ukuran gambar maksimal 2 MB. Silakan pilih gambar yang lebih kecil.",
      "Ukuran Gambar Terlalu Besar",
    );

    event.target.value = "";
    productForm.value.imageFile = null;
    return;
  }

  productForm.value.imageFile = file;
  selectedImage.value = file;
  imagePreview.value = URL.createObjectURL(file);
}

function saveProduct() {
  const product = {
    ...productForm.value,
    price: Number(productForm.value.price),
    stock: Number(productForm.value.stock),
  };

  if (editingProductId.value !== null) {
    emit("update-product", {
      ...product,
      id: editingProductId.value,
    });
  } else {
    emit("create-product", product);
  }

  productDialogOpen.value = false;
}

function deleteProduct(product) {
  showAdminConfirm(
    `Hapus ${product.name}${product.variant ? ` - ${product.variant}` : ""} dari katalog toko?`,
    () => emit("delete-product", product.id),
    "Hapus Produk",
  );
}

function setOrderStatus(orderId, status) {
  emit("update-order-status", { orderId, status });
}

function formatOrderDate(date) {
  if (!date) return "-";

  return new Date(date).toLocaleString("id-ID", {
    day: "numeric",
    month: "long",
    year: "numeric",
    hour: "2-digit",
    minute: "2-digit",
  });
}
</script>

<template>
  <main class="admin-page">
    <header class="admin-header">
      <div class="admin-brand">
        <div class="admin-brand-mark">G</div>
        <div class="admin-brand-name">
          <strong>gan's store</strong>
          <span>ADMIN</span>
        </div>
      </div>

      <nav class="admin-tabs" aria-label="Menu admin">
        <button
          :class="{ active: activeTab === 'products' }"
          @click="activeTab = 'products'"
        >
          Produk <span>{{ uniqueProducts.length }}</span>
        </button>

        <button
          :class="{ active: activeTab === 'categories' }"
          @click="activeTab = 'categories'"
        >
          Kategori <span>{{ props.categoryRecords.length }}</span>
        </button>

        <button
          :class="{ active: activeTab === 'users' }"
          @click="activeTab = 'users'"
        >
          Pengguna <span>{{ props.users.length }}</span>
        </button>

        <button
          :class="{ active: activeTab === 'orders' }"
          @click="activeTab = 'orders'"
        >
          Pesanan <span>{{ props.orders.length }}</span>
        </button>
      </nav>

      <div class="admin-header-actions">
        <span class="admin-account-name" :title="userName">{{ userName }}</span>
        <button class="admin-logout-button" @click="emit('logout')">
          Logout
        </button>
      </div>
    </header>

    <div class="admin-heading">
      <div>
        <p class="admin-eyebrow">GAN'S STORE / ADMIN</p>
        <h1>Manajemen Toko</h1>
        <p class="admin-subtitle">Kelola katalog, pengguna, dan pesanan.</p>
      </div>
    </div>

    <section class="admin-metrics" aria-label="Ringkasan toko">
      <div class="admin-metric">
        <span class="admin-metric-label">Produk perlu dicek</span>
        <strong>{{ lowStockCount }}</strong>
        <small>Stok 5 atau kurang</small>
      </div>
      <div class="admin-metric">
        <span class="admin-metric-label">Pengguna aktif</span>
        <strong>{{ activeUserCount }}</strong>
        <small>{{ users.length }} akun terdaftar</small>
      </div>
      <div class="admin-metric">
        <span class="admin-metric-label">Pesanan berjalan</span>
        <strong>{{ pendingOrderCount }}</strong>
        <small>{{ orders.length }} pesanan keseluruhan</small>
      </div>
    </section>

    <section class="admin-workspace">
      <div v-if="activeTab === 'products'" class="admin-panel">
        <div class="admin-panel-heading">
          <div>
            <h2>Produk</h2>
            <p>Atur informasi produk, kategori, harga, dan stok.</p>
          </div>
          <button class="admin-primary-button" @click="openCreateProduct">
            <span aria-hidden="true">＋</span> Tambah Produk
          </button>
        </div>

        <label class="admin-search">
          <span aria-hidden="true">⌕</span>
          <input
            v-model="productSearch"
            placeholder="Cari nama atau kategori"
          />
        </label>

        <div class="admin-table-wrap">
          <table class="admin-table">
            <thead>
              <tr>
                <th>Nama Produk</th>
                <th>Kategori</th>
                <th>Harga</th>
                <th>Stok</th>
                <th>Etalase</th>
                <th><span class="sr-only">Aksi</span></th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="product in filteredProducts" :key="product.id">
                <td>
                  <div class="admin-product-cell">
                    <img
                      :src="
                        product.image_url ||
                        'https://placehold.co/500x400/f1f1f1/2438d2?text=Gans+Store'
                      "
                      :alt="product.name"
                    />
                    <div>
                      <strong>{{ product.name }}</strong>
                      <small v-if="product.variant">{{
                        product.variant
                      }}</small>
                    </div>
                  </div>
                </td>
                <td>{{ product.category }}</td>
                <td>{{ formatPrice(product.price) }}</td>
                <td>
                  <span
                    class="admin-stock"
                    :class="{ low: product.stock <= 5 }"
                  >
                    {{ product.stock }} unit
                  </span>
                </td>
                <td>
                  <span
                    class="admin-bestseller"
                    :class="{ featured: product.bestseller }"
                  >
                    {{ product.bestseller ? "Best seller" : "Reguler" }}
                  </span>
                </td>
                <td>
                  <div class="admin-row-actions">
                    <button
                      class="admin-icon-button"
                      title="Edit produk"
                      @click="openEditProduct(product)"
                    >
                      ✎
                    </button>
                    <button
                      class="admin-icon-button danger"
                      title="Hapus produk"
                      @click="deleteProduct(product)"
                    >
                      ×
                    </button>
                  </div>
                </td>
              </tr>
              <tr v-if="filteredProducts.length === 0">
                <td colspan="6" class="admin-empty-row">
                  Produk tidak ditemukan.
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <div v-else-if="activeTab === 'users'" class="admin-panel">
        <div class="admin-panel-heading">
          <div>
            <h2>Pengguna</h2>
            <p>Atur peran dan akses akun yang terdaftar.</p>
          </div>
        </div>

        <div class="admin-table-wrap">
          <table class="admin-table">
            <thead>
              <tr>
                <th>Pengguna</th>
                <th>Peran</th>
                <th>Status</th>
                <th>Bergabung</th>
                <th><span class="sr-only">Aksi</span></th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="user in users" :key="user.id">
                <td>
                  <div class="admin-user-cell">
                    <span class="admin-user-avatar">
                      {{
                        (user.name || user.username || "U")
                          .charAt(0)
                          .toUpperCase()
                      }}
                    </span>
                    <div>
                      <strong>{{ user.name || user.username }}</strong>
                      <small>{{ user.email }}</small>
                    </div>
                  </div>
                </td>
                <td>
                  <select
                    class="admin-select"
                    :value="user.role"
                    :disabled="user.isCurrentUser"
                    @change="
                      emit('update-user-role', {
                        userId: user.id,
                        role: $event.target.value,
                      })
                    "
                  >
                    <option value="customer">Customer</option>
                    <option value="admin">Admin</option>
                  </select>
                </td>
                <td>
                  <span
                    class="admin-status"
                    :class="user.status === 'Aktif' ? 'positive' : 'muted'"
                  >
                    {{ user.status }}
                  </span>
                </td>
                <td>{{ user.joinedAt || "Demo" }}</td>
                <td>
                  <button
                    v-if="!user.isCurrentUser"
                    class="admin-text-button"
                    :class="{ reactivate: user.status !== 'Aktif' }"
                    @click="emit('toggle-user-status', user.id)"
                  >
                    {{ user.status === "Aktif" ? "Nonaktifkan" : "Aktifkan" }}
                  </button>
                  <span v-else class="admin-self-label">Akun Anda</span>
                </td>
              </tr>
              <tr v-if="users.length === 0">
                <td colspan="5" class="admin-empty-row">Belum ada pengguna.</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <div v-else-if="activeTab === 'orders'" class="admin-panel">
        <div class="admin-panel-heading">
          <div>
            <h2>Pesanan</h2>
            <p>Perbarui tahap pesanan pelanggan.</p>
          </div>
        </div>

        <div v-if="orders.length" class="admin-orders-list">
          <article
            v-for="order in orders"
            :key="order.id"
            class="admin-order-row"
          >
            <div class="admin-order-main">
              <strong>{{ order.id }}</strong>
              <span
                >{{ order.customerName || "Pelanggan" }} ·
                {{ order.customerEmail || "" }}</span
              >
              <small>
                {{ order.date }} · {{ order.items?.length ?? 0 }} jenis produk
              </small>
            </div>
            <div class="admin-order-total">
              <span>Total</span>
              <strong>{{ formatPrice(order.total) }}</strong>
            </div>
            <label class="admin-order-status">
              <span>Status</span>
              <select
                class="admin-select"
                :value="order.status"
                @change="setOrderStatus(order.id, $event.target.value)"
              >
                <option>Pesanan Dibuat</option>
                <option>Diproses</option>
                <option>Dikirim</option>
                <option>Selesai</option>
                <option>Dibatalkan</option>
              </select>
            </label>
            <div class="admin-order-actions">
              <button
                class="admin-detail-button"
                @click="openOrderDetail(order)"
              >
                Lihat Detail
              </button>
            </div>
          </article>
        </div>
        <div v-else class="admin-empty-state">
          <span>▤</span>
          <strong>Belum ada pesanan</strong>
          <p>Pesanan pelanggan akan muncul di sini.</p>
        </div>
      </div>

      <div v-else-if="activeTab === 'categories'" class="admin-panel">
        <div class="admin-panel-heading">
          <div>
            <h2>Manajemen Kategori</h2>
            <p>Kelola kategori produk Gan's Store.</p>
          </div>

          <button class="admin-primary-button" @click="openCreateCategory">
            <span aria-hidden="true">＋</span> Tambah Kategori
          </button>
        </div>

        <label class="admin-search">
          <span aria-hidden="true">⌕</span>
          <input v-model="categorySearch" placeholder="Cari nama kategori" />
        </label>

        <div class="admin-category-summary">
          <div>
            <span>Total Kategori</span>
            <strong>{{ props.categoryRecords.length }}</strong>
          </div>
          <div>
            <span>Kategori Aktif</span>
            <strong>{{ activeCategoryCount }}</strong>
          </div>
        </div>

        <div class="admin-table-wrap">
          <table class="admin-table">
            <thead>
              <tr>
                <th>Nama Kategori</th>
                <th>Slug</th>
                <th>Deskripsi</th>
                <th>Status</th>
                <th><span class="sr-only">Aksi</span></th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="category in filteredCategories" :key="category.id">
                <td>
                  <strong>{{ category.name }}</strong>
                </td>
                <td>{{ category.slug }}</td>
                <td>{{ category.description || "-" }}</td>
                <td>
                  <span
                    class="admin-status"
                    :class="category.is_active ? 'positive' : 'muted'"
                  >
                    {{ category.is_active ? "Aktif" : "Nonaktif" }}
                  </span>
                </td>
                <td>
                  <div class="admin-row-actions">
                    <button
                      class="admin-icon-button"
                      title="Edit kategori"
                      @click="openEditCategory(category)"
                    >
                      ✎
                    </button>

                    <button
                      class="admin-text-button"
                      :class="{ reactivate: !category.is_active }"
                      @click="toggleCategoryStatus(category)"
                    >
                      {{ category.is_active ? "Nonaktifkan" : "Aktifkan" }}
                    </button>

                    <button
                      class="admin-icon-button danger"
                      title="Hapus kategori"
                      @click="deleteCategory(category)"
                    >
                      ×
                    </button>
                  </div>
                </td>
              </tr>

              <tr v-if="filteredCategories.length === 0">
                <td colspan="5" class="admin-empty-row">
                  Belum ada kategori yang ditemukan.
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </section>

    <div
      v-if="productDialogOpen"
      class="admin-dialog-backdrop"
      @click.self="productDialogOpen = false"
    >
      <section
        class="admin-dialog"
        role="dialog"
        aria-modal="true"
        aria-labelledby="product-dialog-title"
      >
        <div class="admin-dialog-heading">
          <div>
            <p class="admin-eyebrow">KATALOG TOKO</p>
            <h2 id="product-dialog-title">
              {{ editingProductId === null ? "Tambah Produk" : "Edit Produk" }}
            </h2>
          </div>
          <button
            class="admin-icon-button"
            title="Tutup"
            @click="productDialogOpen = false"
          >
            ×
          </button>
        </div>

        <form class="admin-product-form" @submit.prevent="saveProduct">
          <label>
            Nama produk
            <input v-model.trim="productForm.name" required maxlength="80" />
          </label>
          <label>
            Varian
            <input
              v-model.trim="productForm.variant"
              placeholder="Opsional"
              maxlength="60"
            />
          </label>
          <div class="admin-form-row">
            <label>
              Kategori
              <select v-model="productForm.category" required>
                <option
                  v-for="category in categoryOptions"
                  :key="category"
                  :value="category"
                >
                  {{ category }}
                </option>
              </select>
            </label>
            <label>
              Harga (Rp)
              <input
                v-model.number="productForm.price"
                type="number"
                min="1"
                required
              />
            </label>
          </div>
          <div class="admin-form-row">
            <label>
              Stok
              <input
                v-model.number="productForm.stock"
                type="number"
                min="0"
                required
              />
            </label>
            <label class="admin-check-field">
              <input v-model="productForm.bestseller" type="checkbox" />
              Tampilkan sebagai best seller
            </label>
          </div>

          <label class="admin-check-field">
            <input v-model="productForm.is_active" type="checkbox" />
            Produk aktif dan tampil di toko
          </label>

          <label>
            Gambar produk
            <input
              type="file"
              accept="image/jpeg,image/png,image/jpg,image/webp"
              @change="handleImageChange"
            />
            <small v-if="productForm.imageFile">
              {{ productForm.imageFile.name }}
            </small>
            <small
              v-else-if="editingProductId !== null && productForm.image_url"
            >
              Gambar saat ini sudah tersimpan. Pilih file baru jika ingin
              menggantinya.
            </small>
          </label>

          <div v-if="imagePreview" class="admin-image-preview">
            <img :src="imagePreview" alt="Preview gambar produk" />
          </div>

          <label>
            Deskripsi
            <textarea
              v-model.trim="productForm.description"
              rows="3"
              maxlength="500"
            />
          </label>
          <div class="admin-dialog-actions">
            <button
              type="button"
              class="admin-secondary-button"
              @click="productDialogOpen = false"
            >
              Batal
            </button>
            <button type="submit" class="admin-primary-button">
              {{
                editingProductId === null ? "Simpan Produk" : "Simpan Perubahan"
              }}
            </button>
          </div>
        </form>
      </section>
    </div>
  </main>

  <div
    v-if="categoryDialogOpen"
    class="admin-dialog-backdrop"
    @click.self="categoryDialogOpen = false"
  >
    <section
      class="admin-dialog"
      role="dialog"
      aria-modal="true"
      aria-labelledby="category-dialog-title"
    >
      <div class="admin-dialog-heading">
        <div>
          <p class="admin-eyebrow">KATEGORI TOKO</p>
          <h2 id="category-dialog-title">
            {{
              editingCategoryId === null ? "Tambah Kategori" : "Edit Kategori"
            }}
          </h2>
        </div>

        <button
          class="admin-icon-button"
          title="Tutup"
          @click="categoryDialogOpen = false"
        >
          ×
        </button>
      </div>

      <form class="admin-product-form" @submit.prevent="saveCategory">
        <label>
          Nama kategori
          <input
            v-model.trim="categoryForm.name"
            type="text"
            placeholder="Contoh: Fashion"
            maxlength="80"
            required
          />
        </label>

        <label>
          Deskripsi
          <textarea
            v-model.trim="categoryForm.description"
            rows="4"
            maxlength="500"
            placeholder="Masukkan deskripsi kategori..."
          ></textarea>
        </label>

        <label class="admin-check-field">
          <input v-model="categoryForm.is_active" type="checkbox" />
          Kategori aktif
        </label>

        <div class="admin-dialog-actions">
          <button
            type="button"
            class="admin-secondary-button"
            @click="categoryDialogOpen = false"
          >
            Batal
          </button>

          <button type="submit" class="admin-primary-button">
            {{
              editingCategoryId === null
                ? "Simpan Kategori"
                : "Simpan Perubahan"
            }}
          </button>
        </div>
      </form>
    </section>
  </div>

  <!-- MODAL DETAIL PESANAN -->
  <div
    v-if="selectedOrder"
    class="admin-dialog-backdrop"
    @click.self="closeOrderDetail"
  >
    <section
      class="admin-dialog admin-order-dialog"
      role="dialog"
      aria-modal="true"
      aria-labelledby="order-detail-title"
    >
      <div class="admin-dialog-heading">
        <div>
          <p class="admin-eyebrow">INFORMASI TRANSAKSI</p>
          <h2 id="order-detail-title">Detail Pesanan</h2>
          <span class="admin-order-number">
            {{ selectedOrder.orderNumber || selectedOrder.id }}
          </span>
        </div>

        <button
          class="admin-icon-button"
          title="Tutup"
          @click="closeOrderDetail"
        >
          ×
        </button>
      </div>

      <!-- STATUS PESANAN -->
      <div class="admin-detail-status">
        <span>Status Pesanan</span>
        <strong>{{ selectedOrder.status }}</strong>
      </div>

      <!-- INFORMASI PELANGGAN -->
      <div class="admin-detail-section">
        <h3>Informasi Pelanggan</h3>

        <div class="admin-detail-grid">
          <div>
            <span>Nama Penerima</span>
            <strong>{{
              selectedOrder.address?.recipient ||
              selectedOrder.customerName ||
              "-"
            }}</strong>
          </div>

          <div>
            <span>Email</span>
            <strong>{{ selectedOrder.customerEmail || "-" }}</strong>
          </div>

          <div>
            <span>Nomor Telepon</span>
            <strong>{{ selectedOrder.address?.phone || "-" }}</strong>
          </div>

          <div>
            <span>Tanggal Pesanan</span>
            <strong>{{ formatOrderDate(selectedOrder.date) }}</strong>
          </div>
        </div>
      </div>

      <!-- ALAMAT PENGIRIMAN -->
      <div class="admin-detail-section">
        <h3>Alamat Pengiriman</h3>

        <p class="admin-detail-address">
          {{ selectedOrder.address?.detail || "-" }},
          {{ selectedOrder.address?.district || "-" }},
          {{ selectedOrder.address?.city || "-" }},
          {{ selectedOrder.address?.province || "-" }}
          {{ selectedOrder.address?.postalCode || "" }}
        </p>
      </div>

      <!-- PRODUK YANG DIPESAN -->
      <div class="admin-detail-section">
        <h3>Produk Dipesan</h3>

        <div
          v-for="(item, index) in selectedOrder.items || []"
          :key="item.id || index"
          class="admin-detail-product"
        >
          <img v-if="item.image" :src="item.image" :alt="item.name" />

          <div class="admin-detail-product-info">
            <strong>{{ item.name }}</strong>
            <span v-if="item.variant">{{ item.variant }}</span>
            <small> {{ item.quantity }} × {{ formatPrice(item.price) }} </small>
          </div>

          <strong class="admin-detail-subtotal">
            {{ formatPrice(item.subtotal ?? item.price * item.quantity) }}
          </strong>
        </div>

        <p v-if="!selectedOrder.items?.length" class="admin-detail-empty">
          Tidak ada informasi produk.
        </p>
      </div>

      <!-- PEMBAYARAN -->
      <div class="admin-detail-section">
        <h3>Ringkasan Pembayaran</h3>

        <div class="admin-payment-row">
          <span>Metode Pembayaran</span>
          <strong>{{ selectedOrder.paymentMethod || "-" }}</strong>
        </div>

        <div class="admin-payment-row">
          <span>Subtotal Produk</span>
          <strong>
            {{
              formatPrice(
                selectedOrder.subtotal ??
                  (selectedOrder.items || []).reduce(
                    (total, item) =>
                      total +
                      Number(item.subtotal ?? item.price * item.quantity),
                    0,
                  ),
              )
            }}
          </strong>
        </div>

        <div class="admin-payment-row">
          <span>Ongkos Kirim</span>
          <strong>{{ formatPrice(selectedOrder.shippingCost ?? 0) }}</strong>
        </div>

        <div class="admin-payment-total">
          <span>Total Pembayaran</span>
          <strong>{{ formatPrice(selectedOrder.total) }}</strong>
        </div>
      </div>

      <div class="admin-dialog-actions">
        <button class="admin-secondary-button" @click="closeOrderDetail">
          Tutup
        </button>
      </div>
    </section>
  </div>

  <!-- CUSTOM ADMIN POPUP -->
  <div
    v-if="adminDialog.open"
    class="admin-dialog-backdrop admin-alert-backdrop"
    @click.self="closeAdminDialog"
  >
    <section
      class="admin-dialog admin-confirm-dialog"
      role="alertdialog"
      aria-modal="true"
      aria-labelledby="admin-alert-title"
    >
      <div class="admin-alert-icon">
        <span v-if="adminDialog.type === 'confirm'">?</span>
        <span v-else>i</span>
      </div>

      <h2 id="admin-alert-title">{{ adminDialog.title }}</h2>

      <p>{{ adminDialog.message }}</p>

      <div class="admin-dialog-actions">
        <button
          v-if="adminDialog.type === 'confirm'"
          class="admin-secondary-button"
          @click="closeAdminDialog"
        >
          Batal
        </button>

        <button
          class="admin-primary-button"
          @click="
            adminDialog.type === 'confirm'
              ? confirmAdminDialog()
              : closeAdminDialog()
          "
        >
          {{ adminDialog.confirmText }}
        </button>
      </div>
    </section>
  </div>
</template>

<style scoped>
.admin-page {
  width: 100%;
  margin: 0;
  padding: 0 0 64px;
  color: #172033;
}

.admin-header {
  display: flex;
  align-items: center;
  gap: 28px;
  min-height: 78px;
  padding: 10px max(24px, calc((100% - 1200px) / 2));
  border-bottom: 1px solid #e1e5ed;
  background: #fff;
}

.admin-brand {
  display: flex;
  align-items: center;
  gap: 10px;
  flex: 0 0 auto;
}

.admin-brand-mark {
  display: grid;
  place-items: center;
  width: 42px;
  height: 42px;
  border-radius: 50%;
  background: #2438d2;
  color: #fff;
  font-size: 21px;
  font-weight: 700;
}

.admin-brand-name {
  display: grid;
  gap: 2px;
  color: #2438d2;
  white-space: nowrap;
}

.admin-brand-name strong {
  font-size: 14px;
}

.admin-brand-name span {
  color: #69758a;
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 0.1em;
}
.admin-heading,
.admin-panel-heading,
.admin-dialog-heading {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 20px;
}

.admin-heading {
  margin-bottom: 26px;
}

.admin-eyebrow {
  color: #5263cc;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.1em;
}

.admin-heading h1 {
  margin-top: 5px;
  font-size: 30px;
  line-height: 1.2;
}

.admin-subtitle,
.admin-panel-heading p {
  margin-top: 6px;
  color: #657087;
  font-size: 13px;
}

.admin-store-link,
.admin-secondary-button {
  border: 1px solid #d6dbe7;
  border-radius: 6px;
  background: #fff;
  color: #263248;
  padding: 10px 14px;
  font-weight: 600;
}

.admin-metrics {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  border: 1px solid #e1e5ed;
  border-radius: 8px;
  background: #fff;
  margin-bottom: 24px;
}

.admin-metric {
  padding: 18px 22px;
  display: grid;
  gap: 4px;
  border-right: 1px solid #e1e5ed;
}

.admin-metric:last-child {
  border-right: 0;
}

.admin-metric-label,
.admin-metric small {
  color: #69758a;
  font-size: 12px;
}

.admin-metric strong {
  font-size: 25px;
  line-height: 1.2;
}

.admin-workspace {
  border: 1px solid #e1e5ed;
  border-radius: 8px;
  background: #fff;
  overflow: hidden;
}

.admin-tabs {
  display: flex;
  gap: 26px;
  padding: 0 22px;
  border-bottom: 1px solid #e6e9f0;
}

.admin-tabs button {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 16px 2px 13px;
  border: 0;
  border-bottom: 3px solid transparent;
  background: transparent;
  color: #647087;
  font-size: 13px;
  font-weight: 600;
}

.admin-tabs button.active {
  border-bottom-color: #2438d2;
  color: #2438d2;
}

.admin-tabs button span {
  min-width: 22px;
  padding: 2px 6px;
  border-radius: 10px;
  background: #f0f2f8;
  color: #59647a;
  font-size: 11px;
  text-align: center;
}

.admin-header .admin-tabs {
  flex: 1;
  justify-content: center;
  gap: 18px;
  padding: 0;
  border: 0;
}

.admin-header .admin-tabs button {
  align-self: stretch;
  padding: 12px 8px 10px;
  border-bottom-width: 3px;
  white-space: nowrap;
}

.admin-header-actions {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 10px;
  flex: 0 0 auto;
}

.admin-account-name {
  max-width: 130px;
  overflow: hidden;
  color: #263248;
  font-size: 12px;
  font-weight: 600;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.admin-logout-button {
  min-height: 36px;
  padding: 0 12px;
  border: 0;
  border-radius: 6px;
  background: #ff5050;
  color: #fff;
  font-size: 12px;
  font-weight: 600;
}

.admin-logout-button:hover {
  background: #d94343;
}

.admin-panel {
  padding: 22px;
}

.admin-panel-heading {
  margin-bottom: 18px;
}

.admin-panel-heading h2 {
  font-size: 19px;
}

.admin-primary-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 7px;
  min-height: 40px;
  padding: 0 15px;
  border: 0;
  border-radius: 6px;
  background: #2438d2;
  color: #fff;
  font-size: 13px;
  font-weight: 600;
}

.admin-primary-button:hover {
  background: #1729b5;
}

.admin-primary-button span {
  font-size: 18px;
  line-height: 1;
}

.admin-search {
  display: flex;
  align-items: center;
  gap: 8px;
  width: min(350px, 100%);
  height: 39px;
  margin-bottom: 16px;
  padding: 0 10px;
  border: 1px solid #dce1ea;
  border-radius: 6px;
  color: #7a8497;
}

.admin-search input {
  min-width: 0;
  width: 100%;
  border: 0;
  outline: 0;
  color: #172033;
  font: inherit;
  font-size: 12px;
}

.admin-table-wrap {
  width: 100%;
  overflow-x: auto;
}

.admin-table {
  width: 100%;
  border-collapse: collapse;
  text-align: left;
  white-space: nowrap;
}

.admin-table th {
  padding: 11px 12px;
  border-top: 1px solid #e8ebf1;
  border-bottom: 1px solid #e8ebf1;
  background: #f8f9fc;
  color: #68748a;
  font-size: 11px;
  font-weight: 600;
}

.admin-table td {
  padding: 12px;
  border-bottom: 1px solid #edf0f4;
  color: #465269;
  font-size: 12px;
}

.admin-table tbody tr:last-child td {
  border-bottom: 0;
}

.admin-product-cell,
.admin-user-cell {
  display: flex;
  align-items: center;
  gap: 10px;
  min-width: 190px;
}

.admin-product-cell img {
  width: 42px;
  height: 42px;
  border-radius: 5px;
  background: #f1f3f7;
  object-fit: cover;
}

.admin-product-cell div,
.admin-user-cell div {
  display: grid;
  gap: 3px;
}

.admin-product-cell strong,
.admin-user-cell strong {
  color: #222c40;
  font-size: 12px;
}

.admin-product-cell small,
.admin-user-cell small {
  color: #7b8598;
  font-size: 11px;
}

.admin-stock,
.admin-bestseller,
.admin-status {
  display: inline-flex;
  align-items: center;
  min-height: 24px;
  padding: 3px 8px;
  border-radius: 4px;
  background: #eff8f2;
  color: #277146;
  font-size: 11px;
  font-weight: 600;
}

.admin-stock.low {
  background: #fff4e8;
  color: #a75e12;
}

.admin-bestseller {
  background: #f0f2f7;
  color: #69748a;
}

.admin-bestseller.featured {
  background: #eef0ff;
  color: #3349bd;
}

.admin-row-actions {
  display: flex;
  justify-content: flex-end;
  gap: 5px;
}

.admin-icon-button {
  display: inline-grid;
  place-items: center;
  width: 32px;
  height: 32px;
  border: 1px solid #e1e5ed;
  border-radius: 5px;
  background: #fff;
  color: #47546b;
  font-size: 17px;
}

.admin-icon-button:hover {
  border-color: #aeb8d0;
  background: #f7f8fc;
}

.admin-icon-button.danger {
  color: #a33838;
}

.admin-user-avatar {
  display: grid;
  place-items: center;
  flex: 0 0 34px;
  height: 34px;
  border-radius: 50%;
  background: #edf0ff;
  color: #3549bc;
  font-size: 13px;
  font-weight: 700;
}

.admin-status.positive {
  background: #eff8f2;
  color: #277146;
}

.admin-status.muted {
  background: #f0f1f4;
  color: #6b7280;
}

.admin-select {
  min-height: 36px;
  padding: 0 9px;
  border: 1px solid #dce1ea;
  border-radius: 5px;
  background: #fff;
  color: #283349;
  font: inherit;
  font-size: 12px;
}

.admin-select.compact {
  min-height: 31px;
}

.admin-text-button {
  border: 0;
  background: transparent;
  color: #a33b3b;
  font-size: 11px;
  font-weight: 600;
}

.admin-text-button.reactivate {
  color: #277146;
}

.admin-self-label {
  color: #8992a2;
  font-size: 11px;
}

.admin-empty-row {
  padding: 32px !important;
  color: #7a8497 !important;
  text-align: center;
}

.admin-orders-list {
  border-top: 1px solid #e8ebf1;
}

.admin-order-row {
  display: grid;
  grid-template-columns: minmax(210px, 1fr) auto 180px;
  align-items: center;
  gap: 24px;
  padding: 16px 8px;
  border-bottom: 1px solid #edf0f4;
}

.admin-order-main,
.admin-order-total,
.admin-order-status {
  display: grid;
  gap: 5px;
}

.admin-order-main strong,
.admin-order-total strong {
  color: #222c40;
  font-size: 13px;
}

.admin-order-main span,
.admin-order-main small,
.admin-order-total span,
.admin-order-status span {
  color: #758096;
  font-size: 11px;
}

.admin-order-status {
  font-style: normal;
}

.admin-empty-state {
  display: grid;
  justify-items: center;
  gap: 8px;
  padding: 54px 20px;
  color: #68748a;
  text-align: center;
}

.admin-empty-state > span {
  font-size: 28px;
}

.admin-empty-state strong {
  color: #263248;
  font-size: 15px;
}

.admin-empty-state p {
  font-size: 12px;
}

.admin-dialog-backdrop {
  position: fixed;
  inset: 0;
  z-index: 60;
  display: grid;
  place-items: center;
  overflow-y: auto;
  padding: 20px;
  background: #10182b80;
}

.admin-dialog {
  width: min(560px, 100%);
  max-height: min(90vh, 760px);
  overflow-y: auto;
  padding: 24px;
  border-radius: 8px;
  background: #fff;
  box-shadow: 0 20px 70px #10182b30;
}

.admin-dialog-heading {
  margin-bottom: 20px;
}

.admin-dialog-heading h2 {
  margin-top: 4px;
  color: #172033;
  font-size: 21px;
}

.admin-product-form {
  display: grid;
  gap: 14px;
}

.admin-product-form label {
  display: grid;
  gap: 6px;
  color: #47546b;
  font-size: 12px;
  font-weight: 600;
}

.admin-product-form input,
.admin-product-form select,
.admin-product-form textarea {
  width: 100%;
  min-height: 39px;
  padding: 9px 10px;
  border: 1px solid #dce1ea;
  border-radius: 5px;
  background: #fff;
  color: #202b40;
  font: inherit;
  font-size: 13px;
}

.admin-product-form textarea {
  resize: vertical;
}

.admin-form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.admin-product-form .admin-check-field {
  display: flex;
  align-items: center;
  align-self: end;
  gap: 9px;
  min-height: 39px;
  padding: 0 4px;
}

.admin-check-field input {
  width: 16px;
  min-height: 16px;
  accent-color: #2438d2;
}

.admin-dialog-actions {
  display: flex;
  justify-content: flex-end;
  gap: 9px;
  padding-top: 5px;
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
}

.admin-page > .admin-heading,
.admin-page > .admin-metrics,
.admin-page > .admin-workspace {
  width: min(1200px, calc(100% - 48px));
  margin-right: auto;
  margin-left: auto;
}

.admin-heading {
  margin-top: 30px;
}

@media (max-width: 720px) {
  .admin-page {
    width: 100%;
    padding-bottom: 48px;
  }

  .admin-header {
    flex-wrap: wrap;
    gap: 8px 14px;
    padding: 10px 14px 0;
  }

  .admin-brand {
    margin-right: auto;
  }

  .admin-header .admin-tabs {
    order: 3;
    flex: 1 1 100%;
    justify-content: flex-start;
    gap: 12px;
    overflow-x: auto;
  }

  .admin-header .admin-tabs button {
    flex: 0 0 auto;
  }

  .admin-header-actions {
    gap: 6px;
  }

  .admin-page > .admin-heading,
  .admin-page > .admin-metrics,
  .admin-page > .admin-workspace {
    width: calc(100% - 28px);
  }

  .admin-heading h1 {
    font-size: 25px;
  }

  .admin-metric {
    padding: 14px 12px;
  }

  .admin-metric-label,
  .admin-metric small {
    font-size: 10px;
  }

  .admin-metric strong {
    font-size: 21px;
  }

  .admin-panel {
    padding: 16px 12px;
  }

  .admin-panel-heading {
    align-items: flex-start;
  }

  .admin-panel-heading p {
    max-width: 230px;
  }

  .admin-primary-button {
    min-height: 36px;
    padding: 0 10px;
    white-space: nowrap;
    font-size: 11px;
  }

  .admin-order-row {
    grid-template-columns: 1fr 1fr;
    gap: 13px;
  }

  .admin-order-main {
    grid-column: 1 / -1;
  }

  .admin-order-status {
    justify-self: end;
  }
}

@media (max-width: 440px) {
  .admin-account-name {
    max-width: 70px;
    font-size: 11px;
  }

  .admin-header-actions .admin-store-link,
  .admin-logout-button {
    padding: 8px;
    font-size: 10px;
  }

  .admin-heading {
    align-items: flex-start;
  }

  .admin-store-link {
    padding: 8px 9px;
    font-size: 11px;
    white-space: nowrap;
  }

  .admin-metrics {
    grid-template-columns: 1fr;
  }

  .admin-metric {
    grid-template-columns: 1fr auto;
    align-items: center;
    border-right: 0;
    border-bottom: 1px solid #e1e5ed;
  }

  .admin-metric:last-child {
    border-bottom: 0;
  }

  .admin-metric strong {
    grid-column: 2;
    grid-row: 1 / span 2;
  }

  .admin-form-row {
    grid-template-columns: 1fr;
  }
}

/* DETAIL PESANAN */
.admin-order-actions {
  display: flex;
  justify-content: flex-end;
}

.admin-detail-button {
  min-height: 35px;
  padding: 0 12px;
  border: 1px solid #2438d2;
  border-radius: 5px;
  background: #fff;
  color: #2438d2;
  font-size: 11px;
  font-weight: 600;
  white-space: nowrap;
  cursor: pointer;
}

.admin-detail-button:hover {
  background: #f0f2ff;
}

.admin-order-dialog {
  width: min(650px, 100%);
}

.admin-order-number {
  display: inline-block;
  margin-top: 6px;
  color: #7b8598;
  font-size: 12px;
}

.admin-detail-status {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  padding: 13px 15px;
  border-radius: 6px;
  background: #f0f2ff;
  color: #2438d2;
  font-size: 12px;
}

.admin-detail-status strong {
  font-size: 12px;
}

.admin-detail-section {
  display: grid;
  gap: 13px;
  padding: 19px 0;
  border-bottom: 1px solid #edf0f4;
}

.admin-detail-section h3 {
  color: #263248;
  font-size: 14px;
}

.admin-detail-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
}

.admin-detail-grid > div {
  display: grid;
  gap: 5px;
}

.admin-detail-grid span,
.admin-detail-address,
.admin-payment-row span {
  color: #7b8598;
  font-size: 11px;
}

.admin-detail-grid strong {
  color: #263248;
  font-size: 12px;
  overflow-wrap: anywhere;
}

.admin-detail-address {
  color: #465269;
  line-height: 1.7;
}

.admin-detail-product {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 0;
  border-bottom: 1px solid #f0f1f5;
}

.admin-detail-product:last-of-type {
  border-bottom: 0;
}

.admin-detail-product img {
  width: 55px;
  height: 55px;
  flex: 0 0 55px;
  border-radius: 6px;
  background: #f1f3f7;
  object-fit: cover;
}

.admin-detail-product-info {
  display: grid;
  flex: 1;
  min-width: 0;
  gap: 4px;
}

.admin-detail-product-info strong {
  color: #263248;
  font-size: 12px;
}

.admin-detail-product-info span,
.admin-detail-product-info small {
  color: #7b8598;
  font-size: 11px;
}

.admin-detail-subtotal {
  color: #263248;
  font-size: 12px;
  white-space: nowrap;
}

.admin-detail-empty {
  color: #7b8598;
  font-size: 12px;
}

.admin-payment-row,
.admin-payment-total {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

.admin-payment-row strong {
  color: #465269;
  font-size: 12px;
  text-align: right;
}

.admin-payment-total {
  padding-top: 13px;
  border-top: 1px dashed #dce1ea;
}

.admin-payment-total span {
  color: #263248;
  font-size: 13px;
  font-weight: 600;
}

.admin-payment-total strong {
  color: #2438d2;
  font-size: 17px;
}

.admin-order-row {
  grid-template-columns: minmax(210px, 1fr) auto 180px auto;
}

@media (max-width: 720px) {
  .admin-order-row {
    grid-template-columns: 1fr 1fr;
  }

  .admin-order-actions {
    grid-column: 1 / -1;
    justify-content: flex-start;
  }
}

@media (max-width: 440px) {
  .admin-detail-grid {
    grid-template-columns: 1fr;
  }

  .admin-detail-product {
    flex-wrap: wrap;
  }

  .admin-detail-subtotal {
    margin-left: auto;
  }
}

/* CUSTOM ADMIN ALERT & CONFIRMATION */

.admin-alert-backdrop {
  z-index: 100;
  background: rgba(15, 23, 42, 0.48);
  backdrop-filter: blur(3px);
}

.admin-confirm-dialog {
  width: min(390px, 100%);
  padding: 30px 26px 24px;
  border-radius: 12px;
  text-align: center;
  animation: admin-popup-enter 0.2s ease;
}

.admin-alert-icon {
  display: grid;
  place-items: center;
  width: 48px;
  height: 48px;
  margin: 0 auto 17px;
  border-radius: 50%;
  background: #eef0ff;
  color: #2438d2;
  font-size: 23px;
  font-weight: 700;
}

.admin-confirm-dialog h2 {
  margin-bottom: 10px;
  color: #172033;
  font-size: 19px;
}

.admin-confirm-dialog > p {
  margin: 0 auto;
  color: #69758a;
  font-size: 13px;
  line-height: 1.7;
}

.admin-confirm-dialog .admin-dialog-actions {
  justify-content: center;
  margin-top: 20px;
}

.admin-confirm-dialog .admin-primary-button {
  min-width: 115px;
}

@keyframes admin-popup-enter {
  from {
    opacity: 0;
    transform: translateY(10px) scale(0.97);
  }

  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

/* =========================================
   ADMIN DASHBOARD - MODERN UI IMPROVEMENT
========================================= */

.admin-page {
  min-height: 100vh;
  padding-bottom: 60px;
  background: #f6f8fc;
  color: #172033;
  font-family: inherit;
}

/* HEADER */
.admin-header {
  position: sticky;
  top: 0;
  z-index: 30;
  min-height: 76px;
  padding: 10px max(28px, calc((100% - 1320px) / 2));
  gap: 30px;
  background: rgba(255, 255, 255, 0.96);
  border-bottom: 1px solid #e8edf5;
  box-shadow: 0 3px 15px rgba(20, 35, 70, 0.035);
  backdrop-filter: blur(12px);
}

.admin-brand-mark {
  width: 44px;
  height: 44px;
  border-radius: 14px;
  background: linear-gradient(135deg, #3048ed, #182bb9);
  box-shadow: 0 5px 12px rgba(36, 56, 210, 0.2);
}

.admin-brand-name strong {
  color: #172554;
  font-size: 15px;
  font-weight: 800;
}

.admin-header .admin-tabs {
  gap: 12px;
}

.admin-header .admin-tabs button {
  padding: 12px 13px 10px;
  border-radius: 9px 9px 0 0;
  transition: 0.2s ease;
}

.admin-header .admin-tabs button:hover {
  color: #2438d2;
  background: #f5f6ff;
}

.admin-header .admin-tabs button.active {
  color: #2438d2;
  background: #f1f3ff;
}

.admin-header .admin-tabs button span {
  background: #edf0ff;
  color: #3449c8;
  font-weight: 700;
}

.admin-account-name {
  color: #263248;
  font-size: 13px;
  font-weight: 700;
}

.admin-logout-button {
  min-height: 39px;
  padding: 0 17px;
  border-radius: 9px;
  background: #ef4444;
  transition: 0.2s ease;
}

.admin-logout-button:hover {
  background: #dc2626;
  transform: translateY(-1px);
}

/* PAGE HEADING */
.admin-page > .admin-heading,
.admin-page > .admin-metrics,
.admin-page > .admin-workspace {
  width: min(1320px, calc(100% - 64px));
}

.admin-heading {
  margin-top: 38px;
  margin-bottom: 28px;
}

.admin-eyebrow {
  color: #3449d6;
  font-size: 11px;
  letter-spacing: 0.13em;
}

.admin-heading h1 {
  margin-top: 8px;
  color: #172554;
  font-size: 32px;
  font-weight: 800;
  letter-spacing: -0.7px;
}

.admin-subtitle {
  margin-top: 8px;
  color: #718096;
  font-size: 14px;
}

/* STATISTIC CARDS */
.admin-metrics {
  gap: 16px;
  border: 0;
  background: transparent;
  margin-bottom: 28px;
}

.admin-metric {
  position: relative;
  min-height: 120px;
  padding: 22px 24px;
  gap: 7px;
  border: 1px solid #e5eaf4;
  border-radius: 14px;
  background: #fff;
  box-shadow: 0 4px 14px rgba(25, 45, 90, 0.035);
  transition:
    transform 0.2s ease,
    box-shadow 0.2s ease;
}

.admin-metric:hover {
  transform: translateY(-3px);
  box-shadow: 0 9px 22px rgba(25, 45, 90, 0.07);
}

.admin-metric:nth-child(1) {
  border-top: 3px solid #4263eb;
}

.admin-metric:nth-child(2) {
  border-top: 3px solid #10b981;
}

.admin-metric:nth-child(3) {
  border-top: 3px solid #bd4bff;
}

.admin-metric:last-child {
  border-right: 1px solid #e5eaf4;
}

.admin-metric-label {
  color: #718096;
  font-size: 13px;
  font-weight: 500;
}

.admin-metric strong {
  color: #172554;
  font-size: 29px;
  font-weight: 800;
}

.admin-metric small {
  color: #8a96aa;
  font-size: 12px;
}

/* MAIN WORKSPACE */
.admin-workspace {
  border: 1px solid #e5eaf4;
  border-radius: 15px;
  background: #fff;
  box-shadow: 0 5px 22px rgba(25, 45, 90, 0.04);
}

.admin-tabs:not(.admin-header .admin-tabs) {
  padding: 0 25px;
  background: #fff;
}

.admin-tabs button {
  transition: 0.2s ease;
}

/* CONTENT PANEL */
.admin-panel {
  padding: 27px;
}

.admin-panel-heading {
  margin-bottom: 23px;
}

.admin-panel-heading h2 {
  color: #172554;
  font-size: 21px;
  font-weight: 800;
}

.admin-panel-heading p {
  color: #8290a5;
  font-size: 13px;
}

/* BUTTONS */
.admin-primary-button {
  min-height: 43px;
  padding: 0 19px;
  border-radius: 9px;
  background: linear-gradient(135deg, #344bea, #2438d2);
  box-shadow: 0 4px 10px rgba(36, 56, 210, 0.16);
  transition: 0.2s ease;
}

.admin-primary-button:hover {
  background: linear-gradient(135deg, #263bd4, #1729b5);
  box-shadow: 0 6px 14px rgba(36, 56, 210, 0.23);
  transform: translateY(-1px);
}

.admin-secondary-button {
  border-radius: 9px;
  transition: 0.2s ease;
}

.admin-secondary-button:hover {
  border-color: #aebaf0;
  background: #f6f7ff;
}

/* SEARCH */
.admin-search {
  width: min(430px, 100%);
  height: 43px;
  margin-bottom: 19px;
  padding: 0 14px;
  border: 1px solid #e0e6f0;
  border-radius: 9px;
  background: #fff;
  transition: 0.2s ease;
}

.admin-search:focus-within {
  border-color: #6678ed;
  box-shadow: 0 0 0 3px rgba(52, 73, 234, 0.09);
}

.admin-search input {
  font-size: 13px;
}

/* TABLE */
.admin-table-wrap {
  border: 1px solid #e9edf5;
  border-radius: 11px;
  background: #fff;
}

.admin-table {
  border-collapse: separate;
  border-spacing: 0;
}

.admin-table th {
  padding: 15px 14px;
  border-top: 0;
  border-bottom: 1px solid #e8edf5;
  background: #f7f9fd;
  color: #66748a;
  font-size: 12px;
  font-weight: 700;
}

.admin-table th:first-child {
  border-radius: 10px 0 0 0;
}

.admin-table th:last-child {
  border-radius: 0 10px 0 0;
}

.admin-table td {
  padding: 14px;
  border-bottom: 1px solid #edf1f6;
  color: #526078;
  font-size: 12px;
  transition: background 0.15s ease;
}

.admin-table tbody tr:hover td {
  background: #fafbff;
}

.admin-table tbody tr:last-child td {
  border-bottom: 0;
}

.admin-product-cell img {
  width: 48px;
  height: 48px;
  border: 1px solid #edf0f5;
  border-radius: 9px;
  background: #f7f8fb;
}

.admin-product-cell strong,
.admin-user-cell strong {
  color: #25324b;
  font-size: 12px;
  font-weight: 700;
}

/* BADGES */
.admin-stock,
.admin-bestseller,
.admin-status {
  min-height: 26px;
  padding: 4px 10px;
  border-radius: 7px;
  font-size: 11px;
  font-weight: 700;
}

.admin-stock {
  background: #eaf8f0;
  color: #16834c;
}

.admin-stock.low {
  background: #fff3e5;
  color: #c16b12;
}

.admin-bestseller.featured {
  background: #edf0ff;
  color: #3449d6;
}

.admin-status.positive {
  background: #eaf8f0;
  color: #16834c;
}

.admin-status.muted {
  background: #f1f3f7;
  color: #69758a;
}

/* ACTION BUTTONS */
.admin-row-actions {
  gap: 7px;
}

.admin-icon-button {
  width: 35px;
  height: 35px;
  border-radius: 9px;
  color: #3449c8;
  font-size: 16px;
  transition: 0.2s ease;
}

.admin-icon-button:hover {
  border-color: #b9c3f5;
  background: #f0f2ff;
  color: #2438d2;
  transform: translateY(-1px);
}

.admin-icon-button.danger {
  color: #e34b55;
}

.admin-icon-button.danger:hover {
  border-color: #fecaca;
  background: #fff1f2;
  color: #dc2626;
}

/* ORDER LIST */
.admin-order-row {
  padding: 19px 12px;
  border-bottom: 1px solid #edf1f6;
  transition: background 0.2s ease;
}

.admin-order-row:hover {
  background: #fafbff;
}

.admin-order-main strong,
.admin-order-total strong {
  color: #25324b;
  font-size: 13px;
  font-weight: 700;
}

.admin-detail-button {
  min-height: 36px;
  padding: 0 14px;
  border-radius: 8px;
  transition: 0.2s ease;
}

.admin-detail-button:hover {
  background: #edf0ff;
  transform: translateY(-1px);
}

/* EMPTY STATE */
.admin-empty-state {
  padding: 65px 20px;
  color: #8490a5;
}

.admin-empty-state > span {
  display: grid;
  place-items: center;
  width: 58px;
  height: 58px;
  border-radius: 18px;
  background: #f0f2ff;
  color: #3449d6;
  font-size: 27px;
}

.admin-empty-state strong {
  color: #263653;
  font-size: 16px;
}

/* DIALOG */
.admin-dialog {
  border: 1px solid #e9edf5;
  border-radius: 15px;
  box-shadow: 0 24px 75px rgba(15, 23, 42, 0.2);
}

.admin-dialog-heading h2 {
  color: #172554;
  font-weight: 800;
}

.admin-product-form label {
  color: #47546b;
  font-size: 12px;
  font-weight: 700;
}

.admin-product-form input,
.admin-product-form select,
.admin-product-form textarea {
  min-height: 42px;
  border: 1px solid #e0e6f0;
  border-radius: 8px;
  transition: 0.2s ease;
}

.admin-product-form input:focus,
.admin-product-form select:focus,
.admin-product-form textarea:focus {
  border-color: #6678ed;
  outline: none;
  box-shadow: 0 0 0 3px rgba(52, 73, 234, 0.09);
}

/* ORDER DETAIL */
.admin-order-dialog {
  border-radius: 15px;
}

.admin-detail-status {
  border: 1px solid #e0e5ff;
  border-radius: 9px;
}

.admin-detail-section {
  padding: 21px 0;
}

.admin-detail-section h3 {
  color: #25324b;
  font-weight: 700;
}

.admin-detail-product img {
  border: 1px solid #edf0f5;
  border-radius: 9px;
}

/* SCROLLBAR */
.admin-table-wrap::-webkit-scrollbar {
  height: 7px;
}

.admin-table-wrap::-webkit-scrollbar-track {
  background: #f1f3f8;
  border-radius: 10px;
}

.admin-table-wrap::-webkit-scrollbar-thumb {
  background: #c6cde0;
  border-radius: 10px;
}

.admin-table-wrap::-webkit-scrollbar-thumb:hover {
  background: #8997bd;
}

/* RESPONSIVE */
@media (max-width: 900px) {
  .admin-header {
    gap: 14px;
    padding-right: 18px;
    padding-left: 18px;
  }

  .admin-header .admin-tabs {
    gap: 5px;
  }

  .admin-header .admin-tabs button {
    padding-right: 8px;
    padding-left: 8px;
  }

  .admin-page > .admin-heading,
  .admin-page > .admin-metrics,
  .admin-page > .admin-workspace {
    width: calc(100% - 36px);
  }
}

@media (max-width: 720px) {
  .admin-header {
    position: relative;
    padding: 12px 14px 0;
  }

  .admin-header .admin-tabs {
    padding-bottom: 4px;
  }

  .admin-heading {
    margin-top: 25px;
  }

  .admin-heading h1 {
    font-size: 26px;
  }

  .admin-metrics {
    gap: 10px;
  }

  .admin-metric {
    border-radius: 11px;
  }

  .admin-panel {
    padding: 18px 13px;
  }

  .admin-workspace {
    border-radius: 12px;
  }

  .admin-table {
    min-width: 700px;
  }
}

@media (max-width: 440px) {
  .admin-page > .admin-heading,
  .admin-page > .admin-metrics,
  .admin-page > .admin-workspace {
    width: calc(100% - 24px);
  }

  .admin-metrics {
    gap: 8px;
  }

  .admin-metric {
    min-height: 80px;
    padding: 13px;
  }

  .admin-heading h1 {
    font-size: 23px;
  }

  .admin-panel-heading {
    flex-wrap: wrap;
  }

  .admin-panel-heading > * {
    max-width: 100%;
  }

  .admin-dialog {
    padding: 19px;
  }
}
</style>
