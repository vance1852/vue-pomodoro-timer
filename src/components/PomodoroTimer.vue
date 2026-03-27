<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const workDuration = ref(25)
const breakDuration = ref(5)
const currentMode = ref('work')
const timeLeft = ref(workDuration.value * 60)
const isRunning = ref(false)
const completedPomodoros = ref(0)
let timer = null

const totalTime = computed(() => {
  return currentMode.value === 'work' 
    ? workDuration.value * 60 
    : breakDuration.value * 60
})

const progress = computed(() => {
  return ((totalTime.value - timeLeft.value) / totalTime.value) * 100
})

const displayTime = computed(() => {
  const minutes = Math.floor(timeLeft.value / 60)
  const seconds = timeLeft.value % 60
  return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
})

const circumference = 2 * Math.PI * 120
const strokeDashoffset = computed(() => {
  return circumference - (progress.value / 100) * circumference
})

const startTimer = () => {
  if (!isRunning.value) {
    isRunning.value = true
    timer = setInterval(() => {
      if (timeLeft.value > 0) {
        timeLeft.value--
      } else {
        completePomodoro()
      }
    }, 1000)
  }
}

const pauseTimer = () => {
  isRunning.value = false
  if (timer) {
    clearInterval(timer)
    timer = null
  }
}

const resetTimer = () => {
  pauseTimer()
  timeLeft.value = currentMode.value === 'work' 
    ? workDuration.value * 60 
    : breakDuration.value * 60
}

const completePomodoro = () => {
  pauseTimer()
  playNotificationSound()
  showNotification()
  
  if (currentMode.value === 'work') {
    completedPomodoros.value++
    saveCompletedPomodoros()
    currentMode.value = 'break'
  } else {
    currentMode.value = 'work'
  }
  
  timeLeft.value = currentMode.value === 'work' 
    ? workDuration.value * 60 
    : breakDuration.value * 60
}

const playNotificationSound = () => {
  const audioContext = new (window.AudioContext || window.webkitAudioContext)()
  const oscillator = audioContext.createOscillator()
  const gainNode = audioContext.createGain()
  
  oscillator.connect(gainNode)
  gainNode.connect(audioContext.destination)
  
  oscillator.frequency.value = 800
  oscillator.type = 'sine'
  gainNode.gain.value = 0.3
  
  oscillator.start()
  setTimeout(() => {
    oscillator.stop()
  }, 500)
}

const showNotification = () => {
  if (Notification.permission === 'granted') {
    const message = currentMode.value === 'work' 
      ? '工作时间结束！休息一下吧。' 
      : '休息时间结束！开始工作吧。'
    new Notification('番茄钟', {
      body: message,
      icon: '/favicon.svg'
    })
  }
}

const requestNotificationPermission = () => {
  if (Notification.permission !== 'granted') {
    Notification.requestPermission()
  }
}

const saveCompletedPomodoros = () => {
  const today = new Date().toDateString()
  const data = {
    date: today,
    count: completedPomodoros.value
  }
  localStorage.setItem('pomodoroStats', JSON.stringify(data))
}

const loadCompletedPomodoros = () => {
  const saved = localStorage.getItem('pomodoroStats')
  if (saved) {
    const data = JSON.parse(saved)
    const today = new Date().toDateString()
    if (data.date === today) {
      completedPomodoros.value = data.count
    } else {
      completedPomodoros.value = 0
    }
  }
}

const updateWorkDuration = (value) => {
  workDuration.value = parseInt(value)
  if (currentMode.value === 'work' && !isRunning.value) {
    timeLeft.value = workDuration.value * 60
  }
}

const updateBreakDuration = (value) => {
  breakDuration.value = parseInt(value)
  if (currentMode.value === 'break' && !isRunning.value) {
    timeLeft.value = breakDuration.value * 60
  }
}

onMounted(() => {
  loadCompletedPomodoros()
  requestNotificationPermission()
})

onUnmounted(() => {
  if (timer) {
    clearInterval(timer)
  }
})
</script>

