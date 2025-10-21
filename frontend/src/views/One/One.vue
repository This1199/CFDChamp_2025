<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { useStarStore } from '@/stores/useStartStore'
import BtnStar from '@/components/BTN/BtnStar.vue'
import MyRandom from '@/components/Random/MyRandom.vue'
import { storeToRefs } from 'pinia'
import MyLinerRegister from '@/components/LineRegister/MyLinerRegister.vue'
import '@/assets/fonts/fonts.css'

const visibleComponents = ref([])
const isAnimating = ref(false)
const spaceVideoRef = ref(null)
const sunVideoRef = ref(null)
const isSpaceVideoLoaded = ref(false)
const isSunVideoLoaded = ref(false)
const isSpaceVideoPlaying = ref(false)
const isSunVideoPlaying = ref(false)
const currentProgress = ref(0)
const currentVideo = ref('space') // 'space' или 'sun'
const startSdvig = ref(false)


// Аудио ссылки и состояния
const audioRefs = {
  1: ref(null),
  2: ref(null),
  3: ref(null),
  4: ref(null)
}

const audioStates = {
  1: ref(false),
  2: ref(false),
  3: ref(false),
  4: ref(false)
}

// Добавьте эти переменные для анимации числа
const showNumber = ref(false)
const numberPosition = ref({ y: '50%', opacity: 1 })

// Функция для показа числа
const showCenterNumber = () => {
  showNumber.value = true
  numberPosition.value = { y: '50%', opacity: 1 }
  console.log("OK")
  // Через секунду начинаем анимацию исчезновения
  setTimeout(() => {
    numberPosition.value = { y: '400%', opacity: 0 }
    
    // После завершения анимации скрываем элемент
    setTimeout(() => {
      showNumber.value = false
    }, 1000)
  }, 1000)
}

// Добавьте переменную для отслеживания первого показа
const isFirstSunPlay = ref(true)

// Обработчик начала воспроизведения Sun видео
const onSunVideoPlay = () => {
  console.log('Sun видео началось')
  
  // Показываем число только при первом запуске
  if (isFirstSunPlay.value) {
    console.log('Первый запуск Sun видео, показываем число')
    showCenterNumber()
    isFirstSunPlay.value = false
  }
}

const chislo = ref(0xB211)

// Инициализируем хранилище
const dataStore = useStarStore()
const { componentsData } = storeToRefs(dataStore)

// Массив для отслеживания занятых позиций
const occupiedPositions = ref([])

// Функция для проверки пересечения позиций
const isPositionOccupied = (newPos, existingPositions) => {
  const newLeft = parseInt(newPos.left)
  const newTop = parseInt(newPos.top)
  const componentWidth = 300
  const componentHeight = 200
  
  for (const pos of existingPositions) {
    const existingLeft = parseInt(pos.left)
    const existingTop = parseInt(pos.top)
    
    const horizontalOverlap = 
      newLeft < existingLeft + componentWidth && 
      newLeft + componentWidth > existingLeft
      
    const verticalOverlap = 
      newTop < existingTop + componentHeight && 
      newTop + componentHeight > existingTop
      
    if (horizontalOverlap && verticalOverlap) {
      return true
    }
  }
  return false
}

// Функция для получения случайной позиции без наложения
const getRandomPosition = () => {
  const containerWidth = 1200
  const containerHeight = 600
  const componentWidth = 300
  const componentHeight = 200
  
  const maxLeft = containerWidth - componentWidth
  const maxTop = containerHeight - componentHeight
  
  let attempts = 0
  const maxAttempts = 100
  
  while (attempts < maxAttempts) {
    const left = Math.max(0, Math.floor(Math.random() * maxLeft))
    const top = Math.max(0, Math.floor(Math.random() * maxTop))
    
    const newPosition = {
      left: left + 'px',
      top: top + 'px'
    }
    
    if (!isPositionOccupied(newPosition, occupiedPositions.value)) {
      occupiedPositions.value.push(newPosition)
      return newPosition
    }
    
    attempts++
  }
  
  console.warn('Не удалось найти свободную позицию после', maxAttempts, 'попыток')
  return {
    left: (containerWidth / 2 - componentWidth / 2) + 'px',
    top: (containerHeight / 2 - componentHeight / 2) + 'px'
  }
}

// Функция для получения данных компонента из хранилища
const getComponentData = (index) => {
  const data = componentsData.value
  
  if (data && data.length > 0) {
    const dataIndex = index % data.length
    return data[dataIndex]
  }
  
  return {
    title: `Компонент ${index + 1}`,
    description: `Это описание для компонента ${index + 1}`,
    features: [
      `Функция ${index + 1}.1`,
      `Функция ${index + 1}.2`,
      `Функция ${index + 1}.3`,
      `Функция ${index + 1}.4`
    ]
  }
}

