<template>
  <div class="contrast-checker">
    <h3>Проверка контрастности WCAG</h3>

    <div class="contrast-grid">
      <div class="color-row" v-for="(c1, i) in palette" :key="i">
        <div class="color-cell header" :style="{ background: c1 }">
          <span :style="{ color: findBestAccent(c1) }">Color {{ i+1 }}</span>
        </div>
        <div
          v-for="(c2, j) in palette"
          :key="j"
          class="color-cell"
          :style="{ background: c1, color: c2 }"
          v-if="i !== j"
        >
          <div class="contrast-info">
            <strong>{{ tinycolor.readability(c1, c2).toFixed(1) }}</strong>
            <small>{{ getLevel(tinycolor.readability(c1, c2)) }}</small>
          </div>
        </div>
      </div>
    </div>

    <div class="accent-section">
      <h3>Рекомендуемые акцентные цвета</h3>
      <div class="accent-grid">
        <div
          v-for="(color, i) in palette"
          :key="i"
          class="accent-card"
          :style="{ background: color }"
        >
          <span :style="{ color: findBestAccent(color) }">
            Акцентный текст
          </span>
          <small :style="{ color: findBestAccent(color), opacity: 0.8 }">
            Контраст: {{ getBestContrast(color).toFixed(1) }}
          </small>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { defineProps } from 'vue';
import tinycolor from 'tinycolor2';

const props = defineProps({ palette: Array });

const getLevel = (ratio) => {
  if (ratio >= 7) return 'AAA';
  if (ratio >= 4.5) return 'AA';
  if (ratio >= 3) return 'AA Large';
  return 'Fail';
};

const findBestAccent = (background) => {
  let bestColor = props.palette[0];
  let maxRatio = 0;

  props.palette.forEach(color => {
    if (color === background) return;
    const ratio = tinycolor.readability(background, color);
    if (ratio > maxRatio) {
      maxRatio = ratio;
      bestColor = color;
    }
  });

  return bestColor;
};

const getBestContrast = (background) => {
  return tinycolor.readability(background, findBestAccent(background));
};
</script>

<style scoped>
.contrast-checker {
  margin-top: 2rem;
  padding: 1.5rem;
  background: #f8f9fa;
  border-radius: 12px;
}

h3 {
  margin: 0 0 1.5rem 0;
  color: #1a1a1a;
}

.contrast-grid {
  display: flex;
  flex-direction: column;
  gap: 1px;
  background: #e0e0e0;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  overflow: hidden;
}

.color-row {
  display: flex;
}

.color-cell {
  flex: 1;
  min-height: 70px;
  display: grid;
  place-items: center;
  padding: 0.75rem;
  font-size: 0.85rem;
}

.color-cell.header {
  flex: 0.7;
  font-weight: 600;
  border-right: 1px solid rgba(0,0,0,0.1);
}

.contrast-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.25rem;
}

.contrast-info strong {
  font-size: 1rem;
}

.contrast-info small {
  font-size: 0.75rem;
  opacity: 0.9;
}

.accent-section {
  margin-top: 2rem;
}

.accent-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
  gap: 1rem;
  margin-top: 1rem;
}

.accent-card {
  padding: 1.5rem 1rem;
  border-radius: 10px;
  text-align: center;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  transition: transform 0.2s;
}

.accent-card:hover {
  transform: translateY(-2px);
}

.accent-card span {
  font-weight: 600;
  font-size: 0.9rem;
}

.accent-card small {
  font-size: 0.8rem;
}
</style>