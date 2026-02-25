<template>
  <div class="app">
    <div class="container">
      <header class="header">
        <h1 class="title">
          Visualizador de <span class="gradient-text">Algoritmos</span>
        </h1>
        <p class="subtitle">
          Explore algoritmos de ordenação de forma interativa
        </p>
      </header>

      <div class="section">
        <h2 class="section-title">Selecione o Algoritmo</h2>
        <div class="algorithm-grid">
          <button
            v-for="alg in algorithms"
            :key="alg"
            :class="['algorithm-btn', { active: selected === alg }]"
            @click="selectAlgorithm(alg)"
          >
            <div class="algorithm-icon">{{ getAlgorithmIcon(alg) }}</div>
            <span class="algorithm-name">{{ getAlgorithmName(alg) }}</span>
          </button>
        </div>
      </div>

      <div class="section">
        <div class="control-panel">
          <button class="btn btn-primary" @click="run" :disabled="loading">
            <span v-if="!loading">▶ Executar</span>
            <span v-else class="loading-dots">Carregando</span>
          </button>

          <div class="playback-controls">
            <button
              class="btn btn-icon"
              @click="prevStep"
              :disabled="stepIndex === 0 || !steps.length"
              title="Passo anterior"
            >
              ◀
            </button>
            <button
              class="btn btn-secondary"
              @click="togglePlay"
              :disabled="!steps.length"
            >
              {{ playing ? "⏸ Pausar" : "▶ Reproduzir" }}
            </button>
            <button
              class="btn btn-icon"
              @click="nextStep"
              :disabled="stepIndex >= steps.length - 1 || !steps.length"
              title="Próximo passo"
            >
              ▶
            </button>
          </div>

          <div class="speed-control" v-if="steps.length">
            <label class="speed-label">Velocidade:</label>
            <div class="speed-buttons">
              <button
                v-for="spd in speedOptions"
                :key="spd.value"
                :class="['speed-btn', { active: speed === spd.value }]"
                @click="setSpeed(spd.value)"
              >
                {{ spd.label }}
              </button>
            </div>
          </div>
        </div>
      </div>

      <div class="section" v-if="currentStep">
        <div class="status-card">
          <div class="status-header">
            <div class="status-badges">
              <span class="badge badge-primary">{{ currentStep.type }}</span>
              <span class="badge badge-secondary">
                Passo {{ stepIndex + 1 }} de {{ steps.length }}
              </span>
            </div>
            <div class="progress-text">{{ Math.round(progress) }}%</div>
          </div>
          <div class="progress-bar-container">
            <div class="progress-bar" :style="{ width: progress + '%' }"></div>
          </div>
        </div>
      </div>

      <div class="section visualization-section">
        <div v-if="!currentArray.length && !loading" class="empty-state">
          <div class="empty-icon">📊</div>
          <p class="empty-text">
            Clique em "Executar" para começar a visualização
          </p>
        </div>

        <div
          v-else-if="showMergeVisualization"
          :class="[
            'merge-visualization',
            currentStep.type === 'merge' ? 'is-merge' : 'is-divide',
          ]"
        >
          <div
            class="merge-section"
            v-if="currentStep.left && currentStep.left.length"
          >
            <h3 class="merge-label">Esquerda</h3>
            <div class="bars-container-small">
              <div class="bars">
                <div
                  v-for="(value, index) in currentStep.left"
                  :key="`left-${index}-${value}`"
                  class="bar-wrapper"
                >
                  <div
                    class="bar bar-left"
                    :style="{ height: getHeight(value, 160) }"
                  >
                    <span class="bar-value">{{ value }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="merge-arrow" v-if="currentStep.type === 'merge'">↓</div>

          <div
            class="merge-section"
            v-if="currentStep.right && currentStep.right.length"
          >
            <h3 class="merge-label">Direita</h3>
            <div class="bars-container-small">
              <div class="bars">
                <div
                  v-for="(value, index) in currentStep.right"
                  :key="`right-${index}-${value}`"
                  class="bar-wrapper"
                >
                  <div
                    class="bar bar-right"
                    :style="{ height: getHeight(value, 160) }"
                  >
                    <span class="bar-value">{{ value }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="merge-result" v-if="currentArray.length">
            <h3 class="merge-label">
              {{ currentStep.type === "merge" ? "Resultado" : "Array" }}
            </h3>
            <div class="bars-container">
              <div class="bars">
                <div
                  v-for="(value, index) in currentArray"
                  :key="`bar-${index}-${value}`"
                  class="bar-wrapper"
                >
                  <div
                    class="bar"
                    :class="barClass(index)"
                    :style="{ height: value * 4 + 'px' }"
                  >
                    <span class="bar-value">{{ value }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div v-else-if="currentArray.length" class="bars-container">
          <div class="bars">
            <div
              v-for="(value, index) in currentArray"
              :key="`bar-${index}-${value}`"
              class="bar-wrapper"
            >
              <div
                class="bar"
                :class="barClass(index)"
                :style="{ height: value * 4 + 'px' }"
              >
                <span class="bar-value">{{ value }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="section" v-if="currentArray.length">
        <div class="legend">
          <div class="legend-item">
            <div class="legend-color legend-compare"></div>
            <span>Comparando</span>
          </div>
          <div class="legend-item">
            <div class="legend-color legend-swap"></div>
            <span>Trocando</span>
          </div>
          <div class="legend-item">
            <div class="legend-color legend-pivot"></div>
            <span>Pivô</span>
          </div>
          <div class="legend-item">
            <div class="legend-color legend-default"></div>
            <span>Padrão</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";
import axios from "axios";

const algorithms = ["bubble", "merge", "quick"];
const selected = ref("bubble");
const loading = ref(false);

const steps = ref([]);
const stepIndex = ref(0);
const playing = ref(false);
const speed = ref(600);
let interval = null;

const speedOptions = [
  { label: "0.5x", value: 1200 },
  { label: "1x", value: 600 },
  { label: "1.5x", value: 400 },
  { label: "2x", value: 200 },
];

const selectAlgorithm = (alg) => {
  selected.value = alg;
  if (steps.value.length > 0) {
    stop();
    steps.value = [];
    stepIndex.value = 0;
  }
};

const getAlgorithmName = (alg) => {
  const names = {
    bubble: "Bubble Sort",
    merge: "Merge Sort",
    quick: "Quick Sort",
  };
  return names[alg] || alg;
};

const getAlgorithmIcon = (alg) => {
  const icons = {
    bubble: "🫧",
    merge: "🔀",
    quick: "⚡",
  };
  return icons[alg] || "📊";
};

const run = async () => {
  stop();
  loading.value = true;

  try {
    const response = await axios.post(
      `http://localhost:8000/api/sort/${selected.value}`,
      {
        array: generateArray(),
      }
    );

    steps.value = response.data.context.steps;
    stepIndex.value = 0;
  } catch (error) {
    console.error("Erro ao executar algoritmo:", error);
    alert(
      "Erro ao executar o algoritmo. Verifique se o servidor está rodando."
    );
  } finally {
    loading.value = false;
  }
};

const generateArray = () => {
  return Array.from({ length: 15 }, () => Math.floor(Math.random() * 50) + 5);
};

const currentStep = computed(() => steps.value[stepIndex.value]);

const currentArray = computed(() => {
  if (!currentStep.value) return [];

  if (currentStep.value.array) return currentStep.value.array;
  if (currentStep.value.result) return currentStep.value.result;
  if (currentStep.value.original) return currentStep.value.original;

  return [];
});

const showMergeVisualization = computed(() => {
  if (!currentStep.value || selected.value !== "merge") return false;
  return (
    (currentStep.value.type === "divide" ||
      currentStep.value.type === "merge") &&
    (currentStep.value.left || currentStep.value.right)
  );
});

const nextStep = () => {
  if (stepIndex.value < steps.value.length - 1) {
    stepIndex.value++;
  }
};

const prevStep = () => {
  if (stepIndex.value > 0) {
    stepIndex.value--;
  }
};

const togglePlay = () => {
  if (playing.value) {
    stop();
  } else {
    play();
  }
};

const play = () => {
  playing.value = true;
  interval = setInterval(() => {
    if (stepIndex.value >= steps.value.length - 1) {
      stop();
      return;
    }
    stepIndex.value++;
  }, speed.value);
};

const setSpeed = (newSpeed) => {
  speed.value = newSpeed;
  if (playing.value) {
    stop();
    play();
  }
};

const stop = () => {
  playing.value = false;
  clearInterval(interval);
};

const progress = computed(() => {
  if (steps.value.length <= 1) return 0;
  return (stepIndex.value / (steps.value.length - 1)) * 100;
});

const barClass = (index) => {
  if (!currentStep.value) return "";

  const comparing = currentStep.value.comparing;
  const swapped = currentStep.value.swapped;

  if (Array.isArray(comparing) && comparing.includes(index)) {
    return "compare";
  }

  if (Array.isArray(swapped) && swapped.includes(index)) {
    return "swap";
  }

  if (
    currentStep.value.pivot !== undefined &&
    Array.isArray(currentStep.value.array) &&
    currentStep.value.array[index] === currentStep.value.pivot
  ) {
    return "pivot";
  }

  return "";
};

const maxValue = computed(() => {
  return Math.max(...currentArray.value, 1);
});

const getHeight = (value, scale = 180) => {
  return (value / maxValue.value) * scale + "px";
};
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.app {
  min-height: 100vh;
  padding: 20px;
  background: linear-gradient(135deg, #0f172a 0%, #1e293b 50%, #0f172a 100%);
  color: #f1f5f9;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Inter",
    sans-serif;
  position: relative;
  overflow-x: hidden;
}

.app::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: radial-gradient(
      circle at 20% 20%,
      rgba(52, 211, 153, 0.1) 0%,
      transparent 50%
    ),
    radial-gradient(
      circle at 80% 80%,
      rgba(168, 85, 247, 0.08) 0%,
      transparent 50%
    );
  pointer-events: none;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  position: relative;
  z-index: 1;
}

.header {
  text-align: center;
  margin-bottom: 48px;
  padding: 20px 0;
}

.title {
  font-size: clamp(2rem, 5vw, 3rem);
  font-weight: 800;
  margin: 0 0 12px 0;
  letter-spacing: -0.02em;
  line-height: 1.2;
}

.gradient-text {
  background: linear-gradient(135deg, #34d399 0%, #10b981 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.subtitle {
  font-size: 1.125rem;
  color: #94a3b8;
  margin: 0;
  font-weight: 400;
}

.section {
  margin-bottom: 32px;
}

.section-title {
  font-size: 0.875rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #64748b;
  margin: 0 0 16px 0;
}

.algorithm-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
  gap: 12px;
}

.algorithm-btn {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  padding: 20px;
  background: rgba(15, 23, 42, 0.6);
  border: 2px solid #1e293b;
  border-radius: 16px;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  backdrop-filter: blur(8px);
  position: relative;
  overflow: hidden;
}

.algorithm-btn::before {
  content: "";
  position: absolute;
  inset: 0;
  background: linear-gradient(
    135deg,
    rgba(52, 211, 153, 0.1),
    rgba(16, 185, 129, 0.1)
  );
  opacity: 0;
  transition: opacity 0.3s;
}

.algorithm-btn:hover::before {
  opacity: 1;
}

.algorithm-btn:hover {
  border-color: #34d399;
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(52, 211, 153, 0.2);
}

.algorithm-btn.active {
  background: linear-gradient(135deg, #34d399 0%, #10b981 100%);
  border-color: #34d399;
  box-shadow: 0 8px 24px rgba(52, 211, 153, 0.3);
}

.algorithm-btn.active .algorithm-name {
  color: #022c22;
  font-weight: 700;
}

.algorithm-icon {
  font-size: 2rem;
  filter: grayscale(0.3);
  transition: filter 0.3s;
}

.algorithm-btn.active .algorithm-icon,
.algorithm-btn:hover .algorithm-icon {
  filter: grayscale(0);
}

.algorithm-name {
  font-size: 0.9375rem;
  font-weight: 600;
  color: #cbd5e1;
  transition: color 0.3s;
}

.control-panel {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  justify-content: center;
  align-items: center;
  padding: 24px;
  background: rgba(15, 23, 42, 0.6);
  border: 1px solid #1e293b;
  border-radius: 20px;
  backdrop-filter: blur(8px);
}

.playback-controls {
  display: flex;
  gap: 8px;
  align-items: center;
}

.btn {
  padding: 12px 24px;
  border-radius: 12px;
  border: none;
  font-size: 0.9375rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  position: relative;
  overflow: hidden;
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  transform: none !important;
}

.btn-primary {
  background: linear-gradient(135deg, #34d399 0%, #10b981 100%);
  color: #022c22;
  box-shadow: 0 4px 12px rgba(52, 211, 153, 0.3);
}

.btn-primary:hover:not(:disabled) {
  box-shadow: 0 6px 20px rgba(52, 211, 153, 0.4);
  transform: translateY(-1px);
}

.btn-primary:active:not(:disabled) {
  transform: translateY(0);
}

.btn-secondary {
  background: rgba(100, 116, 139, 0.2);
  color: #cbd5e1;
  border: 1px solid #334155;
}

.btn-secondary:hover:not(:disabled) {
  background: rgba(100, 116, 139, 0.3);
  border-color: #475569;
}

.btn-icon {
  padding: 12px 16px;
  background: rgba(100, 116, 139, 0.2);
  color: #cbd5e1;
  border: 1px solid #334155;
  min-width: 48px;
}

.btn-icon:hover:not(:disabled) {
  background: rgba(100, 116, 139, 0.3);
  border-color: #475569;
}

.loading-dots::after {
  content: "...";
  animation: dots 1.5s steps(3, end) infinite;
}

@keyframes dots {
  0%,
  20% {
    content: ".";
  }
  40% {
    content: "..";
  }
  60%,
  100% {
    content: "...";
  }
}

.status-card {
  padding: 24px;
  background: rgba(15, 23, 42, 0.6);
  border: 1px solid #1e293b;
  border-radius: 20px;
  backdrop-filter: blur(8px);
}

.status-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  gap: 16px;
  flex-wrap: wrap;
}

.status-badges {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.badge {
  padding: 6px 14px;
  border-radius: 999px;
  font-size: 0.8125rem;
  font-weight: 600;
  letter-spacing: 0.01em;
}

.badge-primary {
  background: linear-gradient(135deg, #34d399 0%, #10b981 100%);
  color: #022c22;
}

.badge-secondary {
  background: rgba(100, 116, 139, 0.3);
  color: #cbd5e1;
  border: 1px solid #334155;
}

.progress-text {
  font-size: 1.25rem;
  font-weight: 700;
  color: #34d399;
  font-variant-numeric: tabular-nums;
}

.progress-bar-container {
  height: 8px;
  background: rgba(30, 41, 59, 0.8);
  border-radius: 999px;
  overflow: hidden;
  position: relative;
}

.progress-bar {
  height: 100%;
  background: linear-gradient(90deg, #34d399 0%, #10b981 100%);
  border-radius: 999px;
  transition: width 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 0 12px rgba(52, 211, 153, 0.5);
}

.visualization-section {
  min-height: 400px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.empty-state {
  text-align: center;
  padding: 60px 20px;
}

.empty-icon {
  font-size: 4rem;
  margin-bottom: 16px;
  opacity: 0.5;
}

.empty-text {
  font-size: 1.125rem;
  color: #64748b;
  margin: 0;
}

.bars-container {
  width: 100%;
  padding: 40px 20px;
  background: rgba(15, 23, 42, 0.4);
  border: 1px solid #1e293b;
  border-radius: 20px;
  backdrop-filter: blur(8px);
}

.bars {
  display: flex;
  align-items: flex-end;
  justify-content: center;
  gap: 8px;
  min-height: 250px;
}

.bar-wrapper {
  flex: 0 1 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.bar {
  width: clamp(24px, 4vw, 40px);
  border-radius: 8px 8px 4px 4px;
  background: linear-gradient(180deg, #34d399 0%, #10b981 100%);
  display: flex;
  align-items: flex-end;
  justify-content: center;
  padding-bottom: 6px;
  box-shadow: 0 4px 12px rgba(52, 211, 153, 0.3);
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
  min-height: 40px;
}

.bar::before {
  content: "";
  position: absolute;
  inset: 0;
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.2), transparent);
  border-radius: 8px 8px 4px 4px;
}

.bar-value {
  font-size: 0.75rem;
  font-weight: 700;
  color: #022c22;
  position: relative;
  z-index: 1;
}

.bar.compare {
  background: linear-gradient(180deg, #ef4444 0%, #dc2626 100%);
  box-shadow: 0 4px 12px rgba(239, 68, 68, 0.4);
  transform: scale(1.05);
}

.bar.compare .bar-value {
  color: #fff;
}

.bar.swap {
  background: linear-gradient(180deg, #facc15 0%, #eab308 100%);
  box-shadow: 0 4px 12px rgba(250, 204, 21, 0.4);
  transform: scale(1.08) rotate(2deg);
  animation: shake 0.3s;
}

.bar.swap .bar-value {
  color: #422006;
}

.bar.pivot {
  background: linear-gradient(180deg, #a855f7 0%, #9333ea 100%);
  box-shadow: 0 4px 12px rgba(168, 85, 247, 0.4);
  transform: scale(1.05);
}

.bar.pivot .bar-value {
  color: #fff;
}

@keyframes shake {
  0%,
  100% {
    transform: scale(1.08) rotate(2deg);
  }
  25% {
    transform: scale(1.08) rotate(-2deg);
  }
  75% {
    transform: scale(1.08) rotate(2deg);
  }
}

.merge-visualization {
  width: 100%;
  display: grid;
  gap: 24px;
  align-items: start;
}

.merge-visualization.is-divide {
  grid-template-columns: 1fr 1fr;
}

.merge-visualization.is-merge {
  grid-template-columns: 1fr auto 1fr;
}

.merge-section {
  width: 100%;
}

.merge-label {
  text-align: center;
  font-size: 0.875rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #64748b;
  margin: 0 0 12px 0;
}

.merge-arrow {
  text-align: center;
  font-size: 2rem;
  color: #34d399;
  animation: bounce 2s infinite;
}

@keyframes bounce {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-10px);
  }
}

.merge-result {
  grid-column: 1 / -1;
  margin-top: 8px;
  padding-top: 16px;
  border-top: 2px solid #1e293b;
}
.bars-container-small {
  width: 100%;
  padding: 24px 20px;
  background: rgba(15, 23, 42, 0.3);
  border: 1px solid #1e293b;
  border-radius: 16px;
  backdrop-filter: blur(8px);
}

.bars-container-small .bars {
  min-height: 180px;
}

.bar-left {
  background: linear-gradient(180deg, #60a5fa 0%, #3b82f6 100%);
  box-shadow: 0 4px 12px rgba(96, 165, 250, 0.3);
}

.bar-left .bar-value {
  color: #fff;
}

.bar-right {
  background: linear-gradient(180deg, #22d3ee 0%, #06b6d4 100%);
  box-shadow: 0 4px 12px rgba(34, 211, 238, 0.3);
}

.bar-right .bar-value {
  color: #042f2e;
}

.speed-control {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  background: rgba(30, 41, 59, 0.4);
  border-radius: 12px;
  border: 1px solid #334155;
}

.speed-label {
  font-size: 0.875rem;
  font-weight: 600;
  color: #94a3b8;
  white-space: nowrap;
}

.speed-buttons {
  display: flex;
  gap: 4px;
}

.speed-btn {
  padding: 6px 12px;
  border-radius: 8px;
  background: rgba(100, 116, 139, 0.2);
  border: 1px solid #334155;
  color: #cbd5e1;
  font-size: 0.8125rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.speed-btn:hover {
  background: rgba(100, 116, 139, 0.3);
  border-color: #475569;
}

.speed-btn.active {
  background: linear-gradient(135deg, #34d399 0%, #10b981 100%);
  color: #022c22;
  border-color: #34d399;
}

.legend {
  display: flex;
  justify-content: center;
  gap: 24px;
  flex-wrap: wrap;
  padding: 20px;
  background: rgba(15, 23, 42, 0.4);
  border: 1px solid #1e293b;
  border-radius: 16px;
  backdrop-filter: blur(8px);
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.875rem;
  color: #cbd5e1;
}

.legend-color {
  width: 24px;
  height: 24px;
  border-radius: 6px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.legend-compare {
  background: linear-gradient(135deg, #ef4444, #dc2626);
}

.legend-swap {
  background: linear-gradient(135deg, #facc15, #eab308);
}

.legend-pivot {
  background: linear-gradient(135deg, #a855f7, #9333ea);
}

.legend-default {
  background: linear-gradient(135deg, #34d399, #10b981);
}

@media (max-width: 768px) {
  .app {
    padding: 16px;
  }

  .title {
    font-size: 2rem;
  }

  .subtitle {
    font-size: 1rem;
  }

  .control-panel {
    flex-direction: column;
    padding: 16px;
  }

  .playback-controls {
    width: 100%;
    justify-content: center;
  }

  .btn {
    flex: 1;
    min-width: 100px;
  }

  .speed-control {
    width: 100%;
    justify-content: center;
  }

  .speed-buttons {
    flex: 1;
  }

  .speed-btn {
    flex: 1;
  }

  .bars {
    gap: 4px;
  }

  .bar {
    width: 20px;
  }

  .merge-visualization {
    gap: 16px;
  }

  .bars-container-small .bars {
    min-height: 150px;
  }

  .legend {
    gap: 12px;
  }
}
</style>
