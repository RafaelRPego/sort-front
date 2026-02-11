<template>
  <div class="app">
    <h1>Visualizador de <span>Algoritmos</span></h1>

    <div class="controls">
      <button :class="{ active: algo === 'bubble' }" @click="run('bubble')">Bubble</button>
      <button :class="{ active: algo === 'quick' }" @click="run('quick')">Quick</button>
      <button :class="{ active: algo === 'merge' }" @click="run('merge')">Merge</button>
    </div>

    <div class="step-card" v-if="currentStep">
      <div class="badges">
        <span class="badge type">{{ currentStep.type }}</span>
        <span class="badge" v-if="currentStep.depth !== undefined">
          Profundidade {{ currentStep.depth }}
        </span>
      </div>

      <p v-if="currentStep.pivot !== undefined">
        🔮 Pivô: <strong>{{ currentStep.pivot }}</strong>
      </p>

      <p v-if="currentStep.comparing">
        🔍 Comparando: {{ currentStep.comparing.join(' , ') }}
      </p>

      <p v-if="currentStep.swapped">
        🔁 Troca: {{ currentStep.swapped.join(' , ') }}
      </p>
    </div>

    <div v-if="algo === 'merge' && currentStep?.left" class="merge-box">
      <div class="sub">
        <h4>Esquerda</h4>
        <div class="bars">
          <div
            v-for="(v, i) in currentStep.left"
            :key="'l'+i"
            class="bar left"
            :style="{ height: v * scale + 'px' }"
          >{{ v }}</div>
        </div>
      </div>

      <div class="sub">
        <h4>Direita</h4>
        <div class="bars">
          <div
            v-for="(v, i) in currentStep.right"
            :key="'r'+i"
            class="bar right"
            :style="{ height: v * scale + 'px' }"
          >{{ v }}</div>
        </div>
      </div>
    </div>

    <div class="bars main">
      <div
        v-for="(value, index) in currentArray"
        :key="index"
        class="bar"
        :class="barClass(index, value)"
        :style="{ height: value * scale + 'px' }"
      >
        {{ value }}
      </div>
    </div>

    <div class="progress">
      <div
        class="progress-bar"
        :style="{ width: progress + '%' }"
      ></div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import axios from 'axios'

const baseArray = [8, 10, 5, 9, 6, 7, 3, 11, 12, 2, 1, 4]

const algo = ref(null)
const currentArray = ref([])
const currentStep = ref(null)
const steps = ref([])
const index = ref(0)

const scale = 14

const run = async (type) => {
  algo.value = type
  index.value = 0
  currentArray.value = [...baseArray]

  const { data } = await axios.post(
    `http://127.0.0.1:8000/api/sort/${type}`,
    { array: baseArray }
  )

  steps.value = data.context.steps
  animate()
}

const animate = () => {
  const timer = setInterval(() => {
    if (index.value >= steps.value.length) {
      clearInterval(timer)
      return
    }

    currentStep.value = steps.value[index.value]

    if (currentStep.value.array) {
      currentArray.value = currentStep.value.array
    }

    index.value++
  }, 500)
}

const progress = computed(() =>
  steps.value.length
    ? (index.value / steps.value.length) * 100
    : 0
)

const barClass = (i, value) => {
  if (!currentStep.value) return 'normal'

  if (currentStep.value.pivot === value) return 'pivot'
  if (currentStep.value.comparing?.includes(i)) return 'compare'
  if (currentStep.value.swapped?.includes(i)) return 'swap'

  return 'normal'
}
</script>

<style scoped>
.app {
  min-height: 100vh;
  padding: 40px;
  background: radial-gradient(circle at top, #0f172a, #020617);
  color: #e5e7eb;
  font-family: Inter, system-ui;
}

h1 {
  text-align: center;
  margin-bottom: 30px;
}

h1 span {
  color: #34d399;
}

.controls {
  display: flex;
  justify-content: center;
  gap: 12px;
  margin-bottom: 25px;
}

button {
  padding: 10px 22px;
  border-radius: 10px;
  background: #020617;
  border: 1px solid #1f2933;
  color: #9ca3af;
  cursor: pointer;
  transition: 0.3s;
}

button.active,
button:hover {
  background: #34d399;
  color: #022c22;
}

.step-card {
  max-width: 700px;
  margin: 0 auto 30px;
  padding: 16px 20px;
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.04);
  backdrop-filter: blur(12px);
}

.badges {
  display: flex;
  gap: 8px;
  margin-bottom: 8px;
}

.badge {
  padding: 4px 10px;
  border-radius: 999px;
  background: #1f2933;
  font-size: 12px;
}

.badge.type {
  background: #34d399;
  color: #022c22;
}

.merge-box {
  display: flex;
  justify-content: center;
  gap: 30px;
  margin-bottom: 30px;
}

.sub h4 {
  text-align: center;
  margin-bottom: 8px;
  color: #93c5fd;
}

.bars {
  display: flex;
  align-items: flex-end;
  gap: 6px;
}

.bars.main {
  justify-content: center;
  margin-top: 20px;
}

.bar {
  width: 28px;
  border-radius: 8px;
  background: #34d399;
  display: flex;
  justify-content: center;
  align-items: flex-end;
  font-size: 12px;
  transition: 0.3s;
}

.bar.compare {
  background: #ef4444;
}

.bar.swap {
  background: #facc15;
  color: #000;
}

.bar.pivot {
  background: #a855f7;
}

.bar.left {
  background: #60a5fa;
}

.bar.right {
  background: #22d3ee;
}

.progress {
  max-width: 600px;
  height: 6px;
  margin: 40px auto 0;
  background: #1f2933;
  border-radius: 999px;
  overflow: hidden;
}

.progress-bar {
  height: 100%;
  background: #34d399;
  transition: width 0.3s;
}
</style>
