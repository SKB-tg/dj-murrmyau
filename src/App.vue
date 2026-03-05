
<template>
  <div >
                        <div class="logo-wrapper">
                <div ref="limg" style="left:0" class="limg"></div>
                <img  ref="logoImg" src="/DJ_MurrMyau_Album.jpg" class="logo-img" alt="DJ_MurrMyau.jpg">
                <div ref="limg" style="right:0" class="limg"></div>
              <div class="cat-eyes">
                    <div class="eye left" :style="{ opacity: eyey }"></div>
                    <div class="eye right" :style="{ opacity: eyey }"></div>
                </div>
            </div>
        <img @click="onlogoBoost" src="/favicon-16x15.png" style="position:absolute;right:-20px;top:20px" class="btn boost">
                
    <!-- <h2>Плейлист 1: Джаз</h2> -->
  
  <!-- Общий прогресс-бар -->
  <div class="progress-container" v-if="activeTrack1" @click="onProgressClickG">
      <div class="progress-glob" v-if="activeTrack1" :style="{ width: globalProgress + '%' }"></div>
    </div>
 <div class="marquee" v-if="activeTrack1"><span>{{ marqueText }}</span></div>        
<div class="track-info" v-if="activeTrack1">{{ activeTrack1.title }}</div>
<div class="diz"> 
      <label class="toggle-switch" >
  <input id="toggle" type="checkbox" class="toggle-input" switch />
  <div class="toggle-track"></div>
  <span class="toggle-label">AutoPLAY</span>
  </label>
       <AudioPlayer1 v-if="!stop1" :playlist="Spaceelectro" :stop1="stop1" :isPlayingGl="isPlayingGl"
        @track-change="onTrackChange"
        @progress-update="onProgressUpdate"
        @stop-state="offStopState"
        @play-state="onPlayState"/>
        <AudioPlayer2 v-if="!stop2" :playlist="Electron" :stop2="stop2" :isPlayingGl="isPlayingGl"
        @track-change="onTrackChange"
        @progress-update="onProgressUpdate"
        @stop-state="offStopState"
        @play-state="onPlayState"/>
        <AudioPlayer3 v-if="!stop3" :playlist="rw" :stop3="stop3" :isPlayingGl="isPlayingGl"
        @track-change="onTrackChange"
        @progress-update="onProgressUpdate"
        @stop-state="offStopState"
        @play-state="onPlayState"/></div>
    
        <!-- <div class="controls"></div> -->
        <div ref="visualizer" class="visualizer">
            <!-- Canvas, video, particles -->
        <canvas ref="canvas" id="visualizerCanvas">

        </canvas>
            <div ref="particles" class="particles" id="particles"></div>

        </div>
        <div class="info">DJ MurrMyau — визуализатор музыки • Новый Год 2025</div>
    </div>
</template>
<script setup>
import { ref, onMounted } from 'vue'
import AudioPlayer1 from './components/AudioPlayer1.vue'
import AudioPlayer2 from './components/AudioPlayer2.vue'
import AudioPlayer3 from './components/AudioPlayer3.vue'

// Эти данные могут приходить из tracks.json или API
const Spaceelectro = ref([])
const Electron = ref([{}])
const rw = ref([])

//**************************************************************** */

let audioContext = ref(null)
let analyser = ref(null)
let source = ref(null)
let dataArray = ref(null)
let bufferLength = ref(0)
const audioGlobRef = ref (null)
const audioElement = ref(null)
const isPlayingGl = ref(false)
const  mediaSource = ref(HTMLMediaElement)
const ctx = ref(null)
const tracks = ref(null)
const particles = ref(null)
const canvas = ref(null)
const currentTrack = ref({ title: 'Выберите', url: '', artist: '', duration: 0 });
const prbare = ref(null)
         const butImg = ref(0)
         const eyey = ref(0)
        const limg = ref(null)
        const logoImg = ref(null)
let marqueText = ref('Инфо...');
const defaultTracks = ref([{},{}])//['https://raw.githubusercontent.com/kirov-sk/scool/blob/main/Limbo.mp3', 'https://raw.githubusercontent.com/kirov-sk/scool/blob/main/The%20Legend.mp3', 'https://raw.githubusercontent.com/SKB-tg/50projects50days/blob/master/Kolybel.mp3']; // Твоя база, рандом
const defaultTrack = { title: 'Тематический', url: ''};//defaultTracks[Math.floor(Math.random() * defaultTracks.length)] };
const albumTrack = { title: 'Мой трек', url: 'album.mp3' };
const thematicTrack = { title: 'Известный', url: 'thematic.mp3' };
    if (canvas.value != null) {ctx.value = canvas.value.getContext('2d')}
