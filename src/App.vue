<template>
  <div class="pomodoro-app">
    <h1>Vue 3 番茄钟</h1>
    
    <div class="timer-container">
      <CircularTimer 
        :currentTime="timeLeft" 
        :totalTime="totalTime"
      />
    </div>
    
    <div class="session-type">{{ currentSession === 'work' ? '工作时段' : '休息时段' }}</div>
    
    <div class="controls">
      <button @click="startTimer" :disabled="isRunning">开始</button>
      <button @click="pauseTimer" :disabled="!isRunning">暂停</button>
      <button @click="resetTimer">重置</button>
    </div>
    
    <div class="settings">
      <div class="setting-item">
        <label>工作时长（分钟）：</label>
        <input type="number" v-model="workDuration" min="1" max="60" @change="updateWorkDuration">
      </div>
      <div class="setting-item">
        <label>休息时长（分钟）：</label>
        <input type="number" v-model="breakDuration" min="1" max="30" @change="updateBreakDuration">
      </div>
    </div>
    
    <div class="stats">
      <p>今日已完成番茄数：{{ completedPomodoros }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import CircularTimer from './CircularTimer.vue'

// 状态管理
const workDuration = ref(25)
const breakDuration = ref(5)
const timeLeft = ref(25 * 60)

// 计算总时间
const totalTime = computed(() => {
  return currentSession.value === 'work' 
    ? workDuration.value * 60 
    : breakDuration.value * 60
})
const isRunning = ref(false)
const currentSession = ref('work') // 'work' 或 'break'
const completedPomodoros = ref(0)

let timerInterval = null

// 格式化时间
const formatTime = (seconds) => {
  const mins = Math.floor(seconds / 60)
  const secs = seconds % 60
  return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
}

// 开始计时器
const startTimer = () => {
  if (!isRunning.value) {
    isRunning.value = true
    timerInterval = setInterval(() => {
      if (timeLeft.value > 0) {
        timeLeft.value--
      } else {
        // 计时器结束
        clearInterval(timerInterval)
        isRunning.value = false
        
        // 播放提示音和显示通知
        playNotification()
        showNotification()
        
        // 切换时段
        if (currentSession.value === 'work') {
          completedPomodoros.value++
          currentSession.value = 'break'
          timeLeft.value = breakDuration.value * 60
        } else {
          currentSession.value = 'work'
          timeLeft.value = workDuration.value * 60
        }
      }
    }, 1000)
  }
}

// 暂停计时器
const pauseTimer = () => {
  if (isRunning.value) {
    clearInterval(timerInterval)
    isRunning.value = false
  }
}

// 重置计时器
const resetTimer = () => {
  clearInterval(timerInterval)
  isRunning.value = false
  currentSession.value = 'work'
  timeLeft.value = workDuration.value * 60
}

// 更新工作时长
const updateWorkDuration = () => {
  if (currentSession.value === 'work' && !isRunning.value) {
    timeLeft.value = workDuration.value * 60
  }
}

// 更新休息时长
const updateBreakDuration = () => {
  if (currentSession.value === 'break' && !isRunning.value) {
    timeLeft.value = breakDuration.value * 60
  }
}

// 播放提示音
const playNotification = () => {
  // 使用 Web Audio API 播放提示音
  const audioContext = new (window.AudioContext || window.webkitAudioContext)()
  const oscillator = audioContext.createOscillator()
  const gainNode = audioContext.createGain()
  
  oscillator.connect(gainNode)
  gainNode.connect(audioContext.destination)
  
  oscillator.frequency.value = 800
  oscillator.type = 'sine'
  gainNode.gain.value = 0.3
  
  oscillator.start()
  gainNode.gain.exponentialRampToValueAtTime(0.001, audioContext.currentTime + 0.5)
  oscillator.stop(audioContext.currentTime + 0.5)
}

// 显示浏览器通知
const showNotification = () => {
  if ('Notification' in window) {
    Notification.requestPermission().then(permission => {
      if (permission === 'granted') {
        new Notification('番茄钟', {
          body: currentSession.value === 'work' ? '工作时间到！休息一下吧。' : '休息时间到！继续工作吧。',
          icon: '/favicon.ico'
        })
      }
    })
  }
}

// 组件卸载时清除计时器
onUnmounted(() => {
  clearInterval(timerInterval)
})
</script>

<style scoped>
.pomodoro-app {
  max-width: 600px;
  margin: 0 auto;
  padding: 2rem;
  text-align: center;
  font-family: Arial, sans-serif;
}

h1 {
  color: #333;
  margin-bottom: 2rem;
}

.timer-container {
  margin: 2rem 0;
  display: flex;
  justify-content: center;
}

.session-type {
  font-size: 1.2rem;
  color: #666;
  margin-bottom: 1rem;
}

.controls {
  margin: 1.5rem 0;
}

button {
  padding: 0.5rem 1rem;
  margin: 0 0.5rem;
  font-size: 1rem;
  cursor: pointer;
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
}

button:hover {
  background-color: #2980b9;
}

button:disabled {
  background-color: #bdc3c7;
  cursor: not-allowed;
}

.settings {
  margin: 2rem 0;
}

.setting-item {
  margin: 0.5rem 0;
}

.setting-item label {
  margin-right: 0.5rem;
}

.setting-item input {
  padding: 0.25rem;
  width: 60px;
}

.stats {
  margin-top: 2rem;
  padding-top: 1rem;
  border-top: 1px solid #eee;
}

.stats p {
  color: #666;
}
</style>