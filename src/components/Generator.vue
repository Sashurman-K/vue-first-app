<template>
  <div id="app-container" :class="previewTheme">
    <header>
      <h1>🎨 Генератор палитры</h1>
      <div class="controls">
        <div class="control-group">
          <label>Базовый цвет</label>
          <input type="color" v-model="baseColor">
        </div>

        <div class="control-group">
          <label>Тип палитры</label>
          <div class="btn-group">
            <button @click="generatePalette('analogous')" :class="{ active: paletteType === 'analogous' }">
              Аналогичная
            </button>
            <button @click="generatePalette('monochromatic')" :class="{ active: paletteType === 'monochromatic' }">
              Монохромная
            </button>
            <button @click="generatePalette('triad')" :class="{ active: paletteType === 'triad' }">
              Триада
            </button>
            <button @click="generatePalette('complement')" :class="{ active: paletteType === 'complement' }">
              Комплементарная
            </button>
          </div>
        </div>

        <div class="control-group" :class="{ disabled: disableColorCount }">
          <label>Количество цветов</label>
          <div class="radio-group">
            <label v-for="count in [3,5,7]" :key="count">
              <input type="radio" v-model="colorCount" :value="count" :disabled="disableColorCount">
              {{ count }}
            </label>
          </div>
        </div>

        <div class="control-group">
          <label>Формат цвета</label>
          <div class="radio-group">
            <label>
              <input type="radio" v-model="colorFormat" value="hex"> HEX
            </label>
            <label>
              <input type="radio" v-model="colorFormat" value="rgb"> RGB
            </label>
          </div>
        </div>
      </div>
    </header>

    <main>
      <div class="palette">
        <div
          v-for="(color, index) in palette"
          :key="index"
          class="color-card"
          :style="{ backgroundColor: color }"
          @click="copyToClipboard(color)"
        >
          <div class="color-info">
            <span class="color-value">{{ formattedPalette[index] }}</span>
            <button class="lock-btn" @click.stop="toggleLock(index)" :class="{ locked: lockedColors[index] }">
              {{ lockedColors[index] ? '🔒' : '🔓' }}
            </button>
          </div>
        </div>
      </div>

      <div class="action-buttons">
        <button class="save-btn" @click="savePalette">💾 Сохранить</button>
        <button class="export-btn" @click="showExportModal = true">📤 Экспорт</button>
      </div>

      <div v-if="notification" class="notification">{{ notification }}</div>
    </main>

    <section class="preview-section">
      <h2>👁️ Предпросмотр</h2>
      <div class="preview-controls">
        <div class="radio-group">
          <label>
            <input type="radio" v-model="previewTheme" value="light"> 🌞 Светлая
          </label>
          <label>
            <input type="radio" v-model="previewTheme" value="dark"> 🌙 Тёмная
          </label>
        </div>
      </div>

      <div class="preview-content">
        <div class="preview-header">
          <h3 :style="{ color: palette[0] }">Заголовок страницы</h3>
          <p :style="{ color: palette[1] }">Пример основного текста с использованием цветов палитры</p>
        </div>

        <div class="preview-cards">
          <div class="preview-card" :style="{ backgroundColor: palette[2] }">
            <h4 :style="{ color: palette[3] }">Карточка 1</h4>
            <p :style="{ color: palette[4] || palette[1] }">Содержимое карточки с различными стилями</p>
          </div>
          <div class="preview-card" :style="{ backgroundColor: palette[3] }">
            <h4 :style="{ color: palette[4] || palette[0] }">Карточка 2</h4>
            <p :style="{ color: palette[2] }">Ещё один пример использования цветов</p>
          </div>
        </div>

        <div class="preview-actions">
          <button class="preview-button" :style="{
            backgroundColor: palette[4] || palette[2],
            color: palette[0]
          }">
            Основная кнопка
          </button>
          <button class="preview-button secondary" :style="{
            borderColor: palette[3],
            color: palette[3]
          }">
            Вторичная кнопка
          </button>
        </div>
      </div>
    </section>

    <ContrastChecker :palette="palette" />

    <Export v-if="showExportModal" :palette="palette" @close="showExportModal = false" />
  </div>
</template>

<script setup>
import { ref, onMounted, computed, watch } from 'vue';
import tinycolor from 'tinycolor2';
import { useRoute, useRouter } from 'vue-router';
import Export from './Export.vue';
import ContrastChecker from './ContrastChecker.vue';

const baseColor = ref('#3b82f6');
const palette = ref([]);
const colorCount = ref(5);
const colorFormat = ref('hex');
const lockedColors = ref([]);
const notification = ref('');
const previewTheme = ref('light');
const paletteType = ref('analogous');
const showExportModal = ref(false);

const generatePalette = (type = paletteType.value) => {
  paletteType.value = type;
  const color = tinycolor(baseColor.value);
  let newPalette = [];

  switch (type) {
    case 'analogous':
      newPalette = color.analogous(colorCount.value).map(c => c.toHexString());
      break;
    case 'monochromatic':
      newPalette = color.monochromatic(colorCount.value).map(c => c.toHexString());
      break;
    case 'triad':
      newPalette = color.triad().map(c => c.toHexString());
      colorCount.value = 3;
      break;
    case 'complement':
      newPalette = [color.toHexString(), color.complement().toHexString()];
      colorCount.value = 2;
      break;
  }

  palette.value = newPalette.slice(0, colorCount.value);
};