//****************************************** */
Electron.value = [
   { 'title': 'Track 2003', 'artist': 'DJ Bushwacka', 'url': '/audio/Bushwackas.mp3', 'duration': 200}, {"id": "107FM_18_BACK_TO_THE_UNIVERSE_2000", 'title': '107FM_18_BACK_TO_THE_UNIVERSE_2000', 'artist': 'yy kkl', 'url': '/audio/107FM_18_BACK_TO_THE_UNIVERSE_2000.mp3',  'duration': 200},
{"id": "Dep_Mode_vinil_A_41", 'title': '4 Tracks: Depecne Mode', 'artist': 'Depecne Mode', 'url': '/audio/Dep_Mode_vinil_A_41.mp3', 'duration': 1200},
]
 rw.value = [{"id": "Dep_Mode_vinil_A_41", 'title': '4 Tracks: Depecne Mode', 'artist': 'Depecne Mode', 'url': '/audio/Dep_Mode_vinil_A_41.mp3', 'duration': 1200},
   {"id": "Remix_the_hall_of_the_mountain_king_E_G", "url": "/audio/Remix_the_hall_of_the_mountain_king_E_G.mp3", "title": "Remix 'the Hall of the Mountain King' E. G.", "artist": "DJ MurrMyau", "album": "Black & W   creator DJ MurrMyau", "year": "2025", "duration": 259},
    {"id": "Adrenalin", "url": "/audio/Adrenalin.mp3", "title": "Adrenalin", "artist": "DJ MurrMyau", "album": "Black & W   creator DJ MurrMyau",  "year": "2025", "duration": 175},
{"id": "L_D_mix", 'title': 'Remix "Beliver"', 'artist': 'DJ MurrMyau', "album": "Black & W   creator DJ MurrMyau", 'url': '/audio/I_D_mix.mp3', 'duration': 300 }]
// Общее состояние для всех плееров
const activeTrack1 = ref(null)
const globalProgress = ref(0)
const stop1 = ref(false)
const stop2 = ref(false)
const stop3 = ref(false)

// Обработчики событий от дочерних компонентов
function onTrackChange(audioRef, track, idplaylist) {
     //if (activeTrack1.value) {
        activeTrack1.value = track
    // }
    if (audioRef) {
    console.log(audioRef.value)
audioGlobRef.value = audioRef.value}
    if (track != null) { 
  marqueText = `Трек: ${track.title || ' --- '} | Артист: ${track.artist || ' --- '} | 'Альбом: ', ${activeTrack1.value.album || ' --- ' || 'Дефолт'}| Файл: ${ activeTrack1.value.id}.mp3  | Длительность: ${Math.round(activeTrack1.value.duration/60)+' min '+Math.round(activeTrack1.value.duration%60) || track.duration} сек`;
console.log(audioGlobRef.value)
}
if (!audioContext.value) {
//console.log(audioGlobRef.value)
initAudio()
initAudioContext()
 connectAudioToVisualizer(audioGlobRef.value)
} else {
 //connectAudioToVisualizer(audioGlobRef.value)    
}
}

function onProgressUpdate(data) {//if (data) {
  globalProgress.value = data.progressPercent.value
   if (activeTrack1.value) {//console.log(data.progressPercent.value)
    activeTrack1.value.duration = Math.round(data.duration)}
}//}
function offStopState(state, idplaylist) {
//stop1.value = (idplaylist === '111') ? !state : state
    console.log(state)
    toggle.checked = true
//   if (state === false) {
//         stop2.value === false ? stop2.value=true : stop2.value=false
//         stop3.value === false ? stop3.value=true : stop3.value=false
//         stop1.value === false ? stop1.value=true : stop1.value=false}
//         else {
    if (idplaylist === '111') {
        state ? stop2.value=true : stop2.value=false
        state ? stop3.value=true : stop3.value=false
    }
   if (idplaylist === '222') {
        state ? stop1.value=true : stop1.value=false
        state ? stop3.value=true : stop3.value=false
    }   if (idplaylist === '333') {
        state ? stop2.value=true : stop2.value=false
        state ? stop1.value=true : stop1.value=false
     
     console.log(stop2.value)
}}
function onPlayState(isPlaying, idplaylist) {
    isPlayingGl.value = isPlaying
    console.log(isPlayingGl.value)
  // Можно, например,
  //  менять иконку глобальной кнопки
}

    // === ИНИЦИАЛИЗАЦИЯ ===
