<!-- src/components/FloatingModal.vue -->
<template>
  <Teleport to="body">
    <div v-if="modelValue" class="modal-overlay" @click="close">
      <div class="modal-content" @click.stop>
        <button class="modal-close" @click="close">залетаем  ▶ слушаем</button>
        <div
          ref="textContainer"
          class="modal-text"
          :class="{ 'scrollable': isScrollable }"
        >
          <div
            class="floating-text"
            :style="{
              transform: `translateY(${offsetY}px) translateZ(${offsetZ}px)`,
              textShadow: '0 0 10px rgba(255,255,255,0.7)'
            }"
          >Добро пожаловать в музыкальное приложение!
   Здесь вы найдёте лучшие треки из разных вселенных.
    Наслаждайтесь звуком, визуальными эффектами и плавным интерфейсом
    Совет от DJ MM- для получения максимального удовольствия
    прослушивайте подобную музыку качественных наушниках или через
    домашний музыкальный центр.
    **Первое — главное: я скажу всё, что внутри моей головы.
Я устал измучен тем, как всё было.
Тем Видя красоту сквозь...****Ты разбиваешь меня
Ты разбиваешь меня — ты делаешь меня верующим.*
"Believer" от группы Imagine Dragons

            <slot/>
          </div>
        </div>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
import { ref, onMounted, onUnmounted, nextTick, watch } from 'vue'

const props = defineProps({
  modelValue: Boolean // v-model
})

const emit = defineEmits(['update:modelValue'])

const textContainer = ref(null)
const isScrollable = ref(false)
const offsetY = ref(0)
const offsetZ = ref(0)

// Закрытие
const close = () => {
  emit('update:modelValue', false)
}

// Проверка, нужна ли прокрутка
const checkScroll = () => {
  if (!textContainer.value) return
  const el = textContainer.value
  isScrollable.value = el.scrollHeight > el.clientHeight
}

// Плавающая анимация (только если НЕ прокручиваемый)
let animationFrameId = null
const animate = () => {
  if (!isScrollable.value) {
    const time = Date.now() * 0.001
    offsetY.value = Math.sin(time * 0.8) * 8
    offsetZ.value = Math.cos(time * 0.5) * 10
  }
  animationFrameId = requestAnimationFrame(animate)
}

// Отслеживаем изменения содержимого
watch(() => props.modelValue, async (isOpen) => {
  if (isOpen) {
    await nextTick()
    checkScroll()
    if (!isScrollable.value) {
      animate()
    }
  } else {
    if (animationFrameId) {
      cancelAnimationFrame(animationFrameId)
      animationFrameId = null
    }
  }
})

onMounted(() => {
  window.addEventListener('resize', checkScroll)
})

onUnmounted(() => {
  window.removeEventListener('resize', checkScroll)
  if (animationFrame) {
    cancelAnimationFrame(animationFrameId)
  }
})
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.85);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  backdrop-filter: blur(4px);
}

.modal-content {
  position: relative;
  background: linear-gradient(135deg, #1a1a2e, #16213e);
  color: white;
  border-radius: 16px;
  max-width: 90vw;
  max-height: 80vh;
  width: 500px;
  overflow: hidden;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5),
              0 0 40px rgba(100, 149, 237, 0.4);
  perspective: 1000px;
}

.modal-close {
  position: absolute;
  top: 10px;
  right: 26%;
  margin-bottom: 10px;
  background: rgba(255, 255, 255, 0.15);
  border: none;
  color: rgb(141, 136, 136);
  font-size: 18px;
  /* width: 36px; */
  height: 36px;
  padding-top: 7px;
  border-radius: 14px;
  cursor: pointer;
  backdrop-filter: blur(2px);
  transition: background 0.3s;
}

.modal-close:hover {
  background: rgba(255, 255, 255, 0.3);
}

.modal-text {
  padding: 60px 24px 24px;
  height: calc(80vh - 60px);
  overflow: hidden;
}

.modal-text.scrollable {
  overflow-y: auto;
  padding-right: 12px; /* компенсация скролла */
}

.modal-text.scrollable .floating-text {
  transform: none !important;
  text-shadow: none;
  animation: none;
}

.floating-text {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  font-size: 18px;
  line-height: 1.6;
  text-align: center;
  color: #e0e0ff;
  transform-style: preserve-3d;
  transition: transform 0.2s ease-out;
  white-space: pre-wrap;
}

/* Скроллбар (для WebKit) */
.modal-text::-webkit-scrollbar {
  width: 8px;
}
.modal-text::-webkit-scrollbar-track {
  background: rgba(0,0,0,0.2);
  border-radius: 4px;
}
.modal-text::-webkit-scrollbar-thumb {
  background: rgba(100, 149, 237, 0.6);
  border-radius: 4px;
}
</style>