// Управление аудио
const playAudio = async (audioNumber) => {
  const audioRef = audioRefs[audioNumber]
  const audioState = audioStates[audioNumber]
  
  if (audioRef.value && !audioState.value) {
    try {
      audioRef.value.currentTime = 0
      await audioRef.value.play()
      audioState.value = true
      console.log(`Аудио ${audioNumber} запущено`)
    } catch (error) {
      console.log(`Ошибка воспроизведения аудио ${audioNumber}:`, error)
      audioState.value = false
    }
  }
}

const stopAudio = (audioNumber) => {
  const audioRef = audioRefs[audioNumber]
  const audioState = audioStates[audioNumber]
  
  if (audioRef.value) {
    audioRef.value.pause()
    audioRef.value.currentTime = 0
    audioState.value = false
  }
}

const stopAllAudio = () => {
  Object.keys(audioRefs).forEach(audioNumber => {
    stopAudio(audioNumber)
  })
  console.log('Все аудио остановлены')
}

// Обработчики окончания аудио
const onAudioEnded = (audioNumber) => {
  audioStates[audioNumber].value = false
  console.log(`Аудио ${audioNumber} завершено`)
  switch (audioNumber)
  {
    case 1: playAudio(2); 

    window.scrollTo({
    top: 900,
    behavior: 'smooth'})
    setTimeout(() => {
    startSdvig.value = true
    }, 2000)
    break;
    case 2: playAudio(3); 
    window.scrollTo({
    top: 0,
    behavior: 'smooth'})
    break;
    case 3: playAudio(4); break;
  }
  
}

// Управление видео
const startSpaceVideo = async () => {
  if (spaceVideoRef.value && !isSpaceVideoPlaying.value) {
    try {
      // Скрываем солнце, показываем космос
      currentVideo.value = 'space'
      await spaceVideoRef.value.play()
      isSpaceVideoPlaying.value = true
      console.log('Space видео запущено')
    } catch (error) {
      console.log('Ошибка воспроизведения Space видео:', error)
      isSpaceVideoPlaying.value = false
    }
  }
}

const startSunVideo = async () => {
  if (sunVideoRef.value && !isSunVideoPlaying.value) {
    try {
      // Скрываем космос, показываем солнце
      currentVideo.value = 'sun'
      await sunVideoRef.value.play()
      isSunVideoPlaying.value = true
      console.log('Sun видео запущено')
      
      // ВСЕГДА показываем число при запуске Sun видео
      if (isFirstSunPlay.value) {
        console.log('Показываем число')
        showCenterNumber()
        isFirstSunPlay.value = false
      }
    } catch (error) {
      console.log('Ошибка воспроизведения Sun видео:', error)
      isSunVideoPlaying.value = false
    }
  }
}

const stopAllVideos = () => {
  if (spaceVideoRef.value) {
    spaceVideoRef.value.pause()
    spaceVideoRef.value.currentTime = 0
    isSpaceVideoPlaying.value = false
  }
  if (sunVideoRef.value) {
    sunVideoRef.value.pause()
    sunVideoRef.value.currentTime = 0
    isSunVideoPlaying.value = false
  }
  console.log('Все видео остановлены')
}

// Обработчики загрузки видео
const onSpaceVideoLoad = () => {
  isSpaceVideoLoaded.value = true
  console.log('Space видео загружено')
}

const onSunVideoLoad = () => {
  isSunVideoLoaded.value = true
  console.log('Sun видео загружено')
}

const onVideoError = (videoType) => {
  console.error(`Ошибка загрузки ${videoType} видео`)
  if (videoType === 'space') {
    isSpaceVideoLoaded.value = false
  } else {
    isSunVideoLoaded.value = false
  }
}

const showFlash = ref(false)

const onSpaceVideoEnded = () => {
  console.log('Space видео закончилось, запускаем Sun видео')
  isSpaceVideoPlaying.value = false
  
  // Показываем красную вспышку
  showFlash.value = true
  setTimeout(() => {
    showFlash.value = false
    startSunVideo()
  }, 300)
}

const onSunVideoEnded = () => {
  console.log('Sun видео закончилось, зацикливаем его')
  // Зацикливаем Sun видео
  if (sunVideoRef.value) {
    sunVideoRef.value.currentTime = 0
    sunVideoRef.value.play().catch(error => {
      console.log('Ошибка при перезапуске Sun видео:', error)
    })
  }
}

