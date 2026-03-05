<!-- src/components/AudioPlayer.vue -->
<template>
  <div>

  <div class="audio-player" v-if="!props.stop2" >
    <!-- Текущий трек -->
    <!-- <div v-if="currentTrack" class="track-info">
      {{ currentTrack.title }} — {{ currentTrack.artist }}
    </div> -->
    <!-- Двойная кнопка: Play/Pause + Next -->
    <div class="controls-player" >
     <button @click="playBack" class="btn btn-secondary-left">⏭</button>
      <button @click="togglePlayPause" class="btn btn-primary">
      <span v-if="!isPlaying">tematic ▶ longtrack</span>
        <span v-if="isPlaying">tematic <img v-if="isPlaying" src="../pause.svg" width="16" height="17"  style="top: 3px;margin: 0;position: relative; color: aliceblue;"/> longtrack</span>
      </button>
  <button @click="playNext" class="btn btn-secondary-right">⏭</button>
    </div>

    <!-- Прогресс-бар (опционально) -->
    <div v-if="currentTrack" class="progress" @click="onProgressClick">
      <div class="progress-fill" :style="{ width: progressPercent + '%' }"></div>
    </div>

    <!-- Скрытый аудио-элемент -->
    <audio ref="audioRef" preload="metadata"></audio>
  </div>
    </div>

</template>

<script setup>
import { ref, onMounted } from 'vue'

const props = defineProps({
  playlist: {name: "",
    type: [{}],
    required: true
    //default: () => []
  }, stop2: Boolean,
  stop1: Boolean,
  stop3: Boolean,
})
// 👇 ОБЯЗАТЕЛЬНО объявляем, какие события компонент может эмитить
const emit = defineEmits([
  'stop-state',
  'track-change',     // когда сменился трек
  'progress-update',  // когда обновляется прогресс
  'play-state'        // play/pause
])

// Локальное состояние
const audioRef = ref(null)
const currentTrack = ref(null)
const idplaylist = ref('222')
const isPlaying = ref(false)
const progressPercent = ref(0)
   const tracks = ref(null)
const audio = audioRef.value
// Инициализация
onMounted(async() => {
  if (props.playlist.length > 0) {
    currentTrack.value = props.playlist[0]
  } else {
    let response = await fetch('/tracks.json')
  tracks.value = await response.json()
  currentTrack.value = tracks.value[0]  
  }
  const audio = audioRef.value
  if (!audio) return

  // События аудио
  audio.addEventListener('timeupdate', () => {
    if (audio.duration) {
      progressPercent.value = (audio.currentTime / audio.duration) * 100
      emit('progress-update', {
      progressPercent,
      currentTime: audio.currentTime,
      duration: audio.duration
    })
  }  })

 audio.addEventListener('ended', () => {
  playNext()  
  emit('stop-state',
     false,
     idplaylist.value
    )
  })
  audio.addEventListener('play', () => { console.log(currentTrack.value)
    isPlaying.value = true
     emit('track-change', audioRef,  currentTrack.value, idplaylist.value)
     emit('play-state',
     true,
    
     idplaylist.value
    )
            emit('stop-state',
     true,
     idplaylist.value
    )
  })

  audio.addEventListener('pause', () => {
    isPlaying.value = false
    })
 })