const disableColorCount = computed(() => ['triad', 'complement'].includes(paletteType.value));
const copyToClipboard = (color) => {
  const value = colorFormat.value === 'rgb' ? tinycolor(color).toRgbString() : color;
  navigator.clipboard.writeText(value);
  notification.value = `Скопировано: ${value}`;
  setTimeout(() => notification.value = '', 2000);
};

const toggleLock = (index) => {
  lockedColors.value[index] = !lockedColors.value[index];
};

const formattedPalette = computed(() => {
  return palette.value.map(color =>
    colorFormat.value === 'rgb' ? tinycolor(color).toRgbString() : color
  );
});

const savePalette = () => {
  const saved = JSON.parse(localStorage.getItem('savedPalettes') || '[]');
  saved.push({ id: Date.now(), colors: palette.value, name: `Палитра ${saved.length + 1}` });
  localStorage.setItem('savedPalettes', JSON.stringify(saved));
  notification.value = '✅ Палитра сохранена!';
  setTimeout(() => notification.value = '', 2000);
};

const route = useRoute();
const router = useRouter();

watch([colorCount, baseColor], () => !disableColorCount.value && generatePalette());
watch(paletteType, generatePalette);

onMounted(() => {
  if (route.query.palette) {
    try {
      palette.value = JSON.parse(route.query.palette);
      router.replace({ query: {} });
    } catch {
      generatePalette();
    }
  } else {
    generatePalette();
  }
});
</script>

<style scoped>
#app-container {
  min-height: 100vh;
  padding: 1.5rem;
  transition: background 0.3s, color 0.3s;
}
#app-container.light { background: #f8fafc; color: #1e293b; }
#app-container.dark { background: #0f172a; color: #f1f5f9; }

header {
  text-align: center;
  margin-bottom: 2.5rem;
}
header h1 {
  font-size: 2rem;
  margin-bottom: 1.5rem;
  color: inherit;
}

.controls {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1.5rem;
  max-width: 1000px;
  margin: 0 auto;
  padding: 1.5rem;
  background: rgba(255,255,255,0.05);
  border-radius: 12px;
}
.control-group {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}
.control-group.disabled { opacity: 0.5; pointer-events: none; }
.control-group label { font-weight: 600; font-size: 0.9rem; }

input[type="color"] {
  width: 100%;
  height: 40px;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  cursor: pointer;
}

.btn-group {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 0.5rem;
}
.btn-group button {
  padding: 0.5rem 0.75rem;
  border: 1px solid #cbd5e1;
  background: white;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
}
.btn-group button.active {
  background: #3b82f6;
  color: white;
  border-color: #3b82f6;
}

.radio-group {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}
.radio-group label {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  cursor: pointer;
}

.palette {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 1rem;
  margin: 2rem 0;
  max-width: 1000px;
  margin-left: auto;
  margin-right: auto;
}

.color-card {
  height: 160px;
  border-radius: 12px;
  display: flex;
  align-items: flex-end;
  padding: 1rem;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}
.color-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 20px rgba(0,0,0,0.15);
}

.color-info {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: rgba(0,0,0,0.7);
  padding: 0.75rem;
  border-radius: 8px;
  color: white;
}

.color-value {
  font-family: 'Monaco', monospace;
  font-size: 0.8rem;
  word-break: break-all;
}

.lock-btn {
  background: transparent;
  border: none;
  color: white;
  cursor: pointer;
  font-size: 1rem;
  padding: 0.25rem;
  border-radius: 4px;
}
.lock-btn.locked { background: rgba(255,255,255,0.2); }

.action-buttons {
  display: flex;
  gap: 1rem;
  justify-content: center;
  margin: 2rem 0;
}
.save-btn, .export-btn {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  transition: transform 0.2s;
}
.save-btn { background: #10b981; color: white; }
.export-btn { background: #8b5cf6; color: white; }
.save-btn:hover, .export-btn:hover { transform: scale(1.05); }

.notification {
  position: fixed;
  bottom: 2rem;
  left: 50%;
  transform: translateX(-50%);
  background: #1e293b;
  color: white;
  padding: 1rem 1.5rem;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.2);
  z-index: 1000;
  animation: slideUp 0.3s ease;
}

.preview-section {
  max-width: 1000px;
  margin: 3rem auto;
  padding: 2rem;
  background: rgba(255,255,255,0.05);
  border-radius: 16px;
}
.preview-section h2 { margin-bottom: 1.5rem; }

.preview-controls {
  margin-bottom: 2rem;
}

.preview-content {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.preview-header h3 {
  font-size: 1.5rem;
  margin-bottom: 0.5rem;
}
.preview-header p {
  font-size: 1rem;
  opacity: 0.8;
}

.preview-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1.5rem;
}

.preview-card {
  padding: 1.5rem;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}
.preview-card h4 { margin-bottom: 0.75rem; }

.preview-actions {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}

.preview-button {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  transition: opacity 0.2s;
}
.preview-button:hover { opacity: 0.9; }
.preview-button.secondary {
  background: transparent;
  border: 2px solid;
}

@keyframes slideUp {
  from { transform: translate(-50%, 100%); opacity: 0; }
  to { transform: translate(-50%, 0); opacity: 1; }
}
</style>