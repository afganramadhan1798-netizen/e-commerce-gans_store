<script setup>
import { computed, ref } from "vue";

const props = defineProps({
  products: {
    type: Array,
    required: true,
  },
  users: {
    type: Array,
    required: true,
  },
  orders: {
    type: Array,
    required: true,
  },
  categories: {
    type: Array,
    required: true,
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
]);

const activeTab = ref("products");
const productSearch = ref("");
const productDialogOpen = ref(false);
const editingProductId = ref(null);
const productForm = ref(createEmptyProduct());

const categoryOptions = computed(() =>
  props.categories.filter((category) => category !== "Best Seller"),
);

const uniqueProducts = computed(() =>
  [
    ...new Map(props.products.map((product) => [product.id, product])).values(),
  ].map((product) => ({ ...product, stock: product.stock ?? 10 })),
);

const filteredProducts = computed(() => {
  const query = productSearch.value.trim().toLowerCase();
  if (!query) return uniqueProducts.value;

  return uniqueProducts.value.filter((product) =>
    `${product.name} ${product.variant || ""} ${product.category}`
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
  productDialogOpen.value = true;
}

function openEditProduct(product) {
  editingProductId.value = product.id;
  productForm.value = { ...createEmptyProduct(), ...product };
  productDialogOpen.value = true;
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
  if (
    window.confirm(
      `Hapus ${product.name}${product.variant ? ` - ${product.variant}` : ""}?`,
    )
  ) {
    emit("delete-product", product.id);
  }
}

function setOrderStatus(orderId, status) {
  emit("update-order-status", { orderId, status });
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
          :class="{ active: activeTab === 'users' }"
          @click="activeTab = 'users'"
        >
          Pengguna <span>{{ users.length }}</span>
        </button>
        <button
          :class="{ active: activeTab === 'orders' }"
          @click="activeTab = 'orders'"
        >
          Pesanan <span>{{ orders.length }}</span>
        </button>
      </nav>

      <div class="admin-header-actions">
        <span class="admin-account-name" :title="userName">{{ userName }}</span>
        <button class="admin-store-link" @click="emit('home')">
          ← Lihat Toko
        </button>
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
                    <img :src="product.image" :alt="product.name" />
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
                    class="admin-select compact"
                    :value="user.role"
                    :disabled="user.isCurrentUser"
                    aria-label="Peran pengguna"
                    @change="
                      emit('update-user-role', {
                        userId: user.id,
                        role: $event.target.value,
                      })
                    "
                  >
                    <option value="customer">Pelanggan</option>
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

      <div v-else class="admin-panel">
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
              <small
                >{{ order.date }} · {{ order.items.length }} jenis produk</small
              >
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
          </article>
        </div>
        <div v-else class="admin-empty-state">
          <span>▤</span>
          <strong>Belum ada pesanan</strong>
          <p>Pesanan pelanggan akan muncul di sini.</p>
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
          <label>
            URL gambar
            <input
              v-model.trim="productForm.image"
              type="text"
              placeholder="https://... atau /images/..."
              required
            />
          </label>
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
</style>
