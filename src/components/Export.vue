<template>
  <div class="modal-overlay" @click="$emit('close')">
    <div class="modal-content" @click.stop>
      <div class="modal-header">
        <h2>Экспорт палитры</h2>
        <button class="close-btn" @click="$emit('close')">×</button>
      </div>

      <div class="format-tabs">
        <button
          v-for="f in formats"
          :key="f"
          @click="format = f"
          :class="{ active: format === f }"
        >
          {{ f.toUpperCase() }}
        </button>
      </div>

      <div class="code-container">
        <textarea readonly :value="exportedCode" rows="10"></textarea>
        <div class="copy-indicator" v-if="copied">Скопировано!</div>
      </div>

      <div class="modal-actions">
        <button class="copy-btn" @click="copyToClipboard">
          <span>📋</span> Скопировать
        </button>
        <button class="close-btn-secondary" @click="$emit('close')">Закрыть</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const props = defineProps({ palette: Array });
const emit = defineEmits(['close']);

const format = ref('css');
const copied = ref(false);
const formats = ['css', 'scss', 'tailwind'];

const exportedCode = computed(() => {
  const { palette } = props;
  switch (format.value) {
    case 'css': return palette.map((c, i) => `--color-${i + 1}: ${c};`).join('\n');
    case 'scss': return palette.map((c, i) => `$color-${i + 1}: ${c};`).join('\n');
    case 'tailwind':
      return `colors: {\n${palette.map((c, i) => `  'color-${i + 1}': '${c}',`).join('\n')}\n}`;
    default: return '';
  }
});

const copyToClipboard = async () => {
  await navigator.clipboard.writeText(exportedCode.value);
  copied.value = true;
  setTimeout(() => copied.value = false, 1500);
};
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.7);
  display: grid;
  place-items: center;
  padding: 1rem;
}

.modal-content {
  background: #1e1e1e;
  padding: 1.5rem;
  border-radius: 12px;
  width: min(500px, 90vw);
  color: #fff;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
}

.modal-header h2 {
  margin: 0;
  font-size: 1.25rem;
}

.close-btn {
  background: none;
  border: none;
  color: #aaa;
  font-size: 1.5rem;
  cursor: pointer;
  padding: 0;
  width: 32px;
  height: 32px;
  display: grid;
  place-items: center;
  border-radius: 6px;
}

.close-btn:hover {
  background: #333;
  color: #fff;
}

.format-tabs {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1rem;
  background: #2d2d2d;
  padding: 4px;
  border-radius: 8px;
}

.format-tabs button {
  flex: 1;
  padding: 0.75rem;
  border: none;
  background: transparent;
  color: #aaa;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
  font-weight: 500;
}

.format-tabs button.active {
  background: #3a3a3a;
  color: #fff;
}

.code-container {
  position: relative;
  margin-bottom: 1.5rem;
}

textarea {
  width: 100%;
  background: #252525;
  border: 1px solid #3a3a3a;
  border-radius: 8px;
  padding: 1rem;
  color: #e0e0e0;
  font-family: 'Monaco', 'Consolas', monospace;
  font-size: 0.875rem;
  resize: vertical;
  min-height: 120px;
}

.copy-indicator {
  position: absolute;
  top: 0.5rem;
  right: 0.5rem;
  background: #4CAF50;
  color: white;
  padding: 0.25rem 0.75rem;
  border-radius: 4px;
  font-size: 0.75rem;
  animation: fadeOut 1.5s ease-out forwards;
}

@keyframes fadeOut {
  0%, 70% { opacity: 1; }
  100% { opacity: 0; }
}

.modal-actions {
  display: flex;
  gap: 0.75rem;
}

.modal-actions button {
  flex: 1;
  padding: 0.875rem;
  border: none;
  border-radius: 8px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.copy-btn {
  background: #4CAF50;
  color: white;
}

.copy-btn:hover {
  background: #45a049;
}

.close-btn-secondary {
  background: #3a3a3a;
  color: #fff;
}

.close-btn-secondary:hover {
  background: #444;
}
</style>