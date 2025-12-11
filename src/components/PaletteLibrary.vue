<template>
  <div class="palette-library">
    <div class="library-header">
      <h2>Библиотека палитр</h2>
      <div class="search-controls">
        <input 
          v-model="searchQuery"
          placeholder="Поиск по названию..."
          class="search-input"
        />
        <select v-model="sortBy" class="sort-select">
          <option value="date">По дате</option>
          <option value="name">По названию</option>
          <option value="colors">По количеству цветов</option>
        </select>
      </div>
    </div>

    <div v-if="filteredPalettes.length === 0" class="empty-state">
      <p>Нет сохраненных палитр. Создайте свою первую палитру в разделе "Генератор"!</p>
    </div>

    <div v-else class="palettes-grid">
      <div 
        v-for="palette in sortedPalettes" 
        :key="palette.id"
        class="palette-card"
      >
        <div class="palette-colors">
          <div 
            v-for="(color, index) in palette.colors" 
            :key="index"
            class="library-color"
            :style="{ backgroundColor: color }"
            @click="copyColor(color)"
          ></div>
        </div>
        
        <div class="palette-info">
          <h4>{{ palette.name }}</h4>
          <p class="date">{{ formatDate(palette.createdAt) }}</p>
          <p class="color-count">{{ palette.colors.length }} цветов</p>
        </div>
        
        <div class="palette-actions">
          <button @click="loadPalette(palette)" class="action-btn load">
            Загрузить
          </button>
          <button @click="deletePalette(palette.id)" class="action-btn delete">
            Удалить
          </button>
          <button @click="editPalette(palette)" class="action-btn edit">
            Редактировать
          </button>
        </div>
      </div>
    </div>

    <!-- Модальное окно редактирования -->
    <div v-if="editingPalette" class="modal-overlay" @click="closeModal">
      <div class="modal" @click.stop>
        <h3>Редактирование палитры</h3>
        <input 
          v-model="editingPalette.name"
          placeholder="Название палитры"
          class="modal-input"
        />
        
        <div class="modal-colors">
          <div 
            v-for="(color, index) in editingPalette.colors" 
            :key="index"
            class="modal-color"
          >
            <input 
              v-model="editingPalette.colors[index]"
              type="color"
              class="color-picker"
            />
            <input 
              v-model="editingPalette.colors[index]"
              placeholder="#FFFFFF"
              class="color-input"
            />
            <button @click="removeColor(index)" class="remove-color">×</button>
          </div>
        </div>
        
        <button @click="addColor" class="add-color-btn">+ Добавить цвет</button>
        
        <div class="modal-actions">
          <button @click="saveEdit" class="save-edit-btn">Сохранить</button>
          <button @click="closeModal" class="cancel-btn">Отмена</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'

export default {
  name: 'PaletteLibrary',
  setup() {
    const savedPalettes = ref([])
    const searchQuery = ref('')
    const sortBy = ref('date')
    const editingPalette = ref(null)

    const loadPalettes = () => {
      const palettes = localStorage.getItem('savedPalettes')
      savedPalettes.value = palettes ? JSON.parse(palettes) : []
    }

    const filteredPalettes = computed(() => {
      if (!searchQuery.value) return savedPalettes.value
      return savedPalettes.value.filter(palette =>
        palette.name.toLowerCase().includes(searchQuery.value.toLowerCase())
      )
    })

    // Сортировка палитр
    const sortedPalettes = computed(() => {
      const palettes = [...filteredPalettes.value]
      
      switch (sortBy.value) {
        case 'name':
          return palettes.sort((a, b) => a.name.localeCompare(b.name))
        case 'colors':
          return palettes.sort((a, b) => b.colors.length - a.colors.length)
        case 'date':
        default:
          return palettes.sort((a, b) => 
            new Date(b.createdAt) - new Date(a.createdAt)
          )
      }
    })

    // Форматирование даты
    const formatDate = (dateString) => {
      return new Date(dateString).toLocaleDateString('ru-RU', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      })
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

    // Загрузка палитры в генератор
    const loadPalette = (palette) => {
      localStorage.setItem('palette', JSON.stringify(palette.colors))
      localStorage.setItem('lockedColors', JSON.stringify(
        new Array(palette.colors.length).fill(false)
      ))
      alert(`Палитра "${palette.name}" загружена в генератор!`)
    }

    // Удаление палитры
    const deletePalette = (id) => {
      if (confirm('Вы уверены, что хотите удалить эту палитру?')) {
        savedPalettes.value = savedPalettes.value.filter(p => p.id !== id)
        localStorage.setItem('savedPalettes', JSON.stringify(savedPalettes.value))
      }
    }

    // Редактирование палитры
    const editPalette = (palette) => {
      editingPalette.value = JSON.parse(JSON.stringify(palette))
    }

    // Сохранение изменений
    const saveEdit = () => {
      const index = savedPalettes.value.findIndex(p => p.id === editingPalette.value.id)
      if (index !== -1) {
        savedPalettes.value[index] = { ...editingPalette.value }
        localStorage.setItem('savedPalettes', JSON.stringify(savedPalettes.value))
      }
      closeModal()
    }

    // Добавление цвета при редактировании
    const addColor = () => {
      editingPalette.value.colors.push('#FFFFFF')
    }

    // Удаление цвета при редактировании
    const removeColor = (index) => {
      editingPalette.value.colors.splice(index, 1)
    }

    // Закрытие модального окна
    const closeModal = () => {
      editingPalette.value = null
    }

    // Инициализация
    onMounted(() => {
      loadPalettes()
    })

    return {
      savedPalettes,
      searchQuery,
      sortBy,
      editingPalette,
      filteredPalettes,
      sortedPalettes,
      formatDate,
      copyColor,
      loadPalette,
      deletePalette,
      editPalette,
      saveEdit,
      addColor,
      removeColor,
      closeModal
    }
  }
}
</script>