// if (audio) {
//  audio.addEventListener('timeupdate', () => {
//     if (audio.duration) {
//       progressPercent.value = (audio.currentTime / audio.duration) * 100
//       emit('progress-update', {
//       progressPercent,
//       currentTime: audio.currentTime,
//       duration: audio.duration
//     })
//   }  })
//}
// Переключение Play/Pause
function togglePlayPause() {console.log(props.stop1)
  const audio = audioRef.value
  if (!audio) return

  if (isPlaying.value) {
    audio.pause()
       isPlaying.value = false
  } else {
            if (progressPercent.value && !props.stop1 && !props.stop3) {
          audio.currentTime = Math.round(currentTrack.value.duration * progressPercent.value)/100
        audio.play().catch(console.error)
        return
        } 
    // Если трек не загружен — загружаем
    if (!currentTrack.value) {        console.log(currentTrack.value)
      if (props.playlist.length > 0) {
        currentTrack.value = props.playlist
      } else return
    }
                if (!props.stop1 && !props.stop3) {  
       audio.src = currentTrack.value.url
       audio.play().catch(console.error)
       isPlaying.value = true
      emit('play-state', false, idplaylist )
       emit('track-change', audioRef, currentTrack.value, idplaylist.value)
        emit('stop-state', true, idplaylist.value)
      //playNext()
        return
            }                           
    audio.src = currentTrack.value.url
    audio.play().catch(console.error)
    }
}

// Следующий трек (случайный или по порядку)
function playNext() { 
   let nextIndex
  if (props.playlist.length <= 1) {
    if (tracks.value !== null) { 
          //const currentIndex = tracks.value.findIndex(t => t.url === currentTrack.value.url)
      nextIndex = Math.floor(Math.random() * tracks.value.length)
      currentTrack.value = tracks.value[nextIndex]
    console.log(currentTrack.value)

    const audio = audioRef.value 
  audio.src = currentTrack.value.url
  audio.play().catch(console.error)
  isPlaying.value = true
      emit('play-state', false, idplaylist )
       emit('track-change', audioRef, currentTrack.value, idplaylist.value)
              emit('progress-update', {
      progressPercent,
      currentTime: audio.currentTime,
      duration: audio.duration
    })
    return
}
    }
   if (currentTrack.value) {
    const currentIndex = props.playlist.findIndex(t => t.url === currentTrack.value.url)
    // Можно сделать случайный:
    //nextIndex = Math.floor(Math.random() * props.playlist.length)
    // Или следующий по порядку:
    nextIndex = (currentIndex + 1) % props.playlist.length
    console.log(currentTrack.value)
  } else {
    nextIndex = 0
  }

  currentTrack.value = props.playlist[nextIndex]
  const audio = audioRef.value
  
  audio.src = currentTrack.value.url
  audio.play().catch(console.error)
  isPlaying.value = true
      emit('play-state', true, idplaylist )

  emit('track-change', audioRef, currentTrack.value, idplaylist.value)
      emit('progress-update', {
      progressPercent,
      currentTime: audio.currentTime,
      duration: audio.duration
    })
  
}

// Клик по прогресс-бару
function onProgressClick(event) {
  const audio = audioRef.value
  if (!audio || !audio.duration) return

  const rect = event.currentTarget.getBoundingClientRect()
  const clickX = event.clientX - rect.left
  const percent = Math.max(0, Math.min(1, clickX / rect.width))
  audio.currentTime = percent * audio.duration
}


</script>

<style scoped>
.audio-player {
  position: relative;
  display: flex;
    flex-wrap: wrap;
      padding: 30px;
  border-radius: 8px;
  margin-bottom: 16px;
  max-width: 300px;
  
}
.track-info {
  font-weight: bold;
  margin-bottom: 12px 12px 12px 12px;
  min-height: 24px;
}
.controls {
display: flex;
  justify-content: center;
  gap: 1px;
}
.btn {
  padding: 7px 8px 9px 10px;
  border: none;
  cursor: pointer;
  font-size: 16px;
}
.btn-secondary-left { background: #278268; color: white;
  border-radius:  16px 2px 2px 16px;
}
.btn-primary { background: #278268; color: white;
  border-radius:  2px 2px 2px 2px;
  width: 200px;
}
 .btn-secondary-right { background: #278268; color: white;
  border-radius:  2px 16px 16px 2px;
}
 .progress {
  width: 100%;
  height: 6px;
  background: #eee;
  margin-top: 32px;
  cursor: pointer;
  position: relative;
} 
.progress-fill {
  height: 100%;
  background: #007bff;
  transition: width 0.1s ease;

}

</style>