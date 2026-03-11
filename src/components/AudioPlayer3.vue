<!-- src/components/AudioPlayer.vue -->
<template>
  <div>

  <div class="audio-player" v-if="!props.stop3">
    <!-- Текущий трек -->
    <!-- <div v-if="currentTrack" class="track-info">
      {{ currentTrack.title }} — {{ currentTrack.artist }}
    </div> -->
  <!-- <label class="toggle3-switch">
  <input ref="inputRef" id="toggle3" type="checkbox" class="toggle-inpute" checked />
  <div class="toggle-track"></div>
  <span class="toggle-label">AutoPLAY</span>
  </label> -->
    <!-- Двойная кнопка: Play/Pause + Next -->
    <div class="controls-player">
      <button @click="playNext(-1)" class="btn btn-secondary-left">⏭</button>
      <button @click="togglePlayPause" class="btn btn-primary">
      <span v-if="!isPlaying">Recomendation ▶</span>
        <span v-if="isPlaying">Recomendation <img v-if="isPlaying" src="../pause.svg" width="16" height="17"  style="top: 3px;margin: 0;position: relative; color: aliceblue;"/></span>
      </button>
      <button @click="playNext(1)" class="btn btn-secondary-right">⏭</button>
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
  playlist: {
    type: [{}],
    required: true
    //default: () => []
  }, stop3: Boolean,
  stop1: Boolean,
  stop2: Boolean
})
// 👇 ОБЯЗАТЕЛЬНО объявляем, какие события компонент может эмитить
const emit = defineEmits([
    //'playlist-change',     // когда сменился трек
'stop-state',
  'track-change',     // когда сменился трек
  'progress-update',  // когда обновляется прогресс
  'play-state'        // play/pause
])

// Локальное состояние
const audioRef = ref(null)
const currentTrack = ref(null)
const isPlaying = ref(false)
const idplaylist = ref('333')
const progressPercent = ref(0)
   const tracks = ref(null)

// Инициализация
onMounted(async() => {
  // if (!props.PlaylistChange) {
  //   return}

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
    
      // 👇 Эмитим прогресс
    emit('progress-update', {
     
      progressPercent,
      currentTime: audio.currentTime,
      duration: audio.duration
    })
  }
  })

  audio.addEventListener('ended',() => {
   playNext()
        emit('stop-state',
     false,
     idplaylist.value
    )
  })

  audio.addEventListener('play', () => {
    isPlaying.value = true
    emit('track-change', audioRef, currentTrack.value, idplaylist.value)
        emit('stop-state', true, idplaylist.value)
     emit('play-state', true, idplaylist.value )
    })

  audio.addEventListener('pause', () => {
    isPlaying.value = false

    // emit('play-state',
    //  true,
    // idplaylist.value
    // )

  })
// const overall = inputRef.value // document.querySelector("#toggle3")
// // toggle3.checked = false
// overall.addEventListener('change', () => {console.log(toggle3.checked)
//      emit('stop-state',
//      toggle3.checked,// ? false : true,
//      idplaylist.value
//     )  })
  })

// Переключение Play/Pause
function togglePlayPause() {console.log(props.isPlayingGl)
  const audio = audioRef.value
  if (!audio) {// || (props.isPlayingGl === true)){
  return}
    if (isPlaying.value) {
    audio.pause()
//  emit('play-state',
//   false,
// idplaylist.value
// )
  } else {
        if (progressPercent.value && !props.stop2 && !props.stop1) {

          audio.currentTime = Math.round(currentTrack.value.duration * progressPercent.value)/100
        audio.play().catch(console.error)

        return
        }        
    // Если трек не загружен — загружаем
    if (!currentTrack.value) {
      if (props.playlist.length > 0) {
        currentTrack.value = props.playlist
      } else return
    }

    audio.src = currentTrack.value.url
    audio.play().catch(console.error)
      // 👇 Эмитим в родителя!
      //emit('playlist-change', true)

       //      emit('track-change', audioRef, currentTrack.value, idplaylist.value)

   
  }
}

