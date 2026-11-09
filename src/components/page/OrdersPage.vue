<script setup>
import { ref } from "vue";

defineProps({
  orders: {
    type: Array,
    required: true,
  },
  formatPrice: {
    type: Function,
    required: true,
  },
});

const emit = defineEmits(["home", "cancel"]);

const selectedOrder = ref(null);

function openOrderDetail(order) {
  selectedOrder.value = order;
}

function closeOrderDetail() {
  selectedOrder.value = null;
}

function formatDate(date) {
  if (!date) return "-";

  return new Date(date).toLocaleString("id-ID", {
    day: "2-digit",
    month: "long",
    year: "numeric",
    hour: "2-digit",
    minute: "2-digit",
  });
}

function getStatusClass(status) {
  const statusClasses = {
    "Pesanan Dibuat": "status-created",
    Diproses: "status-processing",
    Dikirim: "status-shipped",
    Selesai: "status-completed",
    Dibatalkan: "status-cancelled",
  };

  return statusClasses[status] || "status-created";
}

const orderSteps = [
  {
    title: "Pesanan Dibuat",
    description: "Pesanan berhasil diterima oleh Gan's Store.",
  },
  {
    title: "Diproses",
    description: "Pesanan sedang disiapkan oleh penjual.",
  },
  {
    title: "Dikirim",
    description: "Pesanan sedang dalam perjalanan menuju alamatmu.",
  },
  {
    title: "Selesai",
    description: "Pesanan telah berhasil diselesaikan.",
  },
];

function getCurrentStep(status) {
  const statusIndex = orderSteps.findIndex((step) => step.title === status);

  return statusIndex === -1 ? 0 : statusIndex;
}
</script>

