<template>
  <div class="palette-exporter">
    <div class="exporter-header">
      <h2>Экспорт палитр</h2>
      <p>Экспортируйте ваши палитры в различные форматы</p>
    </div>

    <div class="export-options">
      <div class="format-selector">
        <h3>Формат экспорта</h3>
        <div class="format-buttons">
          <button 
            v-for="format in exportFormats" 
            :key="format.id"
            @click="selectedFormat = format.id"
            :class="{ active: selectedFormat === format.id }"
            class="format-btn"
          >
            {{ format.name }}
          </button>
        </div>
      </div>

      <div class="palette-selector">
        <h3>Выберите палитру</h3>
        <select v-model="selectedPaletteId" class="palette-select">
          <option value="">Текущая палитра</option>
          <option 
            v-for="palette in savedPalettes" 
            :key="palette.id"
            :value="palette.id"
          >
            {{ palette.name }}
          </option>
        </select>
      </div>
    </div>

    <div class="export-preview">
      <h3>Предварительный просмотр</h3>
      <pre class="code-preview">{{ exportCode }}</pre>
      
      <div class="export-actions">
        <button @click="copyCode" class="copy-btn">
          Копировать код
        </button>
        <button @click="downloadFile" class="download-btn">
          Скачать файл
        </button>
      </div>
    </div>

    <div class="preview-components">
      <h3>Превью в UI-компонентах</h3>
      <div class="components-grid">
        <div class="component-card">
          <div class="component-header" :style="{ backgroundColor: primaryColor }">
            Header
          </div>
          <div class="component-body">
            <button class="ui-button" :style="{ backgroundColor: accentColor }">
              Primary Button
            </button>
            <button class="ui-button secondary" :style="{ 
              backgroundColor: secondaryColor,
              color: textColor 
            }">
              Secondary Button
            </button>
          </div>
        </div>

        <div class="component-card">
          <div class="component-sidebar" :style="{ backgroundColor: sidebarColor }">
            Sidebar
          </div>
          <div class="component-main">
            <div class="ui-alert" :style="{ 
              backgroundColor: alertColor,
              color: textColor 
            }">
              Alert Message
            </div>
            <input 
              type="text" 
              class="ui-input"
              :style="{ 
                borderColor: borderColor,
                color: textColor 
              }"
              placeholder="Input field"
            >
          </div>
        </div>
      </div>
    </div>

    <div class="share-section">
      <h3>Шаринг палитры</h3>
      <div class="share-url">
        <input 
          :value="shareableUrl"
          readonly
          class="url-input"
          ref="urlInput"
        >
        <button @click="copyUrl" class="copy-url-btn">
          Копировать ссылку
        </button>
      </div>
      <p class="share-note">
        Ссылка будет доступна в течение 30 дней
      </p>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'

