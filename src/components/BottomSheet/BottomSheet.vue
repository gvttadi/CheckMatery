<script setup lang="ts">
import { computed, onBeforeUnmount, watch, ref } from 'vue'

defineOptions({
  name: 'AppBottomSheet',
})

const props = withDefaults(defineProps<{
  modelValue: boolean
  closeOnOverlayClick?: boolean
  closeOnEsc?: boolean
  maxWidth?: string
}>(), {
  closeOnOverlayClick: true,
  closeOnEsc: true,
  maxWidth: '640px',
})

const emit = defineEmits<{
  (e: 'update:modelValue', value: boolean): void
  (e: 'close'): void
}>()

const isOpen = computed(() => props.modelValue)
const overlayRef = ref<HTMLElement | null>(null)

function close() {
  emit('update:modelValue', false)
  emit('close')
}

function onOverlayClick(e: MouseEvent) {
  if (!props.closeOnOverlayClick) return
  // ensure click on overlay (not the sheet)
  if (e.target === overlayRef.value) close()
}

function onKeydown(e: KeyboardEvent) {
  if (e.key === 'Escape' && props.closeOnEsc) close()
}

watch(isOpen, (open) => {
  if (open) {
    document.body.style.overflow = 'hidden'
    document.addEventListener('keydown', onKeydown)
  } else {
    document.body.style.overflow = ''
    document.removeEventListener('keydown', onKeydown)
  }
})

onBeforeUnmount(() => {
  document.body.style.overflow = ''
  document.removeEventListener('keydown', onKeydown)
})
</script>

<template>
  <Teleport to="body">
    <div v-if="isOpen" ref="overlayRef" class="bs-overlay" @click="onOverlayClick">
      <div v-if="isOpen" class="bs-sheet" :style="{ maxWidth: props.maxWidth }">
        <slot />
      </div>
    </div>
  </Teleport>
</template>

<style scoped>
.bs-overlay {
  position: fixed;
  inset: 0;
  display: flex;
  align-items: flex-end;
  justify-content: center;
  background: rgba(0, 0, 0, 0.45);
  z-index: 1000;
  -webkit-tap-highlight-color: transparent;
  animation: overlayFadeIn 220ms cubic-bezier(.2,.8,.2,1);
}

.bs-sheet {
  width: 100%;
  max-width: 640px;
  background: var(--body-bg, #fff);
  border-radius: 12px 12px 0 0;
  box-shadow: 0 -8px 30px rgba(0,0,0,0.2);
  padding: 16px;
  animation: sheetSlideUp 220ms cubic-bezier(.2,.8,.2,1);
}

@keyframes overlayFadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes sheetSlideUp {
  from {
    transform: translateY(100%);
  }
  to {
    transform: translateY(0);
  }
}
</style>