<template>
  <main class="main-content orders-page">
    <div class="orders-heading">
      <div>
        <h2>Pesanan Saya</h2>
        <p>Lihat dan pantau semua pesananmu di Gan's Store.</p>
      </div>
      <button class="back-home-btn" @click="emit('home')">
        ← Kembali Belanja
      </button>
    </div>

    <div v-if="orders.length" class="orders-list">
      <!-- Modal Detail Pesanan -->
      <div
        v-if="selectedOrder"
        class="order-modal-overlay"
        @click.self="closeOrderDetail"
      >
        <div class="order-modal">
          <div class="order-modal-header">
            <div>
              <h3>Detail Pesanan</h3>
              <p>{{ selectedOrder.orderNumber || selectedOrder.id }}</p>
            </div>

            <button class="close-order-modal" @click="closeOrderDetail">
              ×
            </button>
          </div>

          <div class="order-modal-body">
            <div class="detail-section">
              <span class="detail-label">Status Pesanan</span>
              <span
                class="order-status"
                :class="getStatusClass(selectedOrder.status)"
              >
                {{ selectedOrder.status }}
              </span>
            </div>

            <!-- Timeline Pelacakan Pesanan -->
            <div class="tracking-section">
              <h4>Pelacakan Pesanan</h4>

              <!-- Jika pesanan dibatalkan -->
              <div
                v-if="selectedOrder.status === 'Dibatalkan'"
                class="tracking-cancelled"
              >
                <div class="cancelled-icon">!</div>

                <div>
                  <strong>Pesanan Dibatalkan</strong>
                  <p>
                    Pesanan ini telah dibatalkan dan tidak dilanjutkan ke tahap
                    pengiriman.
                  </p>
                </div>
              </div>

              <!-- Jika pesanan masih berjalan -->
              <div v-else class="tracking-timeline">
                <div
                  v-for="(step, index) in orderSteps"
                  :key="step.title"
                  class="tracking-step"
                  :class="{
                    completed: index < getCurrentStep(selectedOrder.status),
                    current: index === getCurrentStep(selectedOrder.status),
                  }"
                >
                  <div class="tracking-indicator">
                    <div class="tracking-dot">
                      <span v-if="index < getCurrentStep(selectedOrder.status)">
                        ✓
                      </span>
                      <span v-else>{{ index + 1 }}</span>
                    </div>

                    <div
                      v-if="index !== orderSteps.length - 1"
                      class="tracking-line"
                    ></div>
                  </div>

                  <div class="tracking-content">
                    <strong>{{ step.title }}</strong>
                    <p>{{ step.description }}</p>

                    <span
                      v-if="index === getCurrentStep(selectedOrder.status)"
                      class="tracking-current-label"
                    >
                      Status saat ini
                    </span>
                  </div>
                </div>
              </div>
            </div>

            <div class="detail-section">
              <span class="detail-label">Tanggal Pesanan</span>
              <strong>{{ formatDate(selectedOrder.date) }}</strong>
            </div>

            <div class="detail-section">
              <span class="detail-label">Produk Dipesan</span>

              <div
                v-for="(product, index) in selectedOrder.items"
                :key="product.id + '-' + index"
                class="detail-product"
              >
                <img :src="product.image" :alt="product.name" />

                <div>
                  <strong>{{ product.name }}</strong>
                  <p v-if="product.variant">{{ product.variant }}</p>
                  <p>
                    {{ product.quantity }} × {{ formatPrice(product.price) }}
                  </p>
                </div>

                <strong>
                  {{ formatPrice(product.price * product.quantity) }}
                </strong>
              </div>
            </div>

            <div class="detail-section">
              <span class="detail-label">Alamat Pengiriman</span>
              <strong>{{ selectedOrder.address.recipient }}</strong>
              <p>{{ selectedOrder.address.phone }}</p>
              <p>
                {{ selectedOrder.address.detail }},
                {{ selectedOrder.address.district }},
                {{ selectedOrder.address.city }},
                {{ selectedOrder.address.province }},
                {{ selectedOrder.address.postalCode }}
              </p>
            </div>

            <div class="detail-section">
              <span class="detail-label">Metode Pembayaran</span>
              <strong>{{ selectedOrder.paymentMethod }}</strong>
            </div>

            <div class="detail-total">
              <span>Total Pembayaran</span>
              <strong>{{ formatPrice(selectedOrder.total) }}</strong>
            </div>
          </div>

          <button class="close-detail-btn" @click="closeOrderDetail">
            Tutup
          </button>
        </div>
      </div>

      <article v-for="order in orders" :key="order.id" class="order-card">
        <div class="order-card-header">
          <div>
            <span class="order-label">Nomor Pesanan</span>
            <strong>{{ order.orderNumber || order.id }}</strong>
          </div>
          <div class="order-card-actions">
            <button class="detail-order-btn" @click="openOrderDetail(order)">
              Lihat Detail Pesanan
            </button>
          </div>
          <span class="order-status" :class="getStatusClass(order.status)">
            {{ order.status }}
          </span>
        </div>

        <div class="order-date">
          {{ formatDate(order.date) }}
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
          <p>{{ order.address.recipient }} · {{ order.address.phone }}</p>
          <p>
            {{ order.address.detail }}, {{ order.address.district }},
            {{ order.address.city }}, {{ order.address.province }},
            {{ order.address.postalCode }}
          </p>
        </div>

        <div class="order-bottom-actions">
          <button
            v-if="order.status === 'Pesanan Dibuat'"
            class="cancel-order-btn"
            @click="emit('cancel', order.id)"
          >
            Batalkan Pesanan
          </button>
        </div>
      </article>
    </div>

    <div v-else class="orders-empty">
      <div class="orders-empty-icon">📦</div>
      <h3>Belum Ada Pesanan</h3>
      <p>
        Kamu belum memiliki riwayat pesanan. Yuk, temukan produk menarik di
        Gan's Store!
      </p>
      <button class="back-home-btn" @click="emit('home')">Mulai Belanja</button>
    </div>
  </main>
</template>

<style scoped>
.order-status {
  padding: 7px 13px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 600;
}

.status-created {
  background: #eef2ff;
  color: #4f46e5;
  text-align: center;
}

.status-processing {
  background: #fff7ed;
  color: #c2410c;
}

.status-shipped {
  background: #eff6ff;
  color: #2563eb;
}

.status-completed {
  background: #ecfdf5;
  color: #059669;
}

.status-cancelled {
  background: #fef2f2;
  color: #dc2626;
}

.order-card-actions {
  display: flex;
  justify-content: flex-end;
  align-items: flex-end;
  margin-top: 18px;
  width: 100%;
}

.detail-order-btn {
  padding: 10px 18px;
  border: 1px solid #2563eb;
  border-radius: 8px;
  background: white;
  color: #2563eb;
  font-weight: 600;
  cursor: pointer;
}

.detail-order-btn:hover {
  background: #eff6ff;
}

.order-modal-overlay {
  position: fixed;
  inset: 0;
  z-index: 1000;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
  background: rgba(15, 23, 42, 0.55);
}

.order-modal {
  width: 100%;
  max-width: 600px;
  max-height: 85vh;
  overflow-y: auto;
  padding: 24px;
  border-radius: 16px;
  background: white;
}

.order-modal-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding-bottom: 18px;
  border-bottom: 1px solid #e5e7eb;
}