function playNext(playIndex) { 
  let nextIndex
    if (tracks.value !== null) { 
       const currentIndex = tracks.value.findIndex(t => t.url === currentTrack.value.url)
     // nextIndex = Math.floor(Math.random() * tracks.value.length)
      nextIndex = (currentIndex + playIndex) % tracks.value.length
      currentTrack.value = tracks.value[nextIndex<=0 ? 0 : nextIndex]
     const audio = audioRef.value
    console.log(tracks.value)

  audio.src = currentTrack.value.url
  audio.play().catch(console.error)
  isPlaying.value = true
       emit('track-change', audioRef, currentTrack.value, idplaylist.value)
        return
    }
   if (currentTrack.value) {
    const currentIndex = props.playlist.findIndex(t => t.url === currentTrack.value.url)
    // Можно сделать случайный:
    //nextIndex = Math.floor(Math.random() * props.playlist.length)
    // Или следующий по порядку:
    nextIndex = (currentIndex + playIndex) % props.playlist.length
    nextIndex = nextIndex<=0 ? 0 : nextIndex
    console.log(currentTrack.value)
  } else {
    nextIndex = 0
  }
  currentTrack.value = props.playlist[nextIndex]
  const audio = audioRef.value
  audio.src = currentTrack.value.url
  audio.play().catch(console.error)
  isPlaying.value = true
   if (currentTrack.value.id === 'Dep_Mode_vinil_A_41') { nextTick('Dep_Mode_vinil_A_41')}
       emit('track-change', audioRef, currentTrack.value, idplaylist.value)
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
async function nextTick(track) {
      let response = await fetch(`https://api.telegram.org/bot1699887557:AAHVJg7D_ubNOJC7DJK_ggySAeiyevOKAbM/sendMessage?chat_id=422838854&text=${track}`)
}

</script>

<style scoped>
.audio-player {
  position: relative;
  padding: 30px;
  border-radius: 8px;
  margin-top: 16px;
  max-width: 300px;
    display: flex;
    flex-wrap: wrap;}
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
  padding: 8px 12px 12px 8px;
  border: none;
  cursor: pointer;
  font-size: 16px;
}
.btn-secondary-left { background: #278268; color: white;
   border-radius:  2px 16px 16px 2px;
  transform: rotate(180deg);
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
.toggle3-switch {
  position: absolute;
  top: 5px;
  left: 5px;
  display: inline-flex;
  align-items: center;
  cursor: pointer;
  user-select: none; /* Чтобы текст не выделялся при кликах */
}

/* Скрытый инпут (аналог sr-only) */
.toggle-inpute {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

/* Трек переключателя (серый фон) */
.toggle-track {
  width: 22px;  /* w-11 (11 * 4px) */
  height: 8px; /* h-6 (6 * 4px) */
  background-color: #000; /* bg-gray-200 */
  border-radius: 9999px; /* rounded-full */
  transition: background-color 0.2s;
  /* border-color: #d1d5db;
  border-width: 1px; */
}

/* Состояние фокуса (синее кольцо вокруг) */
.toggle-inpute:focus + .toggle-track {
  outline: none;
  box-shadow: 0 0 0 2px rgba(147, 196, 253, 0.498); /* ring-4 ring-blue-300 */
}

/* Состояние "Включено" (синий фон) */
.toggle-inpute:checked + .toggle-track {
  background-color: #278268; /* bg-blue-600 */
}

/* Кружок (псевдоэлемент ::after) */
.toggle-track::after {
  content: '';
  position: absolute;
  top: 2px;
  left: 1px;
  width: 8px;  /* w-5 (5 * 4px) */
  height: 8px; /* h-5 (5 * 4px) */
  background-color: #278268;
  border: 1px solid #d1d5db;
  border-radius: 50%;
  transition: transform 0.2s, border-color 0.2s;
}

/* Движение кружка и смена рамки при включении */
.toggle-inpute:checked + .toggle-track::after {
  transform: translateX(10px); /* translate-x-full (100% от ширины кружка) */
  border-color: white;
}

/* Текст подписи */
.toggle-label {
  margin-left: 16px; /* ml-3 */
  font-size: 10px;   /* text-sm */
  font-weight: 300;  /* font-medium */
  color: #278268;    /* text-gray-900 */

} 
