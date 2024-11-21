<!-- Timer.vue -->
<template>
  <div class="timer-container" :class="{ 'timer-warning': timeLeft <= 60 && timeLeft > 0, 'timer-overtime': timeLeft < 0 }">
    <!-- Zeit-Einstellung (nur sichtbar wenn Timer nicht läuft) -->
    <div v-if="!isRunning" class="time-settings">
      <div class="settings-group">
        <label>Minuten:</label>
        <input 
          type="number" 
          v-model="minutes" 
          min="0" 
          max="999"
          @change="updateTimer"
        >
      </div>
      <div class="settings-group">
        <label>Sekunden:</label>
        <input 
          type="number" 
          v-model="seconds" 
          min="0" 
          max="59"
          @change="updateTimer"
        >
      </div>
    </div>
    
    <!-- Timer Display -->
    <div class="time" :style="dynamicFontSize">
      <span v-if="timeLeft < 0">-</span>{{ formatTime(Math.abs(timeLeft)) }}
    </div>
    
    <!-- Steuerung -->
    <div class="controls">
      <button @click="startTimer" :disabled="isRunning">Start</button>
      <button @click="pauseTimer" :disabled="!isRunning">Pause</button>
      <button @click="resetTimer">Reset</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Timer',
  data() {
    return {
      timeLeft: 420,
      isRunning: false,
      timerId: null,
      minutes: 7,
      seconds: 0,
      containerWidth: 0,
      containerHeight: 0,
      initialTime: 420,
    }
  },
  mounted() {
    this.updateContainerSize()
    window.addEventListener('resize', this.updateContainerSize)
  },
  beforeDestroy() {
    if (this.timerId) {
      clearInterval(this.timerId)
    }
    window.removeEventListener('resize', this.updateContainerSize)
  },
  computed: {
    dynamicFontSize() {
      const timeString = this.formatTime(Math.abs(this.timeLeft))
      const extraChar = this.timeLeft < 0 ? 1 : 0 // Berücksichtige das Minus-Zeichen
      const containerWidth = this.containerWidth
      const containerHeight = this.containerHeight
      
      const fontSize = Math.min(
        containerWidth / ((timeString.length + extraChar) * 0.6),
        containerHeight / 1.5
      )
      
      return {
        fontSize: `${fontSize}px`,
        lineHeight: '1.1'
      }
    }
  },
  methods: {
    updateContainerSize() {
      this.containerWidth = window.innerWidth * 0.9
      this.containerHeight = window.innerHeight * 0.9
    },
    startTimer() {
      if (!this.isRunning) {
        this.isRunning = true
        this.timerId = setInterval(() => {
          this.timeLeft--
        }, 1000)
      }
    },
    pauseTimer() {
      this.isRunning = false
      clearInterval(this.timerId)
    },
    resetTimer() {
      this.pauseTimer()
      this.updateTimer()
    },
    updateTimer() {
      this.minutes = Math.min(Math.max(parseInt(this.minutes) || 0, 0), 999)
      this.seconds = Math.min(Math.max(parseInt(this.seconds) || 0, 0), 59)
      this.timeLeft = (this.minutes * 60) + this.seconds
      this.initialTime = this.timeLeft
    },
    formatTime(seconds) {
      const mins = Math.floor(seconds / 60)
      const secs = seconds % 60
      return `${mins}:${secs.toString().padStart(2, '0')}`
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  margin: 0;
  overflow: hidden;
}
</style>

<style scoped>
.timer-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: #000;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  transition: background-color 0.5s;
}

.timer-warning {
  background-color: #831414;
}

.timer-overtime {
  background-color: #dc2b2b;
}

.time {
  color: white;
  font-family: monospace;
  text-align: center;
  width: 100%;
}

.controls {
  position: fixed;
  bottom: 2rem;
  display: flex;
  gap: 1rem;
  justify-content: center;
}

button {
  padding: 1rem 2rem;
  font-size: max(1.5rem, min(3vw, 2rem));
  border: none;
  border-radius: 0.5rem;
  background-color: #333;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover:not(:disabled) {
  background-color: #444;
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.time-settings {
  position: fixed;
  top: 2rem;
  display: flex;
  gap: 2rem;
}

.settings-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
}

label {
  color: white;
  font-size: max(1rem, min(2vw, 1.5rem));
}

input {
  width: 5em;
  padding: 0.5rem;
  font-size: max(1rem, min(2vw, 1.5rem));
  text-align: center;
  background-color: #333;
  color: white;
  border: 1px solid #666;
  border-radius: 0.5rem;
}

input:focus {
  outline: none;
  border-color: #888;
}

@media (orientation: landscape) {
  .time-settings {
    flex-direction: row;
  }
}

@media (orientation: portrait) {
  .time-settings {
    flex-direction: column;
  }
}
</style>