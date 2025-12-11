<template>
  <div class="advanced-tools">
    <div class="tools-header">
      <h2>Расширенные инструменты</h2>
      <p>Продвинутая генерация и анализ цветов</p>
    </div>

    <div class="tools-grid">
      <!-- Генерация на основе базового цвета -->
      <div class="tool-section">
        <h3>Генерация на основе цвета</h3>
        <div class="base-color-picker">
          <input 
            v-model="baseColor"
            type="color"
            class="color-input"
          />
          <span class="color-value">{{ baseColor }}</span>
        </div>
        
        <div class="harmony-types">
          <button 
            v-for="type in harmonyTypes" 
            :key="type.id"
            @click="generateHarmony(type.id)"
            :class="{ active: selectedHarmony === type.id }"
            class="harmony-btn"
          >
            {{ type.name }}
          </button>
        </div>
        
        <div class="generated-harmony">
          <div 
            v-for="(color, index) in harmonyColors" 
            :key="index"
            class="harmony-color"
            :style="{ backgroundColor: color }"
            @click="copyColor(color)"
          >
            <span class="harmony-color-value">{{ color }}</span>
          </div>
        </div>
      </div>

      <!-- Анализ контрастности -->
      <div class="tool-section">
        <h3>Анализ контрастности (WCAG)</h3>
        <div class="contrast-checker">
          <div class="color-pair">
            <input v-model="foregroundColor" type="color" class="color-input">
            <input v-model="backgroundColor" type="color" class="color-input">
          </div>
          
          <div class="contrast-results">
            <div 
              class="contrast-sample"
              :style="{ 
                backgroundColor: backgroundColor, 
                color: foregroundColor 
              }"
            >
              Пример текста
            </div>
            
            <div class="contrast-info">
              <div class="contrast-ratio">
                Контрастность: {{ contrastRatio }}:1
              </div>
              
              <div class="wcag-levels">
                <div 
                  class="wcag-level"
                  :class="{ passed: passesWCAGAA }"
                >
                  WCAG AA: {{ passesWCAGAA ? '✅' : '❌' }}
                </div>
                <div 
                  class="wcag-level"
                  :class="{ passed: passesWCAGAAA }"
                >
                  WCAG AAA: {{ passesWCAGAAA ? '✅' : '❌' }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Генерация по настроению -->
      <div class="tool-section">
        <h3>Генерация по настроению</h3>
        <div class="mood-selector">
          <button 
            v-for="mood in moods" 
            :key="mood.id"
            @click="generateMoodPalette(mood.id)"
            class="mood-btn"
            :style="{ backgroundColor: mood.color }"
          >
            {{ mood.name }}
          </button>
        </div>
        
        <div class="mood-palette">
          <div 
            v-for="(color, index) in moodPalette" 
            :key="index"
            class="mood-color"
            :style="{ backgroundColor: color }"
          ></div>
        </div>
      </div>

      <!-- Цветовой круг -->
      <div class="tool-section">
        <h3>Цветовой круг</h3>
        <div class="color-wheel-container">
          <canvas ref="colorWheelCanvas" class="color-wheel"></canvas>
          <div class="wheel-controls">
            <input 
              v-model="wheelHue"
              type="range" 
              min="0" 
              max="360" 
              class="hue-slider"
              @input="updateColorWheel"
            >
            <span>Оттенок: {{ wheelHue }}°</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'

export default {
  name: 'AdvancedTools',
  setup() {
    // Базовый цвет для генерации
    const baseColor = ref('#667eea')
    const selectedHarmony = ref('analogous')
    const harmonyColors = ref([])

    // Контрастность
    const foregroundColor = ref('#000000')
    const backgroundColor = ref('#ffffff')

    // Настроение
    const moodPalette = ref([])

    // Цветовой круг
    const colorWheelCanvas = ref(null)
    const wheelHue = ref(0)

    // Типы гармонии
    const harmonyTypes = [
      { id: 'analogous', name: 'Аналогичная' },
      { id: 'monochromatic', name: 'Монохромная' },
      { id: 'triadic', name: 'Триада' },
      { id: 'complementary', name: 'Комплементарная' },
      { id: 'splitComplementary', name: 'Разделенная комплементарная' }
    ]

    // Настроения
    const moods = [
      { id: 'calm', name: 'Спокойные', color: '#4ecdc4' },
      { id: 'energetic', name: 'Энергичные', color: '#ff6b6b' },
      { id: 'professional', name: 'Профессиональные', color: '#667eea' },
      { id: 'warm', name: 'Теплые', color: '#ffa502' },
      { id: 'cool', name: 'Холодные', color: '#3742fa' }
    ]

    // Генерация гармоничной палитры
    const generateHarmony = (type) => {
      selectedHarmony.value = type
      const base = hexToRgb(baseColor.value)
      
      switch (type) {
        case 'analogous':
          harmonyColors.value = generateAnalogous(base)
          break
        case 'monochromatic':
          harmonyColors.value = generateMonochromatic(base)
          break
        case 'triadic':
          harmonyColors.value = generateTriadic(base)
          break
        case 'complementary':
          harmonyColors.value = generateComplementary(base)
          break
        case 'splitComplementary':
          harmonyColors.value = generateSplitComplementary(base)
          break
      }
    }

    // Преобразование HEX в RGB
    const hexToRgb = (hex) => {
      const r = parseInt(hex.slice(1, 3), 16)
      const g = parseInt(hex.slice(3, 5), 16)
      const b = parseInt(hex.slice(5, 7), 16)
      return { r, g, b }
    }

    // Преобразование RGB в HEX
    const rgbToHex = (r, g, b) => {
      return '#' + [r, g, b].map(x => {
        const hex = x.toString(16)
        return hex.length === 1 ? '0' + hex : hex
      }).join('')
    }

    // Генерация аналогичной палитры
    const generateAnalogous = (baseRgb) => {
      const colors = []
      const hsv = rgbToHsv(baseRgb.r, baseRgb.g, baseRgb.b)
      
      for (let i = -2; i <= 2; i++) {
        const newHue = (hsv.h + i * 30 + 360) % 360
        const rgb = hsvToRgb(newHue, hsv.s, hsv.v)
        colors.push(rgbToHex(rgb.r, rgb.g, rgb.b))
      }
      
      return colors
    }

    // Генерация монохромной палитры
    const generateMonochromatic = (baseRgb) => {
      const colors = []
      const hsv = rgbToHsv(baseRgb.r, baseRgb.g, baseRgb.b)
      
      for (let i = 0; i < 5; i++) {
        const value = Math.max(0.2, Math.min(0.9, 0.2 + i * 0.15))
        const rgb = hsvToRgb(hsv.h, hsv.s, value)
        colors.push(rgbToHex(rgb.r, rgb.g, rgb.b))
      }
      
      return colors
    }

    // Генерация триады
    const generateTriadic = (baseRgb) => {
      const colors = []
      const hsv = rgbToHsv(baseRgb.r, baseRgb.g, baseRgb.b)
      
      for (let i = 0; i < 3; i++) {
        const newHue = (hsv.h + i * 120) % 360
        const rgb = hsvToRgb(newHue, hsv.s, hsv.v)
        colors.push(rgbToHex(rgb.r, rgb.g, rgb.b))
      }
      
      return colors
    }

    // Генерация комплементарной палитры
    const generateComplementary = (baseRgb) => {
      const colors = []
      const hsv = rgbToHsv(baseRgb.r, baseRgb.g, baseRgb.b)
      
      // Основной цвет
      colors.push(rgbToHex(baseRgb.r, baseRgb.g, baseRgb.b))
      
      // Комплементарный цвет
      const compHue = (hsv.h + 180) % 360
      const compRgb = hsvToRgb(compHue, hsv.s, hsv.v)
      colors.push(rgbToHex(compRgb.r, compRgb.g, compRgb.b))
      
      return colors
    }

    // Генерация разделенной комплементарной палитры
    const generateSplitComplementary = (baseRgb) => {
      const colors = []
      const hsv = rgbToHsv(baseRgb.r, baseRgb.g, baseRgb.b)
      
      // Основной цвет
      colors.push(rgbToHex(baseRgb.r, baseRgb.g, baseRgb.b))
      
      // Два соседних к комплементарному
      const compHue = (hsv.h + 180) % 360
      for (let i = -1; i <= 1; i += 2) {
        const newHue = (compHue + i * 30 + 360) % 360
        const rgb = hsvToRgb(newHue, hsv.s, hsv.v)
        colors.push(rgbToHex(rgb.r, rgb.g, rgb.b))
      }
      
      return colors
    }

    // Преобразование RGB в HSV
    const rgbToHsv = (r, g, b) => {
      r /= 255
      g /= 255
      b /= 255
      
      const max = Math.max(r, g, b)
      const min = Math.min(r, g, b)
      const delta = max - min
      
      let h = 0
      if (delta !== 0) {
        if (max === r) h = ((g - b) / delta) % 6
        else if (max === g) h = (b - r) / delta + 2
        else h = (r - g) / delta + 4
        
        h = Math.round(h * 60)
        if (h < 0) h += 360
      }
      
      const s = max === 0 ? 0 : delta / max
      const v = max
      
      return { h, s, v }
    }

    // Преобразование HSV в RGB
    const hsvToRgb = (h, s, v) => {
      const c = v * s
      const x = c * (1 - Math.abs((h / 60) % 2 - 1))
      const m = v - c
      
      let r, g, b
      
      if (h < 60) [r, g, b] = [c, x, 0]
      else if (h < 120) [r, g, b] = [x, c, 0]
      else if (h < 180) [r, g, b] = [0, c, x]
      else if (h < 240) [r, g, b] = [0, x, c]
      else if (h < 300) [r, g, b] = [x, 0, c]
      else [r, g, b] = [c, 0, x]
      
      return {
        r: Math.round((r + m) * 255),
        g: Math.round((g + m) * 255),
        b: Math.round((b + m) * 255)
      }
    }

    // Расчет контрастности
    const relativeLuminance = (color) => {
      const rgb = hexToRgb(color)
      const [r, g, b] = [rgb.r, rgb.g, rgb.b].map(c => {
        c /= 255
        return c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4)
      })
      return 0.2126 * r + 0.7152 * g + 0.0722 * b
    }

    const contrastRatio = computed(() => {
      const l1 = relativeLuminance(foregroundColor.value)
      const l2 = relativeLuminance(backgroundColor.value)
      const lighter = Math.max(l1, l2)
      const darker = Math.min(l1, l2)
      const ratio = (lighter + 0.05) / (darker + 0.05)
      return ratio.toFixed(2)
    })

    const passesWCAGAA = computed(() => {
      return parseFloat(contrastRatio.value) >= 4.5
    })

    const passesWCAGAAA = computed(() => {
      return parseFloat(contrastRatio.value) >= 7
    })

    // Генерация палитры по настроению
    const generateMoodPalette = (moodId) => {
      const moodColors = {
        calm: ['#4ecdc4', '#556270', '#c7f464', '#ff6b6b', '#c44d58'],
        energetic: ['#ff6b6b', '#ffa502', '#ffd32a', '#ff9ff3', '#f368e0'],
        professional: ['#667eea', '#764ba2', '#5f72be', '#9d50bb', '#6a11cb'],
        warm: ['#ffa502', '#ff9ff3', '#feca57', '#ff9f43', '#ee5a24'],
        cool: ['#3742fa', '#7158e2', '#cd84f1', '#7d5fff', '#32ff7e']
      }
      moodPalette.value = moodColors[moodId] || moodColors.calm
    }

    // Рисование цветового круга
    const drawColorWheel = () => {
      const canvas = colorWheelCanvas.value
      if (!canvas) return
      
      const ctx = canvas.getContext('2d')
      const centerX = canvas.width / 2
      const centerY = canvas.height / 2
      const radius = Math.min(centerX, centerY) - 10
      
      ctx.clearRect(0, 0, canvas.width, canvas.height)
      
      // Рисуем цветовой круг
      for (let angle = 0; angle < 360; angle += 1) {
        const startAngle = (angle - 0.5) * Math.PI / 180
        const endAngle = (angle + 0.5) * Math.PI / 180
        
        const hue = (angle + wheelHue.value) % 360
        const rgb = hsvToRgb(hue, 1, 1)
        const color = `rgb(${rgb.r}, ${rgb.g}, ${rgb.b})`
        
        ctx.beginPath()
        ctx.moveTo(centerX, centerY)
        ctx.arc(centerX, centerY, radius, startAngle, endAngle)
        ctx.closePath()
        ctx.fillStyle = color
        ctx.fill()
      }
      
      // Рисуем центральный круг
      ctx.beginPath()
      ctx.arc(centerX, centerY, radius * 0.3, 0, Math.PI * 2)
      ctx.fillStyle = '#ffffff'
      ctx.fill()
    }

    const updateColorWheel = () => {
      drawColorWheel()
    }

    // Копирование цвета
    const copyColor = async (color) => {
      try {
        await navigator.clipboard.writeText(color)
        alert('Цвет скопирован!')
      } catch (err) {
        console.error('Ошибка копирования:', err)
      }
    }

    // Инициализация
    onMounted(() => {
      generateHarmony('analogous')
      generateMoodPalette('calm')
      
      // Инициализация canvas
      const canvas = colorWheelCanvas.value
      if (canvas) {
        canvas.width = 300
        canvas.height = 300
        drawColorWheel()
      }
    })

    return {
      baseColor,
      selectedHarmony,
      harmonyColors,
      foregroundColor,
      backgroundColor,
      moodPalette,
      colorWheelCanvas,
      wheelHue,
      harmonyTypes,
      moods,
      contrastRatio,
      passesWCAGAA,
      passesWCAGAAA,
      generateHarmony,
      generateMoodPalette,
      updateColorWheel,
      copyColor
    }
  }
}
</script>

