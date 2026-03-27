<template>
  <div class="circular-timer">
    <svg viewBox="0 0 200 200" class="timer-svg">
      <!-- 背景圆 -->
      <circle
        cx="100"
        cy="100"
        r="90"
        fill="none"
        stroke="#e0e0e0"
        stroke-width="10"
      />
      <!-- 进度圆 -->
      <circle
        cx="100"
        cy="100"
        r="90"
        fill="none"
        stroke="#3498db"
        stroke-width="10"
        stroke-linecap="round"
        :stroke-dasharray="circumference"
        :stroke-dashoffset="dashOffset"
        class="progress-circle"
      />
      <!-- 中心文字 -->
      <text x="100" y="100" text-anchor="middle" dy="0.3em" class="timer-text" transform="rotate(90, 100, 100)">
        {{ displayTime }}
      </text>
    </svg>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  currentTime: {
    type: Number,
    required: true
  },
  totalTime: {
    type: Number,
    required: true
  }
})

const radius = 90
const circumference = 2 * Math.PI * radius

// 计算进度偏移量
const dashOffset = computed(() => {
  const progress = props.currentTime / props.totalTime
  return circumference * (1 - progress)
})

// 格式化显示时间
const displayTime = computed(() => {
  const minutes = Math.floor(props.currentTime / 60)
  const seconds = props.currentTime % 60
  return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
})
</script>

<style scoped>
.circular-timer {
  display: flex;
  justify-content: center;
  align-items: center;
}

.timer-svg {
  width: 200px;
  height: 200px;
  transform: rotate(-90deg);
}

.progress-circle {
  transition: stroke-dashoffset 1s linear;
}

.timer-text {
  font-size: 2rem;
  font-weight: bold;
  fill: #333;
}
</style>