
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
    default: "Lanjutkan",
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

const emit = defineEmits([
  "update:modelValue",
  "confirm",
  "cancel",
]);

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
        <!-- ICON -->
        <div
          class="popup-icon"
          :class="`popup-${type}`"
        >
          {{ icon }}
        </div>

        <!-- TITLE -->
        <h3 class="popup-title">
          {{ title }}
        </h3>

        <!-- MESSAGE -->
        <p
          v-if="message"
          class="popup-message"
        >
          {{ message }}
        </p>

        <!-- BUTTONS -->
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
/* OVERLAY */

.popup-overlay {
  position: fixed;
  inset: 0;
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  background: rgba(0, 0, 0, 0.48);
}

/* POPUP CARD */

.popup-card {
  width: 100%;
  max-width: 700px;
  min-height: 400px;
  padding: 38px 30px 40px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  border-radius: 22px;
  background: #ffffff;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.2);
  text-align: center;
}

/* ICON */

.popup-icon {
  width: 134px;
  height: 134px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 32px;
  border: 8px solid currentColor;
  border-radius: 50%;
  font-size: 100px;
  font-family: Arial, sans-serif;
  font-weight: 400;
  line-height: 1;
}

/* SUCCESS */

.popup-success {
  color: #00b83f;
  background: #ffffff;
}

/* ERROR */

.popup-error {
  color: #e53939;
  background: #ffffff;
}

/* WARNING */

.popup-warning {
  color: #e5a000;
  background: #ffffff;
}

/* INFO & CONFIRM */

.popup-info,
.popup-confirm {
  color: #2864d8;
  background: #ffffff;
}

/* TITLE */

.popup-title {
  margin: 0 0 45px;
  color: #111111;
  font-size: 25px;
  font-weight: 600;
  line-height: 1.5;
}

/* MESSAGE */

.popup-message {
  margin: -30px 0 35px;
  color: #555555;
  font-size: 15px;
  line-height: 1.7;
  white-space: pre-line;
}

/* ACTIONS */

.popup-actions {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
}

/* BUTTON BASE */

.popup-actions button {
  min-width: 200px;
  min-height: 70px;
  padding: 12px 25px;
  border: none;
  border-radius: 11px;
  color: #ffffff;
  font-family: inherit;
  font-size: 25px;
  font-weight: 700;
  cursor: pointer;
  box-shadow: 0 3px 5px rgba(0, 0, 0, 0.25);
  transition:
    background 0.2s ease,
    transform 0.2s ease;
}

.popup-actions button:hover {
  transform: translateY(-2px);
}

/* SUCCESS BUTTON */

.popup-button-success {
  background: #00b83f;
}

.popup-button-success:hover {
  background: #009e35;
}

/* ERROR BUTTON */

.popup-button-error {
  background: #e53939;
}

.popup-button-error:hover {
  background: #c62828;
}

/* WARNING BUTTON */

.popup-button-warning {
  background: #e5a000;
}

.popup-button-warning:hover {
  background: #c68a00;
}

/* INFO & CONFIRM BUTTON */

.popup-button-info,
.popup-button-confirm {
  background: #2864d8;
}

.popup-button-info:hover,
.popup-button-confirm:hover {
  background: #1e50b5;
}

/* CANCEL BUTTON */

.popup-actions .popup-cancel-btn {
  color: #555555;
  background: #eeeeee;
  box-shadow: none;
}

.popup-actions .popup-cancel-btn:hover {
  background: #dddddd;
}

/* ANIMATION */

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

/* RESPONSIVE */

@media (max-width: 600px) {
  .popup-card {
    min-height: auto;
    padding: 30px 20px 32px;
    border-radius: 18px;
  }

  .popup-icon {
    width: 105px;
    height: 105px;
    margin-bottom: 25px;
    border-width: 7px;
    font-size: 78px;
  }

  .popup-title {
    margin-bottom: 30px;
    font-size: 19px;
  }

  .popup-message {
    margin-top: -18px;
    margin-bottom: 25px;
    font-size: 13px;
  }

  .popup-actions {
    flex-direction: column-reverse;
  }

  .popup-actions button {
    width: 100%;
    min-height: 55px;
    font-size: 19px;
  }
}
</style>
