
<script setup>
import { computed } from "vue";

const props = defineProps({
  modelValue: {
    type: Boolean,
    default: false,
  },
  type: {
    type: String,
    default: "success",
  },
  title: {
    type: String,
    default: "Informasi",
  },
  message: {
    type: String,
    default: "",
  },
  confirmText: {
    type: String,
    default: "Mengerti",
  },
  cancelText: {
    type: String,
    default: "Batal",
  },
  showCancel: {
    type: Boolean,
    default: false,
  },
});

const emit = defineEmits(["update:modelValue", "confirm", "cancel"]);

const icon = computed(() => {
  const icons = {
    success: "✓",
    error: "×",
    warning: "!",
    info: "i",
    confirm: "?",
  };

  return icons[props.type] || "i";
});

function closePopup() {
  emit("update:modelValue", false);
  emit("cancel");
}

function confirmPopup() {
  emit("update:modelValue", false);
  emit("confirm");
}
</script>

<template>
  <Transition name="popup">
    <div
      v-if="modelValue"
      class="popup-overlay"
      @click.self="closePopup"
      @keydown.esc="closePopup"
    >
      <div
        class="popup-card"
        role="dialog"
        aria-modal="true"
        :aria-label="title"
      >
        <div class="popup-icon" :class="`popup-${type}`">
          {{ icon }}
        </div>

        <h3 class="popup-title">{{ title }}</h3>

        <p class="popup-message">{{ message }}</p>

        <div class="popup-actions">
          <button
            v-if="showCancel"
            class="popup-cancel-btn"
            @click="closePopup"
          >
            {{ cancelText }}
          </button>

          <button
            class="popup-confirm-btn"
            :class="`popup-button-${type}`"
            @click="confirmPopup"
          >
            {{ confirmText }}
          </button>
        </div>
      </div>
    </div>
  </Transition>
</template>

<style scoped>
.popup-overlay {
  position: fixed;
  inset: 0;
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  background: rgba(15, 23, 42, 0.48);
  backdrop-filter: blur(4px);
}

.popup-card {
  width: 100%;
  max-width: 390px;
  padding: 32px 28px 26px;
  border: 1px solid #e8edf5;
  border-radius: 20px;
  background: #ffffff;
  box-shadow: 0 20px 60px rgba(15, 23, 42, 0.16);
  text-align: center;
}

.popup-icon {
  width: 58px;
  height: 58px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 18px;
  border-radius: 50%;
  font-size: 28px;
  font-weight: 700;
}

.popup-success {
  color: #16845b;
  background: #e8f7f0;
}

.popup-error {
  color: #dc4545;
  background: #fff0f0;
}

.popup-warning {
  color: #c98516;
  background: #fff6e5;
}

.popup-info {
  color: #2864d8;
  background: #edf3ff;
}

.popup-confirm {
  color: #2864d8;
  background: #edf3ff;
}

.popup-title {
  margin: 0 0 10px;
  color: #17233c;
  font-size: 20px;
  font-weight: 700;
}

.popup-message {
  margin: 0;
  color: #64748b;
  font-size: 14px;
  line-height: 1.7;
  white-space: pre-line;
}

.popup-actions {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-top: 26px;
}

.popup-actions button {
  min-width: 115px;
  padding: 12px 18px;
  border: none;
  border-radius: 10px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: 0.2s ease;
}

.popup-cancel-btn {
  color: #475569;
  background: #f1f5f9;
}

.popup-cancel-btn:hover {
  background: #e2e8f0;
}

.popup-confirm-btn {
  color: white;
  background: #2864d8;
}

.popup-confirm-btn:hover {
  background: #1e50b5;
}

.popup-button-success {
  background: #2864d8;
}

.popup-button-error {
  background: #dc4545;
}

.popup-button-warning {
  background: #c98516;
}

.popup-enter-active,
.popup-leave-active {
  transition: opacity 0.2s ease;
}

.popup-enter-active .popup-card,
.popup-leave-active .popup-card {
  transition: transform 0.2s ease;
}

.popup-enter-from,
.popup-leave-to {
  opacity: 0;
}

.popup-enter-from .popup-card,
.popup-leave-to .popup-card {
  transform: translateY(12px) scale(0.97);
}

@media (max-width: 480px) {
  .popup-card {
    padding: 28px 20px 22px;
  }

  .popup-actions {
    flex-direction: column-reverse;
  }

  .popup-actions button {
    width: 100%;
  }
}
</style>
