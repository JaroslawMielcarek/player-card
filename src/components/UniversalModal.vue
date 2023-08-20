<template>
  <transition name="modal-animation">
    <div v-show="props.modalActive" :class="['modal', 'theme-' + theme]">
      <transition name="modal-animation-inner">
        <div v-show="props.modalActive" :class="['modal-inner', 'theme-' + theme]">
          <span class="close" @click="close"></span>
          <slot></slot>
        </div>
      </transition>
    </div>
  </transition>
</template>
<script setup lang="ts">
import { inject } from 'vue';

const props = defineProps<{modalActive: boolean}>()
const emit = defineEmits(['close'])

const theme = inject('theme')
const close = () => emit('close')

</script>

<style>
.modal-animation-enter-active,
.modal-animation-leave-active {
  transition: opacity 300ms ease-in-out;
}
.modal-animation-enter-from,
.modal-animation-leave-to {
  opacity: 0;
}
.modal-animation-inner-enter-active {
  transition: all 300ms ease-in-out 500ms;
}
.modal-animation-inner-leave-active {
  transition: all 300ms ease-in-out;
}
.modal-animation-inner-enter-from {
  opacity: 0;
  transform: scale(0.8);
}
.modal-animation-inner-leave-to {
  transform: scale(0.8);
}

.modal {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  width: 100vw;
  position: fixed;
  top: 0;
  left: 0;
  background-color: rgba(var(--position-bg-color-rgb), 0.7);
  z-index: 100;
}
.modal-inner {
  position: relative;
  max-width: 400px;
  width: 80%;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  background-color: var(--position-bg-color);
  color: var(--text-color);
  padding: 32px 16px;
}

.close {
  position: absolute;
  right: 12px;
  top: 12px;
  width: 16px;
  height: 16px;
  opacity: 0.3;
}
.close:hover {
  opacity: 1;
}
.close:before, .close:after {
  position: absolute;
  left: 6px;
  content: ' ';
  height: 16px;
  width: 2px;
  background-color: var(--text-color);
}
.close:before {
  transform: rotate(45deg);
}
.close:after {
  transform: rotate(-45deg);
}
</style>