const showMultipleRandom = async () => {
  if (isAnimating.value) return
  
  // ЗАПУСКАЕМ ПЕРВОЕ АУДИО ПРИ НАЧАЛЕ АНИМАЦИИ
  
  await playAudio(1)
  setTimeout(async () => {
    window.scrollTo({
    top: 0,
    behavior: 'smooth'
  })
    
  // СБРОСИТЬ ФЛАГ ПЕРЕД КАЖДЫМ ЗАПУСКОМ
  isFirstSunPlay.value = true
  
  // Запускаем Space видео
  await startSpaceVideo()
  
  isAnimating.value = true
  visibleComponents.value = []
  occupiedPositions.value = []
  currentProgress.value = 0
  
  // Ждем завершения анимации числа перед показом компонентов
  await new Promise(resolve => setTimeout(resolve, 2100))
  
  const totalComponents = 10
  
  for (let i = 0; i < totalComponents; i++) {
    currentProgress.value = i + 1
    
    if (i > 0) {
      await new Promise(resolve => {
        const checkDisappearance = () => {
          if (visibleComponents.value.length === 0 || 
              !visibleComponents.value[visibleComponents.value.length - 1].visible) {
            resolve()
          } else {
            setTimeout(checkDisappearance, 100)
          }
        }
        checkDisappearance()
      })
      
      await new Promise(resolve => setTimeout(resolve, 500))
    }
    
    const componentData = getComponentData(i)
    
    const newComponent = {
      id: Date.now() + i,
      visible: true,
      position: getRandomPosition(),
      title: componentData.title,
      description: componentData.description,
      features: componentData.features,
      timer: null
    }
    
    visibleComponents.value.push(newComponent)
    
    newComponent.timer = setTimeout(() => {
      const index = visibleComponents.value.findIndex(comp => comp.id === newComponent.id)
      if (index !== -1) {
        visibleComponents.value[index].visible = false
        
        setTimeout(() => {
          const removeIndex = visibleComponents.value.findIndex(comp => comp.id === newComponent.id)
          if (removeIndex !== -1) {
            const positionIndex = occupiedPositions.value.findIndex(pos => 
              pos.left === newComponent.position.left && pos.top === newComponent.position.top
            )
            if (positionIndex !== -1) {
              occupiedPositions.value.splice(positionIndex, 1)
            }
            visibleComponents.value.splice(removeIndex, 1)
          }
        }, 600)
      }
    }, 3000)
  }
  
  await new Promise(resolve => setTimeout(resolve, 4000))
  
  currentProgress.value = 0
  isAnimating.value = false
  // Не останавливаем видео - Sun видео продолжает зацикленно играть
}, 9000)
}

// Сбросьте флаг при остановке анимации
const pauseAnimation = () => {
  if (isAnimating.value) {
    isAnimating.value = false
    currentProgress.value = 0
    isFirstSunPlay.value = true // Сбрасываем флаг
    
    visibleComponents.value.forEach(component => {
      if (component.timer) {
        clearTimeout(component.timer)
      }
    })
    
    visibleComponents.value = []
    occupiedPositions.value = []
    stopAllVideos()
    stopAllAudio() // Останавливаем все аудио
  }
}

// Жизненный цикл
onMounted(() => {
  console.log('Компонент монтирован - видео готовы к запуску')
  console.log('Данные из хранилища:', componentsData.value)
})

onUnmounted(() => {
  pauseAnimation()
  if (spaceVideoRef.value) {
    spaceVideoRef.value.src = ''
  }
  if (sunVideoRef.value) {
    sunVideoRef.value.src = ''
  }
})
</script>

