<script setup>
import { ref, onMounted } from 'vue';

// Реактивные данные
const loading = ref(false);
const videoUrl = ref('');
const isPlaying = ref(false);

// Доступ к DOM элементам через ref
const videoElement = ref(null);

// Обработчик загрузки файла
const handleVideoChange = (e) => {
  const file = e.target.files[0];
  if (!file) return;

  loading.value = true;
  const url = URL.createObjectURL(file);
  videoUrl.value = url;

  // Устанавливаем src в video
  videoElement.value.src = url;

  // Добавляем обработчик loadeddata
  videoElement.value.addEventListener('loadeddata', () => {
    loading.value = false;
    if (isPlaying.value) {
      // Тестовый вызов (ваша логика здесь)
      console.log('Screen switch started');
    }
  });
};

// Обработчик ошибок
const handleVideoError = (e) => {
  alert('Ошибка загрузки видео.');
  loading.value = false;
};

// Инициализация
onMounted(() => {
  // Автоматически инициализируем videoElement
  videoElement.value = document.querySelector('video');
});
</script>

<template>
  <div class="container">
    <input id="videoFile"
      type="file" 
      ref="videoFileInput" 
      @change="handleVideoChange"
      accept="video/*"
      class="hidden"
    />
    <div v-if="loading" class="loading">Загрузка видео...</div>
    <video 
      ref="video" 
      :src="videoUrl"
      @error="handleVideoError"
      class="video"
      controls
    />
    <button @click="ref('videoFileInput')" >🎥 Загрузить видео</button>

  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.loading {
  display: block;
  color: #fff;
  text-align: center;
  padding: 10px;
  background-color: #000;
  border-radius: 4px;
  margin: 10px 0;
}

.video {
  width: 100%;
  max-width: 800px;
  aspect-ratio: 16 / 9;
  border-radius: 4px;
  margin-top: 10px;
}
button {
    background: linear-gradient(45deg, #000, #0ff);
    border: none;
    color: white;
    padding: 10px 20px;
    margin: 0 8px;
    border-radius: 30px;
    cursor: pointer;
    font-weight: bold;
    transition: all 0.2s ease;
    box-shadow: 0 0 10px rgba(0,255,255,0.5);
}

button:hover {
    transform: scale(1.05);
    box-shadow: 0 0 20px rgba(0,255,255,0.8);
}
</style>