<template>
  <div class="pomodoro-container">
    <h1 class="title">番茄钟</h1>
    
    <div class="stats">
      <span class="stats-label">今日已完成：</span>
      <span class="stats-count">{{ completedPomodoros }}</span>
      <span class="stats-label"> 个番茄</span>
    </div>
    
    <div class="mode-indicator">
      <span :class="{ active: currentMode === 'work' }">工作</span>
      <span class="separator">|</span>
      <span :class="{ active: currentMode === 'break' }">休息</span>
    </div>
    
    <div class="timer-wrapper">
      <svg class="timer-circle" width="280" height="280">
        <circle
          class="timer-circle-bg"
          cx="140"
          cy="140"
          r="120"
        />
        <circle
          class="timer-circle-progress"
          cx="140"
          cy="140"
          r="120"
          :stroke-dasharray="circumference"
          :stroke-dashoffset="strokeDashoffset"
          :class="{ work: currentMode === 'work', break: currentMode === 'break' }"
        />
      </svg>
      <div class="timer-display">
        <div class="time">{{ displayTime }}</div>
        <div class="mode-text">{{ currentMode === 'work' ? '工作中' : '休息中' }}</div>
      </div>
    </div>
    
    <div class="controls">
      <button v-if="!isRunning" @click="startTimer" class="btn btn-start">
        开始
      </button>
      <button v-else @click="pauseTimer" class="btn btn-pause">
        暂停
      </button>
      <button @click="resetTimer" class="btn btn-reset">
        重置
      </button>
    </div>
    
    <div class="settings">
      <div class="setting-item">
        <label>工作时长（分钟）：</label>
        <input
          type="number"
          :value="workDuration"
          @input="updateWorkDuration($event.target.value)"
          min="1"
          max="60"
          :disabled="isRunning"
        />
      </div>
      <div class="setting-item">
        <label>休息时长（分钟）：</label>
        <input
          type="number"
          :value="breakDuration"
          @input="updateBreakDuration($event.target.value)"
          min="1"
          max="30"
          :disabled="isRunning"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.pomodoro-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 24px;
  padding: 40px 20px;
}

.title {
  font-size: 32px;
  font-weight: 600;
  margin: 0;
  color: var(--text-h);
}

.stats {
  font-size: 18px;
  color: var(--text);
}

.stats-count {
  font-weight: 600;
  color: var(--accent);
  font-size: 24px;
}

.mode-indicator {
  font-size: 18px;
  color: var(--text);
  display: flex;
  gap: 8px;
}

.mode-indicator span {
  transition: color 0.3s;
}

.mode-indicator .active {
  color: var(--accent);
  font-weight: 600;
}

.timer-wrapper {
  position: relative;
  width: 280px;
  height: 280px;
}

.timer-circle {
  transform: rotate(-90deg);
}

.timer-circle-bg {
  fill: none;
  stroke: var(--border);
  stroke-width: 12;
}

.timer-circle-progress {
  fill: none;
  stroke-width: 12;
  stroke-linecap: round;
  transition: stroke-dashoffset 0.3s;
}

.timer-circle-progress.work {
  stroke: #ff6b6b;
}

.timer-circle-progress.break {
  stroke: #4ecdc4;
}

.timer-display {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
}

.time {
  font-size: 48px;
  font-weight: 700;
  color: var(--text-h);
  font-family: var(--mono);
}

.mode-text {
  font-size: 18px;
  color: var(--text);
  margin-top: 8px;
}

.controls {
  display: flex;
  gap: 16px;
}

.btn {
  padding: 12px 32px;
  font-size: 16px;
  font-weight: 600;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
}

.btn-start {
  background: var(--accent);
  color: white;
}

.btn-start:hover {
  background: #9933e6;
}

.btn-pause {
  background: #ff6b6b;
  color: white;
}

.btn-pause:hover {
  background: #ff5252;
}

.btn-reset {
  background: var(--border);
  color: var(--text);
}

.btn-reset:hover {
  background: #d1d1d1;
}

.settings {
  display: flex;
  flex-direction: column;
  gap: 16px;
  margin-top: 24px;
}

.setting-item {
  display: flex;
  align-items: center;
  gap: 12px;
}

.setting-item label {
  font-size: 16px;
  color: var(--text);
  min-width: 140px;
}

.setting-item input {
  width: 80px;
  padding: 8px 12px;
  border: 2px solid var(--border);
  border-radius: 6px;
  font-size: 16px;
  background: var(--bg);
  color: var(--text-h);
}

.setting-item input:focus {
  outline: none;
  border-color: var(--accent);
}

.setting-item input:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

@media (max-width: 640px) {
  .title {
    font-size: 28px;
  }
  
  .time {
    font-size: 40px;
  }
  
  .controls {
    flex-wrap: wrap;
    justify-content: center;
  }
  
  .btn {
    padding: 10px 24px;
    font-size: 14px;
  }
}
</style>