<template>
  <div class="one" data-aos="zoom-in">
    <!-- Аудио элементы -->
    <audio
      :ref="audioRefs[1]"
      preload="auto"
      @ended="onAudioEnded(1)"
      @error="console.error('Ошибка загрузки аудио 1')"
    >
      <source src="@/assets/audio/1.mp3" type="audio/mpeg">
      Ваш браузер не поддерживает аудио элементы.
    </audio>
    
    <audio
      :ref="audioRefs[2]"
      preload="auto"
      @ended="onAudioEnded(2)"
      @error="console.error('Ошибка загрузки аудио 2')"
    >
      <source src="@/assets/audio/2.mp3" type="audio/mpeg">
      Ваш браузер не поддерживает аудио элементы.
    </audio>
    
    <audio
      :ref="audioRefs[3]"
      preload="auto"
      @ended="onAudioEnded(3)"
      @error="console.error('Ошибка загрузки аудио 3')"
    >
      <source src="@/assets/audio/3.mp3" type="audio/mpeg">
      Ваш браузер не поддерживает аудио элементы.
    </audio>
    
    <audio
      :ref="audioRefs[4]"
      preload="auto"
      @ended="onAudioEnded(4)"
      @error="console.error('Ошибка загрузки аудио 4')"
    >
      <source src="@/assets/audio/4.mp3" type="audio/mpeg">
      Ваш браузер не поддерживает аудио элементы.
    </audio>
    
    <div class="random-container">
      <!-- Видеофоны -->
      <div class="video-background">
        <!-- Space видео (первое) -->
        <video
          ref="spaceVideoRef"
          muted
          playsinline
          preload="auto"
          @loadeddata="onSpaceVideoLoad"
          @error="onVideoError('space')"
          @ended="onSpaceVideoEnded"
          class="background-video space-video"
          :class="{ 
            'video-playing': isSpaceVideoPlaying,
            'video-visible': currentVideo === 'space',
            'video-hidden': currentVideo !== 'space'
          }"
        >
          <source src="@/assets/Space.mp4" type="video/mp4">
        </video>
        
        <div 
          v-if="showFlash"
          class="video-transition-overlay"
        ></div>

        <!-- Sun видео (второе, зацикленное) -->
        <video
          ref="sunVideoRef"
          muted
          loop
          playsinline
          preload="auto"
          @loadeddata="onSunVideoLoad"
          @error="onVideoError('sun')"
          @ended="onSunVideoEnded"
          @play="onSunVideoPlay" 
          class="background-video sun-video"
          :class="{ 
            'video-playing': isSunVideoPlaying,
            'video-visible': currentVideo === 'sun',
            'video-hidden': currentVideo !== 'sun'
          }"
        >
          <source src="@/assets/Sun.mp4" type="video/mp4">
        </video>
        
        <!-- Анимированное число -->
        <div 
          v-if="showNumber"
          class="center-number"
          :style="{
            transform: `translate(-50%, ${numberPosition.y})`,
            opacity: numberPosition.opacity
          }"
        >
          {{chislo.toString(2).toUpperCase()}}
        </div>
        
        <!-- Индикаторы состояния видео -->
        <div v-if="!isSpaceVideoLoaded && !isSpaceVideoPlaying" class="video-loading">
          <div class="loading-spinner"></div>
          Загрузка Space видео...
        </div>
        
        <div v-else-if="!isSunVideoLoaded && !isSunVideoPlaying && currentVideo === 'sun'" class="video-loading">
          <div class="loading-spinner"></div>
          Загрузка Sun видео...
        </div>
      </div>
      
      <!-- Компоненты поверх видео -->
      <TransitionGroup name="stagger">
        <MyRandom 
          v-for="component in visibleComponents" 
          :key="component.id"
          :title="component.title"
          :description="component.description"
          :features="component.features"
          :class="['random-item', { 'visible': component.visible }]"
          :style="component.position"
        />
      </TransitionGroup>
      
      <!-- Счетчик и прогресс -->
      <div v-if="isAnimating" class="animation-progress">
        <div class="progress-text">
          Прогресс: {{ currentProgress }}/10
        </div>
        <div class="progress-bar">
          <div 
            class="progress-fill" 
            :style="{ width: (currentProgress / 10) * 100 + '%' }"
          ></div>
        </div>
        <div class="progress-details">
          <span class="detail-item">Текущий: {{ currentProgress }}</span>
          <span class="detail-item">Видимых: {{ visibleComponents.filter(c => c.visible).length }}</span>
          <span class="detail-item" :class="{ 
            'video-space': currentVideo === 'space',
            'video-sun': currentVideo === 'sun'
          }">
            Видео: {{ currentVideo === 'space' ? 'Space' : 'Sun' }}
          </span>
          <!-- Индикаторы аудио -->
          <span 
            v-for="i in 4" 
            :key="i"
            class="detail-item audio-indicator"
            :class="{ 'audio-playing': audioStates[i].value }"
          >
            Аудио{{ i }}: {{ audioStates[i].value ? '▶' : '⏸' }}
          </span>
        </div>
      </div>
    </div>
    
    <div class="controls">
      <BtnStar 
        variant="secondary" 
        :text="isAnimating ? `Генерация... (${currentProgress}/10)` : 'Запустить последовательность'" 
        size="medium"
        :disabled="isAnimating"
        @click="showMultipleRandom"
      /> 
    </div>
    <div>
      <MyLinerRegister
        :chislo="chislo"
        :startSdvig="startSdvig"
      />
    </div>
  </div>
</template>