export default {
  name: 'PaletteExporter',
  setup() {
    const selectedFormat = ref('css')
    const selectedPaletteId = ref('')
    const savedPalettes = ref([])
    const urlInput = ref(null)

    const exportFormats = [
      { id: 'css', name: 'CSS Variables' },
      { id: 'scss', name: 'SCSS Variables' },
      { id: 'tailwind', name: 'Tailwind Config' },
      { id: 'json', name: 'JSON' },
      { id: 'android', name: 'Android Colors' }
    ]

    // Загрузка палитр
    const loadPalettes = () => {
      const palettes = localStorage.getItem('savedPalettes')
      savedPalettes.value = palettes ? JSON.parse(palettes) : []
    }

    // Получение текущей палитры
    const currentPalette = computed(() => {
      if (selectedPaletteId.value) {
        const palette = savedPalettes.value.find(p => p.id === selectedPaletteId.value)
        return palette ? palette.colors : []
      }
      
      // Текущая палитра из localStorage
      const palette = localStorage.getItem('palette')
      return palette ? JSON.parse(palette) : ['#667eea', '#764ba2', '#4ecdc4', '#ff6b6b', '#ffa502']
    })

    // Генерация кода для экспорта
    const exportCode = computed(() => {
      const colors = currentPalette.value
      const paletteName = selectedPaletteId.value 
        ? savedPalettes.value.find(p => p.id === selectedPaletteId.value)?.name 
        : 'Моя палитра'

      switch (selectedFormat.value) {
        case 'css':
          return generateCssVariables(colors, paletteName)
        case 'scss':
          return generateScssVariables(colors, paletteName)
        case 'tailwind':
          return generateTailwindConfig(colors, paletteName)
        case 'json':
          return generateJson(colors, paletteName)
        case 'android':
          return generateAndroidColors(colors, paletteName)
        default:
          return ''
      }
    })

    // CSS Variables
    const generateCssVariables = (colors, name) => {
      let code = `/* ${name} - CSS Variables */\n:root {\n`
      colors.forEach((color, index) => {
        const varName = `--color-${index + 1}`
        code += `  ${varName}: ${color};\n`
      })
      code += '}\n\n/* Использование */\n.button {\n  background-color: var(--color-1);\n}'
      return code
    }

    // SCSS Variables
    const generateScssVariables = (colors, name) => {
      let code = `// ${name} - SCSS Variables\n`
      colors.forEach((color, index) => {
        const varName = `$color-${index + 1}`
        code += `${varName}: ${color};\n`
      })
      code += '\n// Использование\n.button {\n  background-color: $color-1;\n}'
      return code
    }

    // Tailwind Config
    const generateTailwindConfig = (colors, name) => {
      let code = `// ${name} - Tailwind Config\nmodule.exports = {\n  theme: {\n    extend: {\n      colors: {\n`
      colors.forEach((color, index) => {
        const colorName = `color${index + 1}`
        code += `        '${colorName}': '${color}',\n`
      })
      code += '      }\n    }\n  }\n}'
      return code
    }

    // JSON
    const generateJson = (colors, name) => {
      const data = {
        name,
        colors,
        createdAt: new Date().toISOString(),
        format: 'hex'
      }
      return JSON.stringify(data, null, 2)
    }

    // Android Colors
    const generateAndroidColors = (colors, name) => {
      let code = `<!-- ${name} - Android Colors -->\n<resources>\n`
      colors.forEach((color, index) => {
        const colorName = `color_${index + 1}`
        // Конвертация HEX в ARGB для Android
        const androidColor = color.replace('#', '#FF')
        code += `  <color name="${colorName}">${androidColor}</color>\n`
      })
      code += '</resources>'
      return code
    }

    // Копирование кода
    const copyCode = async () => {
      try {
        await navigator.clipboard.writeText(exportCode.value)
        alert('Код скопирован в буфер обмена!')
      } catch (err) {
        console.error('Ошибка копирования:', err)
      }
    }

    // Скачивание файла
    const downloadFile = () => {
      const blob = new Blob([exportCode.value], { type: 'text/plain' })
      const url = URL.createObjectURL(blob)
      const a = document.createElement('a')
      
      let extension = '.txt'
      switch (selectedFormat.value) {
        case 'css': extension = '.css'; break
        case 'scss': extension = '.scss'; break
        case 'json': extension = '.json'; break
        case 'android': extension = '.xml'; break
        case 'tailwind': extension = '.js'; break
      }
      
      const paletteName = selectedPaletteId.value 
        ? savedPalettes.value.find(p => p.id === selectedPaletteId.value)?.name 
        : 'my-palette'
      
      a.href = url
      a.download = `${paletteName}${extension}`
      document.body.appendChild(a)
      a.click()
      document.body.removeChild(a)
      URL.revokeObjectURL(url)
    }

    // Цвета для превью
    const primaryColor = computed(() => currentPalette.value[0] || '#667eea')
    const accentColor = computed(() => currentPalette.value[1] || '#764ba2')
    const secondaryColor = computed(() => currentPalette.value[2] || '#4ecdc4')
    const sidebarColor = computed(() => currentPalette.value[3] || '#ff6b6b')
    const alertColor = computed(() => currentPalette.value[4] || '#ffa502')
    const textColor = computed(() => '#333333')
    const borderColor = computed(() => '#dddddd')

    // Шаринг
    const shareableUrl = computed(() => {
      const colors = encodeURIComponent(JSON.stringify(currentPalette.value))
      return `${window.location.origin}/share?palette=${colors}`
    })

    const copyUrl = async () => {
      try {
        await navigator.clipboard.writeText(shareableUrl.value)
        alert('Ссылка скопирована в буфер обмена!')
      } catch (err) {
        console.error('Ошибка копирования:', err)
      }
    }

    // Инициализация
    onMounted(() => {
      loadPalettes()
    })

    return {
      selectedFormat,
      selectedPaletteId,
      savedPalettes,
      urlInput,
      exportFormats,
      exportCode,
      primaryColor,
      accentColor,
      secondaryColor,
      sidebarColor,
      alertColor,
      textColor,
      borderColor,
      shareableUrl,
      copyCode,
      downloadFile,
      copyUrl
    }
  }
}
</script>