function initAudio() {
  audioElement.value = new Audio({autoplay: false, preload: "metadata"})
      //audioRef.value = audioElement.value
}
function initAudioContext() {
    if (audioContext.value === null) {
                    
        audioContext.value = new window.AudioContext({
            latencyHint: "interactive",
        })
        ctx.value = audioContext.value.createAnalyser();

        analyser.value = audioContext.value.createAnalyser();
        analyser.value.fftSize = 2048;
        bufferLength = analyser.value.frequencyBinCount;
        dataArray.value = new Uint8Array(bufferLength);
    } //else {audioContext.value = audioElement.value;
    //     ctx.value = audioContext.value.createAnalyser();
    // console.log(audioContext)
    //     analyser.value = audioContext.value.createAnalyser();
    //     analyser.value.fftSize = 2048;
    //     bufferLength = analyser.value.frequencyBinCount;
    //     dataArray.value = new Uint8Array(bufferLength);}
        console.log(analyser.value)

}

function resizeCanvas() {
    canvas.value.width = canvas.value.offsetWidth;
    canvas.value.height = canvas.value.offsetHeight;
}

// Клик по прогресс-бару
function onProgressClickG(event) {
  const audio = audioGlobRef.value
  if (!audio || !audio.duration) return

  const rect = event.currentTarget.getBoundingClientRect()
  const clickX = event.clientX - rect.left
  const percent = Math.max(0, Math.min(1, clickX / rect.width))
  audio.currentTime = percent * audio.duration
}

onMounted(() => {
const overall = document.querySelector("#toggle")
// toggle3.checked = false
overall.addEventListener('change', () => {

    if (!toggle.checked) {
        stop2.value = false 
        stop1.value = false
        stop3.value = false
      //  xxx.value = -11
     } 
})  
//    var track = ref({})
//   let response = await fetch('/tracks.json')
//   tracks.value = await response.json()
//   Spaceelectro.value = tracks.value
//           if (Spaceelectro.value.length > 0) {
//            // for (track in Space_electro){
//               console.log( Spaceelectro.value)}
resizeCanvas()
createParticles()
logo_dens()
//play_Button();
window.addEventListener('resize', () => {
   logo_dens()
  resizeCanvas()
    createParticles()
})
 //draw()

})
function connectAudioToVisualizer(track) {
   //initAudioContext();
    if (audioContext.value.state === 'suspended') {
        audioContext.value.resume();
    }
         track
    console.log(mediaSource)
    if (track != null) {
    if (source.value != null) {source.value.disconnect();}
    source.value = audioContext.value.createMediaElementSource(track, mediaSource);
    source.value.connect(analyser.value);
    analyser.value.connect(audioContext.value.destination);
}}
        function onlogoBoost() {
            butImg.value++ 
           if ((Math.floor(butImg.value/2) - butImg.value/2) === 0) {
            logoImg.value.style = "max-width: 630px;"
            limg.value.style = "height: 630px;width: 630px;"
           } else {
            logoImg.value.style = "max-width: 450px;"
            limg.value.style = "height: 450px;width: 450px;" 
           }           
        }
        // === ЧЕРЕДОВАНИЕ ЭКРАНОВ ===
        function startScreenSwitch() {
            clearTimeout(screenTimer);
            firstScreenTime = 30 || parseInt(screenTimeInput.value);

            showFirstScreen();
            screenTimer = setTimeout(() => {
                if (!videoElement.src || !isPlaying) return;
                showVideoScreen();
                videoElement.addEventListener('ended', () => {
                    if (isPlaying) startScreenSwitch();
                }, { once: true });
            }, firstScreenTime * 180);
        }

        function showFirstScreen() {
            canvas.style.display = 'block';
            videoElement.style.display = 'none';
            videoElement.pause();
        }

        function showVideoScreen() {
            canvas.style.display = 'none';
            videoElement.style.display = 'block';
            videoElement.play();
        }
  // === ЧАСТИЦЫ ===
    function createParticles() {
        particles.innerHTML = '';
        const count = Math.min(window.innerWidth / 20, 50);
        for (let i = 0; i < count; i++) {
            const particle = document.createElement('div');
            particle.classList.add('particle');
            particle.style.left = `${Math.random() * 100}%`;
            particle.style.top = `${Math.random() * 100}%`;
            particle.style.opacity = Math.random() * 0.7 + 0.3;
            particles.value.appendChild(particle);
            animateParticle(particle);
        }
    }

    function animateParticle(particle) {
        const duration = 3000 + Math.random() * 5000;
        const startX = parseFloat(particle.style.left);
        const startY = parseFloat(particle.style.top);
        const endX = Math.random() * 100;
        const endY = Math.random() * 100;

        let start = null;
        function step(timestamp) {
            if (!start) start = timestamp;
            const progress = timestamp - start;
            const percent = Math.min(progress / duration, 1);
            const eased = 1 - Math.pow(1 - percent, 3);

            particle.style.left = `${startX + (endX - startX) * eased}%`;
            particle.style.top = `${startY + (endY - startY) * eased}%`;

            if (percent < 1) {
                requestAnimationFrame(step);
            } else {
                animateParticle(particle);
            }
        }
        requestAnimationFrame(step);
    }

 // === ВИЗУАЛИЗАЦИЯ ===//
    // Динамика логотипа: изменение цвета
    function logo_dens(){
    window.requestAnimationFrame(logo_dens);

    if (analyser.value === null) return;

    analyser.value.getByteFrequencyData(dataArray.value)
    //********************************* */
           const bass = dataArray.value[0] + dataArray.value[1] + dataArray.value[2] / 3;
            const blink = Math.min(bass / 396, 1);
 
        if (limg.value != null) limg.value.style.opacity = 1 - blink * 0.85

         if (eyey.value != null) eyey.value = 1 - blink * 0.95

            // Динамика логотипа: изменение цвета
            const logoHue = (bass / 255) * 360;
           if (logoImg.value != null) {logoImg.value.style.filter = `hue-rotate(${logoHue}deg) drop-shadow(0 0 20px rgba(0, 255, 255, ${blink}))`}
    }
