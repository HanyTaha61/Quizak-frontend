<template>
  <transition name="slide-down">
    <div v-if="isVisible" class="sticky-ad">
      <div class="ad-content">
        <span>🔥 إعلان مميز هنا - اكتشف الآن!</span>
        <button class="close-btn" @click="closeAd">✕</button>
      </div>
    </div>
  </transition>
</template>

<script setup>
import { ref } from "vue";

const isVisible = ref(true);

const closeAd = () => {
  isVisible.value = false;
};
</script>

<style scoped>
.sticky-ad {
  position: sticky;
  top: 0;
  z-index: 9999;
  background: linear-gradient(90deg, #ff7a18, #ffb347);
  color: white;
  padding: 20px 16px;
}

.ad-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.close-btn {
  background: transparent;
  border: none;
  color: white;
  font-size: 16px;
  cursor: pointer;
}

/* animation */
.slide-down-enter-active,
.slide-down-leave-active {
  transition: all 0.3s ease;
}
.slide-down-enter-from {
  transform: translateY(-100%);
  opacity: 0;
}
.slide-down-leave-to {
  transform: translateY(-100%);
  opacity: 0;
}
</style>