<style scoped>
.palette-exporter {
  max-width: 1200px;
  margin: 0 auto;
}

.exporter-header {
  text-align: center;
  margin-bottom: 2rem;
}

.export-options {
  display: flex;
  flex-wrap: wrap;
  gap: 2rem;
  margin-bottom: 2rem;
  padding: 1.5rem;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.format-selector,
.palette-selector {
  flex: 1;
  min-width: 300px;
}

.format-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-top: 1rem;
}

.format-btn {
  padding: 0.5rem 1rem;
  background-color: #f8f9fa;
  border: 2px solid #ddd;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.format-btn.active {
  background-color: #667eea;
  color: white;
  border-color: #667eea;
}

.palette-select {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid #ddd;
  border-radius: 5px;
  margin-top: 1rem;
  font-size: 1rem;
}

.export-preview {
  margin-bottom: 2rem;
  padding: 1.5rem;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.code-preview {
  background-color: #f8f9fa;
  padding: 1rem;
  border-radius: 5px;
  overflow-x: auto;
  font-family: 'Courier New', monospace;
  font-size: 0.9rem;
  line-height: 1.5;
  margin: 1rem 0;
  max-height: 300px;
  overflow-y: auto;
}

.export-actions {
  display: flex;
  gap: 1rem;
  margin-top: 1rem;
}

.copy-btn,
.download-btn {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  transition: background-color 0.3s ease;
}

.copy-btn {
  background-color: #007bff;
  color: white;
}

.download-btn {
  background-color: #28a745;
  color: white;
}

.preview-components {
  margin-bottom: 2rem;
  padding: 1.5rem;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.components-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
  gap: 2rem;
  margin-top: 1rem;
}

.component-card {
  border: 2px solid #ddd;
  border-radius: 10px;
  overflow: hidden;
}

.component-header {
  padding: 1.5rem;
  color: white;
  font-weight: bold;
  text-align: center;
}

.component-body {
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.ui-button {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 5px;
  color: white;
  font-weight: bold;
  cursor: pointer;
}

.ui-button.secondary {
  background-color: #6c757d;
}

.component-sidebar {
  width: 30%;
  height: 200px;
  padding: 1rem;
  background-color: #f8f9fa;
  float: left;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
}

.component-main {
  margin-left: 30%;
  padding: 1rem;
}

.ui-alert {
  padding: 1rem;
  border-radius: 5px;
  margin-bottom: 1rem;
  font-weight: bold;
}

.ui-input {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid #ddd;
  border-radius: 5px;
  font-size: 1rem;
}

.share-section {
  padding: 1.5rem;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.share-url {
  display: flex;
  gap: 1rem;
  margin: 1rem 0;
}

.url-input {
  flex: 1;
  padding: 0.75rem;
  border: 2px solid #ddd;
  border-radius: 5px;
  font-size: 1rem;
  background-color: #f8f9fa;
}

.copy-url-btn {
  padding: 0.75rem 1.5rem;
  background-color: #17a2b8;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.share-note {
  color: #666;
  font-size: 0.9rem;
  font-style: italic;
  margin-top: 0.5rem;
}
</style>