<style scoped>
.palette-library {
  max-width: 1200px;
  margin: 0 auto;
}

.library-header {
  margin-bottom: 2rem;
  padding: 1.5rem;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.search-controls {
  display: flex;
  gap: 1rem;
  margin-top: 1rem;
}

.search-input {
  flex: 1;
  padding: 0.75rem;
  border: 2px solid #ddd;
  border-radius: 5px;
  font-size: 1rem;
}

.sort-select {
  padding: 0.75rem;
  border: 2px solid #ddd;
  border-radius: 5px;
  font-size: 1rem;
  background-color: white;
}

.empty-state {
  text-align: center;
  padding: 4rem 2rem;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  color: #666;
}

.palettes-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 1.5rem;
}

.palette-card {
  background-color: white;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  transition: transform 0.3s ease;
}

.palette-card:hover {
  transform: translateY(-5px);
}

.palette-colors {
  display: flex;
  height: 100px;
}

.library-color {
  flex: 1;
  cursor: pointer;
  transition: all 0.3s ease;
}

.library-color:hover {
  transform: scale(1.1);
  z-index: 1;
  box-shadow: 0 0 10px rgba(0,0,0,0.3);
}

.palette-info {
  padding: 1rem;
}

.palette-info h4 {
  margin-bottom: 0.5rem;
  color: #333;
}

.date {
  font-size: 0.9rem;
  color: #666;
  margin-bottom: 0.25rem;
}

.color-count {
  font-size: 0.9rem;
  color: #667eea;
  font-weight: bold;
}

.palette-actions {
  padding: 1rem;
  display: flex;
  gap: 0.5rem;
  border-top: 1px solid #eee;
}

.action-btn {
  flex: 1;
  padding: 0.5rem;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: background-color 0.3s ease;
}

.action-btn.load {
  background-color: #e3f2fd;
  color: #1976d2;
}

.action-btn.delete {
  background-color: #ffebee;
  color: #d32f2f;
}

.action-btn.edit {
  background-color: #f3e5f5;
  color: #7b1fa2;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal {
  background-color: white;
  padding: 2rem;
  border-radius: 10px;
  max-width: 500px;
  width: 90%;
  max-height: 80vh;
  overflow-y: auto;
}

.modal h3 {
  margin-bottom: 1.5rem;
  color: #333;
}

.modal-input {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid #ddd;
  border-radius: 5px;
  margin-bottom: 1.5rem;
  font-size: 1rem;
}

.modal-colors {
  margin-bottom: 1.5rem;
}

.modal-color {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 0.5rem;
  align-items: center;
}

.color-picker {
  width: 50px;
  height: 50px;
  border: none;
  cursor: pointer;
}

.color-input {
  flex: 1;
  padding: 0.5rem;
  border: 2px solid #ddd;
  border-radius: 5px;
  font-family: monospace;
}

.remove-color {
  width: 30px;
  height: 30px;
  border: none;
  background-color: #ff4757;
  color: white;
  border-radius: 50%;
  cursor: pointer;
  font-size: 1.2rem;
}

.add-color-btn {
  width: 100%;
  padding: 0.75rem;
  background-color: #f8f9fa;
  border: 2px dashed #ddd;
  border-radius: 5px;
  cursor: pointer;
  margin-bottom: 1.5rem;
  font-size: 1rem;
}

.modal-actions {
  display: flex;
  gap: 1rem;
}

.save-edit-btn {
  flex: 2;
  padding: 0.75rem;
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
}

.cancel-btn {
  flex: 1;
  padding: 0.75rem;
  background-color: #dc3545;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
</style>