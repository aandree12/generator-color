<template>
  <div id="app">

    <nav class="navigation">
      <button
        @click="currentView = 'generator'"
        :class="{ active: currentView === 'generator' }"
        class="nav-button"
      >
        Генератор
      </button>
      <button
        @click="currentView = 'library'"
        :class="{ active: currentView === 'library' }"
        class="nav-button"
      >
        Библиотека
      </button>
      <button
        @click="currentView = 'export'"
        :class="{ active: currentView === 'export' }"
        class="nav-button"
      >
        Экспорт
      </button>
      <button
        @click="currentView = 'advanced'"
        :class="{ active: currentView === 'advanced' }"
        class="nav-button"
      >
        Расширенные инструменты
      </button>
    </nav>

    <main class="main-content">
      <PaletteGenerator v-if="currentView === 'generator'" />
      <PaletteLibrary v-else-if="currentView === 'library'" />
      <PaletteExporter v-else-if="currentView === 'export'" />
      <AdvancedTools v-else-if="currentView === 'advanced'" />
    </main>

    <footer class="app-footer">
      <p>Vue 3 • Генератор палитр • Практика 27-28</p>
    </footer>
  </div>
</template>

<script>
import { ref } from 'vue'
import PaletteGenerator from './components/PaletteGenerator.vue'
import PaletteLibrary from './components/PaletteLibrary.vue'
import PaletteExporter from './components/PaletteExporter.vue'
import AdvancedTools from './components/AdvancedTools.vue'

export default {
  name: 'App',
  components: {
    PaletteGenerator,
    PaletteLibrary,
    PaletteExporter,
    AdvancedTools
  },
  setup() {
    const currentView = ref('generator')
    
    return {
      currentView
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
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f5f5f5;
  color: #333;
  line-height: 1.6;
}

#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.app-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 2rem;
  text-align: center;
}

.app-header h1 {
  margin-bottom: 0.5rem;
  font-size: 2.5rem;
}

.app-header p {
  opacity: 0.9;
  font-size: 1.1rem;
}

.navigation {
  display: flex;
  justify-content: center;
  gap: 1rem;
  padding: 1.5rem;
  background-color: white;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  flex-wrap: wrap;
}

.nav-button {
  padding: 0.75rem 1.5rem;
  border: 2px solid #667eea;
  background-color: transparent;
  color: #667eea;
  border-radius: 25px;
  cursor: pointer;
  font-size: 1rem;
  transition: all 0.3s ease;
}

.nav-button:hover {
  background-color: #667eea;
  color: white;
  transform: translateY(-2px);
}

.nav-button.active {
  background-color: #667eea;
  color: white;
}

.main-content {
  flex: 1;
  padding: 2rem;
  max-width: 1400px;
  margin: 0 auto;
  width: 100%;
}

.app-footer {
  background-color: #333;
  color: white;
  text-align: center;
  padding: 1rem;
  margin-top: auto;
}
</style>