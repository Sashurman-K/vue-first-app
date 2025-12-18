<template>
  <div class="saved-palettes">
    <h1>📁 Сохранённые палитры</h1>

    <div v-if="!palettes.length" class="empty-state">
      <p>Нет сохранённых палитр</p>
    </div>

    <div v-else class="palettes-grid">
      <div
        v-for="p in palettes"
        :key="p.id"
        class="palette-card"
        @click="selectPalette(p)"
      >
        <div class="palette-colors">
          <div
            v-for="(color, i) in p.colors"
            :key="i"
            :style="{ background: color }"
          />
        </div>
        <div class="palette-info">
          <span>{{ p.name || `Палитра #${p.id}` }}</span>
          <button @click.stop="deletePalette(p.id)">🗑️</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();
const palettes = ref([]);

const loadPalettes = () => {
  const saved = JSON.parse(localStorage.getItem('savedPalettes') || '[]');
  palettes.value = saved.sort((a, b) => b.id - a.id);
};

const deletePalette = (id) => {
  palettes.value = palettes.value.filter(p => p.id !== id);
  localStorage.setItem('savedPalettes', JSON.stringify(palettes.value));
};

const selectPalette = (palette) => {
  router.push({
    path: '/',
    query: { palette: JSON.stringify(palette.colors) }
  });
};

onMounted(loadPalettes);
</script>

<style scoped>
.saved-palettes {
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
}

h1 {
  margin-bottom: 2rem;
  color: #1e293b;
}

.empty-state {
  text-align: center;
  padding: 4rem;
  color: #64748b;
}

.palettes-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 1.5rem;
}

.palette-card {
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 12px rgba(0,0,0,0.08);
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.palette-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 20px rgba(0,0,0,0.12);
}

.palette-colors {
  display: flex;
  height: 120px;
}

.palette-colors div {
  flex: 1;
  transition: flex 0.3s;
}

.palette-colors:hover div {
  flex: 1.2;
}

.palette-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
  border-top: 1px solid #f1f5f9;
}

.palette-info span {
  font-weight: 500;
  color: #334155;
}

.palette-info button {
  background: transparent;
  border: none;
  color: #ef4444;
  font-size: 1.2rem;
  cursor: pointer;
  padding: 0.5rem;
  border-radius: 6px;
  transition: background 0.2s;
}

.palette-info button:hover {
  background: #fee2e2;
}
</style>