<style scoped>
.advanced-tools {
  max-width: 1200px;
  margin: 0 auto;
}

.tools-header {
  margin-bottom: 2rem;
  text-align: center;
}

.tools-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(500px, 1fr));
  gap: 2rem;
}

.tool-section {
  background-color: white;
  padding: 1.5rem;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.tool-section h3 {
  margin-bottom: 1rem;
  color: #333;
}

.base-color-picker {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1rem;
}

.color-input {
  width: 50px;
  height: 50px;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}

.color-value {
  font-family: monospace;
  font-weight: bold;
}

.harmony-types {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.harmony-btn {
  padding: 0.5rem 1rem;
  background-color: #f8f9fa;
  border: 2px solid #ddd;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.harmony-btn.active {
  background-color: #667eea;
  color: white;
  border-color: #667eea;
}

.generated-harmony {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(80px, 1fr));
  gap: 0.5rem;
  margin-top: 1rem;
}

.harmony-color {
  height: 80px;
  border-radius: 5px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}

.harmony-color-value {
  background-color: rgba(255, 255, 255, 0.9);
  padding: 0.25rem 0.5rem;
  border-radius: 3px;
  font-size: 0.8rem;
  font-family: monospace;
  font-weight: bold;
}

.contrast-checker {
  margin-top: 1rem;
}

.color-pair {
  display: flex;
  gap: 1rem;
  margin-bottom: 1rem;
}

.contrast-results {
  margin-top: 1rem;
}

.contrast-sample {
  padding: 1.5rem;
  border-radius: 5px;
  font-size: 1.2rem;
  font-weight: bold;
  text-align: center;
  margin-bottom: 1rem;
  border: 2px solid #ddd;
}

.contrast-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.contrast-ratio {
  font-size: 1.1rem;
  font-weight: bold;
  color: #333;
}

.wcag-levels {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.wcag-level {
  padding: 0.5rem 1rem;
  background-color: #ffebee;
  border-radius: 5px;
  font-weight: bold;
  text-align: center;
}

.wcag-level.passed {
  background-color: #e8f5e9;
}

.mood-selector {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin: 1rem 0;
}

.mood-btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  color: white;
  font-weight: bold;
  transition: transform 0.3s ease;
}

.mood-btn:hover {
  transform: translateY(-2px);
}

.mood-palette {
  display: flex;
  height: 60px;
  border-radius: 5px;
  overflow: hidden;
  margin-top: 1rem;
}

.mood-color {
  flex: 1;
  cursor: pointer;
  transition: transform 0.3s ease;
}

.mood-color:hover {
  transform: scale(1.1);
}

.color-wheel-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 1rem;
}

.color-wheel {
  width: 300px;
  height: 300px;
  border-radius: 50%;
  margin-bottom: 1rem;
}

.wheel-controls {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.hue-slider {
  width: 200px;
}
</style>