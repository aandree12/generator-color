<template>
  <div class="palette-generator">
    <div class="generator-header">
      <h2>Генератор палитр</h2>
      <div class="controls">
        <button @click="generateRandomPalette" class="generate-btn">
          Случайная палитра
        </button>
        
        <div class="control-group">
          <label>Количество цветов:</label>
          <select v-model="colorCount" @change="updatePaletteCount">
            <option value="3">3</option>
            <option value="5">5</option>
            <option value="7">7</option>
          </select>
        </div>
        
        <div class="control-group">
          <label>Формат:</label>
          <div class="format-toggle">
            <button 
              @click="displayFormat = 'hex'"
              :class="{ active: displayFormat === 'hex' }"
            >
              HEX
            </button>
            <button 
              @click="displayFormat = 'rgb'"
              :class="{ active: displayFormat === 'rgb' }"
            >
              RGB
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Отображение палитры -->
    <div class="palette-display">
      <div 
        v-for="(color, index) in currentPalette" 
        :key="index"
        class="color-card"
        :style="{ backgroundColor: color }"
        @click="copyToClipboard(color)"
        :class="{ locked: lockedColors[index] }"
      >
        <div class="color-info" :class="{ dark: isDarkColor(color) }">
          <span class="color-value">
            {{ displayFormat === 'hex' ? color : rgbColor(color) }}
          </span>
          <button 
            class="lock-btn"
            @click.stop="toggleLock(index)"
            :title="lockedColors[index] ? 'Разблокировать' : 'Заблокировать'"
          >
            {{ lockedColors[index] ? '🔒' : '🔓' }}
          </button>
        </div>
      </div>
    </div>

    <!-- Уведомление о копировании -->
    <div v-if="showNotification" class="notification">
      Цвет скопирован в буфер обмена!
    </div>

    <!-- Просмотрщик -->
    <div class="preview-section">
      <h3>Превью палитры</h3>
      <div class="preview-controls">
        <button @click="toggleBackground" class="bg-toggle">
          {{ darkBackground ? 'Светлый фон' : 'Темный фон' }}
        </button>
      </div>
      
      <div class="mockup" :class="{ dark: darkBackground }">
        <div class="mockup-header" :style="{ backgroundColor: currentPalette[0] || '#667eea' }">
          <h4>Шапка сайта</h4>
          <button class="mockup-btn" :style="{ backgroundColor: currentPalette[1] || '#764ba2' }">
            Кнопка
          </button>
        </div>
        
        <div class="mockup-content">
          <div class="mockup-card" :style="{ backgroundColor: currentPalette[2] || '#ffffff' }">
            <h5>Карточка</h5>
            <p>Пример текста на карточке</p>
            <button class="mockup-btn secondary" :style="{ backgroundColor: currentPalette[3] || '#4ecdc4' }">
              Подробнее
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Сохранение палитры -->
    <div class="save-section">
      <input 
        v-model="paletteName"
        placeholder="Название палитры"
        class="palette-input"
      />
      <button @click="savePalette" class="save-btn">
        Сохранить палитру
      </button>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'

export default {
  name: 'PaletteGenerator',
  setup() {
    // Реактивные переменные
    const colorCount = ref(5)
    const displayFormat = ref('hex')
    const currentPalette = ref([])
    const lockedColors = ref([])
    const showNotification = ref(false)
    const darkBackground = ref(false)
    const paletteName = ref('')

    // Генерация случайного цвета в HEX
    const generateRandomColor = () => {
      return '#' + Math.floor(Math.random() * 16777215).toString(16).padStart(6, '0')
    }

    // Генерация новой палитры
    const generateRandomPalette = () => {
      const newPalette = []
      for (let i = 0; i < colorCount.value; i++) {
        // Если цвет заблокирован, сохраняем его
        if (lockedColors.value[i]) {
          newPalette.push(currentPalette.value[i])
        } else {
          newPalette.push(generateRandomColor())
        }
      }
      currentPalette.value = newPalette
      saveToLocalStorage()
    }

    // Обновление количества цветов
    const updatePaletteCount = () => {
      if (currentPalette.value.length > colorCount.value) {
        currentPalette.value = currentPalette.value.slice(0, colorCount.value)
        lockedColors.value = lockedColors.value.slice(0, colorCount.value)
      } else {
        const needed = colorCount.value - currentPalette.value.length
        for (let i = 0; i < needed; i++) {
          currentPalette.value.push(generateRandomColor())
          lockedColors.value.push(false)
        }
      }
      saveToLocalStorage()
    }

    // Конвертация HEX в RGB
    const rgbColor = (hex) => {
      const r = parseInt(hex.slice(1, 3), 16)
      const g = parseInt(hex.slice(3, 5), 16)
      const b = parseInt(hex.slice(5, 7), 16)
      return `rgb(${r}, ${g}, ${b})`
    }

    // Проверка темного цвета
    const isDarkColor = (hex) => {
      const r = parseInt(hex.slice(1, 3), 16)
      const g = parseInt(hex.slice(3, 5), 16)
      const b = parseInt(hex.slice(5, 7), 16)
      const brightness = (r * 299 + g * 587 + b * 114) / 1000
      return brightness < 128
    }

    const copyToClipboard = async (color) => {
      const text = displayFormat.value === 'hex' ? color : rgbColor(color)
      try {
        await navigator.clipboard.writeText(text)
        showNotification.value = true
        setTimeout(() => {
          showNotification.value = false
        }, 2000)
      } catch (err) {
        console.error('Не удалось скопировать: ', err)
      }
    }

    const toggleLock = (index) => {
      lockedColors.value[index] = !lockedColors.value[index]
      saveToLocalStorage()
    }
    const toggleBackground = () => {
      darkBackground.value = !darkBackground.value
    }

    const saveToLocalStorage = () => {
      localStorage.setItem('palette', JSON.stringify(currentPalette.value))
      localStorage.setItem('lockedColors', JSON.stringify(lockedColors.value))
    }

    const loadFromLocalStorage = () => {
      const savedPalette = localStorage.getItem('palette')
      const savedLocks = localStorage.getItem('lockedColors')
      
      if (savedPalette) {
        currentPalette.value = JSON.parse(savedPalette)
        colorCount.value = currentPalette.value.length
      }
      
      if (savedLocks) {
        lockedColors.value = JSON.parse(savedLocks)
      } else {
        lockedColors.value = new Array(colorCount.value).fill(false)
      }
    }

    // Сохранение палитры в библиотеку
    const savePalette = () => {
      if (!paletteName.value.trim()) {
        alert('Введите название палитры')
        return
      }

      const savedPalettes = JSON.parse(localStorage.getItem('savedPalettes') || '[]')
      const newPalette = {
        id: Date.now(),
        name: paletteName.value,
        colors: [...currentPalette.value],
        createdAt: new Date().toISOString()
      }
      
      savedPalettes.push(newPalette)
      localStorage.setItem('savedPalettes', JSON.stringify(savedPalettes))
      
      paletteName.value = ''
      alert('Палитра сохранена!')
    }

    // Инициализация
    onMounted(() => {
      loadFromLocalStorage()
      if (currentPalette.value.length === 0) {
        generateRandomPalette()
      }
    })

    return {
      colorCount,
      displayFormat,
      currentPalette,
      lockedColors,
      showNotification,
      darkBackground,
      paletteName,
      generateRandomPalette,
      updatePaletteCount,
      rgbColor,
      isDarkColor,
      copyToClipboard,
      toggleLock,
      toggleBackground,
      savePalette
    }
  }
}
</script>