.order-modal-header h3 {
  margin: 0 0 6px;
}

.order-modal-header p {
  margin: 0;
  color: #64748b;
  font-size: 13px;
}

.close-order-modal {
  border: none;
  background: none;
  font-size: 28px;
  cursor: pointer;
  color: #64748b;
}

.order-modal-body {
  display: flex;
  flex-direction: column;
  gap: 20px;
  padding: 20px 0;
}

.detail-section {
  display: flex;
  flex-direction: column;
  gap: 7px;
}

.detail-label {
  color: #64748b;
  font-size: 13px;
}

.detail-section p {
  margin: 0;
  color: #475569;
  font-size: 13px;
  line-height: 1.6;
}

.detail-product {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 0;
  border-bottom: 1px solid #f1f5f9;
}

.detail-product img {
  width: 60px;
  height: 60px;
  object-fit: cover;
  border-radius: 8px;
}

.detail-product div {
  flex: 1;
}

.detail-product strong {
  font-size: 13px;
}

.detail-product > strong {
  white-space: nowrap;
}

.detail-total {
  display: flex;
  justify-content: space-between;
  padding-top: 18px;
  border-top: 1px solid #e5e7eb;
}

.detail-total strong {
  color: #2563eb;
  font-size: 18px;
}

.close-detail-btn {
  width: 100%;
  padding: 12px;
  border: none;
  border-radius: 8px;
  background: #2563eb;
  color: white;
  font-weight: 600;
  cursor: pointer;
}

.close-detail-btn:hover {
  background: #1d4ed8;
}

.order-bottom-actions {
  display: flex;
  justify-content: flex-end;
  margin-top: 18px;
  padding-top: 15px;
  border-top: 1px solid #f1f5f9;
}

.cancel-order-btn {
  padding: 10px 18px;
  border: 1px solid #dc2626;
  border-radius: 8px;
  background: white;
  color: #dc2626;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  transition: 0.2s;
}

.cancel-order-btn:hover {
  background: #fef2f2;
}

/* Tracking Pesanan */
.tracking-section {
  padding: 18px;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  background: #f8fafc;
}

.tracking-section h4 {
  margin: 0 0 22px;
  color: #1e293b;
  font-size: 15px;
}

.tracking-timeline {
  display: flex;
  flex-direction: column;
}

.tracking-step {
  display: flex;
  gap: 14px;
  min-height: 85px;
}

.tracking-indicator {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 30px;
  flex-shrink: 0;
}

.tracking-dot {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 28px;
  height: 28px;
  border: 2px solid #cbd5e1;
  border-radius: 50%;
  background: white;
  color: #94a3b8;
  font-size: 12px;
  font-weight: 700;
  z-index: 1;
}

.tracking-line {
  flex: 1;
  width: 2px;
  min-height: 45px;
  background: #cbd5e1;
}

.tracking-step.completed .tracking-dot {
  border-color: #16a34a;
  background: #16a34a;
  color: white;
}

.tracking-step.completed .tracking-line {
  background: #16a34a;
}

.tracking-step.current .tracking-dot {
  border-color: #2563eb;
  background: #2563eb;
  color: white;
  box-shadow: 0 0 0 4px #dbeafe;
}

.tracking-content {
  padding-bottom: 22px;
}

.tracking-content strong {
  display: block;
  color: #64748b;
  font-size: 13px;
}

.tracking-content p {
  margin: 5px 0;
  color: #94a3b8;
  font-size: 12px;
  line-height: 1.5;
}

.tracking-step.completed .tracking-content strong,
.tracking-step.current .tracking-content strong {
  color: #1e293b;
}

.tracking-step.completed .tracking-content p,
.tracking-step.current .tracking-content p {
  color: #64748b;
}

.tracking-current-label {
  display: inline-block;
  margin-top: 5px;
  padding: 4px 9px;
  border-radius: 20px;
  background: #dbeafe;
  color: #1d4ed8;
  font-size: 10px;
  font-weight: 700;
}

.tracking-cancelled {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  padding: 14px;
  border: 1px solid #fecaca;
  border-radius: 10px;
  background: #fef2f2;
}

.cancelled-icon {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 28px;
  height: 28px;
  flex-shrink: 0;
  border-radius: 50%;
  background: #dc2626;
  color: white;
  font-weight: 700;
}

.tracking-cancelled strong {
  color: #b91c1c;
  font-size: 13px;
}

.tracking-cancelled p {
  margin: 5px 0 0;
  color: #991b1b;
  font-size: 12px;
  line-height: 1.6;
}
</style>