//*************************** */
</script>

<style scoped>
         .track-info {display: block;
            margin: 15px 0;
            font-size: 18px;
            font-weight: bold;
            color: #0ff;
            text-shadow: 0 0 10px rgba(0, 255, 255, 0.7);
            min-height: 28px;
        }

    
        .progress-container {
            width: 90%;
            max-width: 600px;
            height: 8px;
            background: rgba(255, 255, 255, 0.11);
            border-radius: 4px;
            margin: 10px auto;
            overflow: hidden;
            cursor: pointer;
        }

        .progress-glob {
            height: 6px;
            background: linear-gradient(90deg, #0ff, #0f0);
            border-radius: 4px;
            transition: width 0.1s linear;
        } 
                  .logo-wrapper {
            position: relative;
            display: inline-block;
            height: inherit;
            margin-bottom: 20px;
        }
            .logo-img {
            margin-top: 25px;
            height: inherit;
            max-width: 450px;
            transform: filter 0.5s ease;
        }

        .limg {
            position: absolute;
            top: 25px;
            width: 450px;
            height: 450px;
            background: #00ff03;
            z-index: -1;
         filter: drop-shadow(0 0 70px rgba(0, 255, 255, 0.9)); 
        }

        .cat-eyes {
            position: absolute;
            top: 0; left: 0;
            width: 100%; height: 100%;
            pointer-events: none;
        }

        .eye {
            position: absolute;
            width: 15px;
            height: 12px;
            background: #e20831;
            border-radius: 50%;
            box-shadow: 0 0 10px rgba(0, 255, 255, 0.8);
            transform: opacity 0.5s ease;
        }
 .marquee {
			height: 77%;
			    max-height: 24px;
            width: 100%;
            max-width: 600px;
            margin: 15px auto;
            overflow: hidden;
            white-space: nowrap;
            box-sizing: border-box;
            color: #0f0;
            font-size: 20px;
        }

        .marquee span {
            display: inline-block;
            padding-left: 100%;
            animation: marquee 15s linear infinite;
        }
        @keyframes marquee {
            0% { transform: translate(0, 0); }
            100% { transform: translate(-100%, 0); }
        }   
 .boost {
                cursor: pointer;
            font-weight: bold;
            transition: all 0.2s ease;
            box-shadow: 0 5px 10px rgba(0,255,255,0.5);
 }
        boost:hover {
            transform: scale(1.15);
            box-shadow: 0 0 20px rgba(0,255,255,0.8);
        }
.btn {
  border-radius: 0;
  margin: 0;
}
.btn:first-child {
  border-top-left-radius: 20px;
  border-bottom-left-radius: 20px;
}
.btn:last-child {
  border-top-right-radius: 20px;
  border-bottom-right-radius: 20px;
}
   /* Контейнер (аналог label) */
.toggle-switch {
  position: absolute;
  left: 38%;
  top: 0px;
  display: inline-flex;
  align-items: center;
  cursor: pointer;
  z-index: 999;
  user-select: none; /* Чтобы текст не выделялся при кликах */
}

/* Скрытый инпут (аналог sr-only) */
.toggle-input {
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
.toggle-input:focus + .toggle-track {
  outline: none;
  box-shadow: 0 0 0 2px rgba(147, 196, 253, 0.498); /* ring-4 ring-blue-300 */
}

/* Состояние "Включено" (синий фон) */
.toggle-input:checked + .toggle-track {
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
.toggle-input:checked + .toggle-track::after {
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
</style>