<style scoped>
.palette-generator {
  max-width: 1200px;
  margin: 0 auto;
}

.generator-header {
  margin-bottom: 2rem;
  padding: 1.5rem;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 1.5rem;
  align-items: center;
  margin-top: 1rem;
}

.generate-btn {
  padding: 0.75rem 1.5rem;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  font-size: 1rem;
  font-weight: bold;
  transition: transform 0.3s ease;
}

.generate-btn:hover {
  transform: translateY(-2px);
}

.control-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.control-group label {
  font-weight: 600;
  color: #555;
}

.control-group select {
  padding: 0.5rem;
  border: 2px solid #ddd;
  border-radius: 5px;
  font-size: 1rem;
}

.format-toggle {
  display: flex;
  gap: 0.5rem;
}

.format-toggle button {
  padding: 0.5rem 1rem;
  border: 2px solid #667eea;
  background-color: transparent;
  color: #667eea;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.format-toggle button.active {
  background-color: #667eea;
  color: white;
}

.palette-display {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
  margin-bottom: 2rem;
}

.color-card {
  height: 150px;
  border-radius: 10px;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
  border: 3px solid transparent;
}

.color-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 20px rgba(0,0,0,0.2);
}

.color-card.locked {
  border-color: #667eea;
}

.color-info {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 1rem;
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.color-info.dark {
  background-color: rgba(255, 255, 255, 0.9);
  color: #333;
}

.color-value {
  font-family: monospace;
  font-weight: bold;
  font-size: 0.9rem;
  user-select: all;
}

.lock-btn {
  background: none;
  border: none;
  color: inherit;
  cursor: pointer;
  font-size: 1.2rem;
  padding: 0.25rem;
}

.notification {
  position: fixed;
  top: 20px;
  right: 20px;
  padding: 1rem 1.5rem;
  background-color: #28a745;
  color: white;
  border-radius: 5px;
  animation: slideIn 0.3s ease;
  z-index: 1000;
}

@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

.preview-section {
  margin: 2rem 0;
  padding: 1.5rem;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.preview-controls {
  margin: 1rem 0;
}

.bg-toggle {
  padding: 0.5rem 1rem;
  background-color: #6c757d;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.mockup {
  border: 2px solid #ddd;
  border-radius: 10px;
  overflow: hidden;
  margin-top: 1rem;
}

.mockup.dark {
  background-color: #333;
  color: white;
}

.mockup-header {
  padding: 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.mockup-btn {
  padding: 0.5rem 1.5rem;
  border: none;
  border-radius: 5px;
  color: white;
  font-weight: bold;
  cursor: pointer;
}

.mockup-btn.secondary {
  color: #333;
}

.mockup-content {
  padding: 2rem;
}

.mockup-card {
  padding: 1.5rem;
  border-radius: 10px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.save-section {
  display: flex;
  gap: 1rem;
  margin-top: 2rem;
  padding: 1.5rem;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.palette-input {
  flex: 1;
  padding: 0.75rem;
  border: 2px solid #ddd;
  border-radius: 5px;
  font-size: 1rem;
}

.save-btn {
  padding: 0.75rem 1.5rem;
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  transition: background-color 0.3s ease;
}

.save-btn:hover {
  background-color: #218838;
}
</style>