<style scoped>
.center-number {
  position: absolute;
  top: 250px;
  left: 50%;
  font-size: 40px;
  font-weight: 900;
  color: white;
  z-index: 5;
  transition: all 1s cubic-bezier(0.4, 0, 0.2, 1);
  pointer-events: none;
  font-family: 'Atmospheric', sans-serif;
  background: linear-gradient(135deg, #6366f1, #8b5cf6, #ec4899);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.video-transition-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: radial-gradient(circle,rgba(255, 236, 94, 0.811) 0%, rgba(0, 0, 0, 0) 50%);
  z-index: 3;
  opacity: 0;
  pointer-events: none;
  animation: redFlash 0.6s ease-in-out;
}

@keyframes redFlash {
  0% {
    opacity: 0;
    transform: scale(0.8);
  }
  50% {
    opacity: 0.8;
    transform: scale(1);
  }
  100% {
    opacity: 0;
    transform: scale(1.2);
  }
}

.one {
  position: relative;
  min-height: 70vh;
  padding: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
}

.random-container {
  position: relative;
  width: 90vw;
  height: 75vh;
  border: 2px dashed rgba(99, 102, 241, 0.5);
  border-radius: 12px;
  margin: 20px 0;
  background: rgba(99, 102, 241, 0.05);
  overflow: hidden;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
}

/* Стили для видеофона */
.video-background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  overflow: hidden;
}

.background-video {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: center;
  transition: opacity 0.8s ease-in-out;
  position: absolute;
  top: 0;
  left: 0;
}

.space-video,
.sun-video {
  opacity: 1;
}

.video-visible {
  opacity: 1;
  z-index: 1;
}

.video-hidden {
  opacity: 0;
  z-index: 0;
}

/* Индикаторы состояния видео */
.video-loading {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: rgba(0, 0, 0, 0.8);
  color: white;
  padding: 15px 25px;
  border-radius: 10px;
  z-index: 2;
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 14px;
}

.loading-spinner {
  width: 20px;
  height: 20px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-top: 2px solid white;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* Улучшенный прогресс анимации */
.animation-progress {
  position: absolute;
  top: 10px;
  left: 10px;
  background: rgba(0, 0, 0, 0.85);
  color: rgb(255, 255, 255);
  padding: 12px 16px;
  border-radius: 10px;
  z-index: 4;
  min-width: 200px;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.progress-text {
  margin-bottom: 8px;
  font-size: 14px;
  font-weight: 600;
}

.progress-bar {
  width: 100%;
  height: 6px;
  background: rgba(255, 255, 255, 0.15);
  border-radius: 3px;
  overflow: hidden;
  margin-bottom: 6px;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #6366f1, #8b5cf6);
  transition: width 0.5s ease;
  border-radius: 3px;
}

.progress-details {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
  font-size: 11px;
  opacity: 0.7;
}

.detail-item {
  padding: 2px 6px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 4px;
}

.detail-item.video-space {
  background: rgba(59, 130, 246, 0.3);
  color: #3b82f6;
}

.detail-item.video-sun {
  background: rgba(234, 179, 8, 0.3);
  color: #eab308;
}

.detail-item.audio-indicator.audio-playing {
  background: rgba(34, 197, 94, 0.3);
  color: #22c55e;
}

/* Стили для компонентов */
.random-item {
  position: absolute;
  transition: all 0.8s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 1;
  width: 300px;
  height: 200px;
  background: rgba(255, 255, 255, 0.15);
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  max-width: calc(100% - 20px);
  max-height: calc(100% - 20px);
}

.random-item.visible {
  z-index: 3;
  box-shadow: 
    0 0 20px rgba(99, 102, 241, 0.6),
    0 0 40px rgba(99, 102, 241, 0.3);
  background: rgba(255, 255, 255, 0.2);
  border: 1px solid rgba(99, 102, 241, 0.5);
}

/* Управляющие кнопки */
.controls {
  display: flex;
  gap: 15px;
  align-items: center;
  margin-top: 20px;
}

/* Анимации */
.stagger-enter-active {
  animation: slide-in 0.6s ease-out both;
}

.stagger-leave-active {
  animation: slide-out 0.6s ease-in both;
  position: absolute !important;
}

@keyframes slide-in {
  0% {
    opacity: 0;
    transform: scale(0.8) translateY(20px);
  }
  100% {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

@keyframes slide-out {
  0% {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
  100% {
    opacity: 0;
    transform: scale(0.8) translateY(-20px);
  }
}

/* Адаптивность */
@media (max-width: 768px) {
  .animation-progress {
    min-width: 180px;
    padding: 10px 12px;
  }
  
  .progress-details {
    flex-direction: column;
    gap: 2px;
  }
}
</style>