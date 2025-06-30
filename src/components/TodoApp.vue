<template>
  <div class="container">
    <!-- Header dengan Search -->
    <div class="header">
      <h1>🚀 Todo list Accell</h1>
      <p>Kelola tugas Anda dengan mudah dan efisien</p>
      
      <!-- Search Bar -->
      <div class="search-container">
        <input 
          v-model="searchQuery"
          type="text" 
          placeholder="🔍 Cari tugas..." 
          class="search-input"
        />
        <button v-if="searchQuery" @click="clearSearch" class="clear-search">
          ✕
        </button>
      </div>
    </div>

    <!-- Todo Form -->
    <TodoForm 
      :form="form"
      :editing-id="editingId"
      @add-todo="addTodo"
      @update-todo="updateTodo"
      @cancel-edit="cancelEdit"
    />

    <!-- Filter Controls dengan Badge -->
    <div class="filter-controls">
      <button 
        @click="currentFilter = 'all'" 
        class="filter-btn"
        :class="{ active: currentFilter === 'all' }"
      >
        <span class="filter-icon">📋</span>
        Semua
        <span class="badge">{{ todos.length }}</span>
      </button>
      <button 
        @click="currentFilter = 'pending'" 
        class="filter-btn"
        :class="{ active: currentFilter === 'pending' }"
      >
        <span class="filter-icon">⏳</span>
        Belum Selesai
        <span class="badge pending">{{ pendingCount }}</span>
      </button>
      <button 
        @click="currentFilter = 'completed'" 
        class="filter-btn"
        :class="{ active: currentFilter === 'completed' }"
      >
        <span class="filter-icon">✅</span>
        Selesai
        <span class="badge completed">{{ completedCount }}</span>
      </button>
    </div>

    <!-- Sort Controls -->
    <div class="sort-controls">
      <label>Urutkan: </label>
      <select v-model="sortBy" class="sort-select">
        <option value="newest">Terbaru</option>
        <option value="oldest">Terlama</option>
        <option value="title">Judul A-Z</option>
        <option value="priority">Prioritas</option>
      </select>
    </div>

    <!-- Todo List -->
    <TodoList 
      :todos="sortedAndFilteredTodos"
      :current-filter="currentFilter"
      @toggle-todo="toggleTodo"
      @edit-todo="editTodo"
      @delete-todo="deleteTodo"
      @toggle-priority="togglePriority"
    />

    <!-- Empty State -->
    <div v-if="sortedAndFilteredTodos.length === 0 && !searchQuery" class="empty-state">
      <div class="empty-icon">📝</div>
      <h3>Belum ada tugas</h3>
      <p>Mulai dengan menambahkan tugas pertama Anda!</p>
    </div>

    <!-- No Search Results -->
    <div v-if="sortedAndFilteredTodos.length === 0 && searchQuery" class="empty-state">
      <div class="empty-icon">🔍</div>
      <h3>Tidak ada hasil</h3>
      <p>Tidak ditemukan tugas dengan kata kunci "{{ searchQuery }}"</p>
    </div>

    <!-- Statistics dengan Progress Bar -->
    <div v-if="todos.length > 0" class="stats">
      <div class="progress-section">
        <div class="progress-header">
          <span>Progress Keseluruhan</span>
          <span class="progress-percentage">{{ completionPercentage }}%</span>
        </div>
        <div class="progress-bar">
          <div 
            class="progress-fill" 
            :style="{ width: completionPercentage + '%' }"
          ></div>
        </div>
      </div>
      
      <div class="stat-grid">
        <div class="stat-item total">
          <span class="stat-number">{{ todos.length }}</span>
          <span class="stat-label">Total Tugas</span>
        </div>
        <div class="stat-item completed">
          <span class="stat-number">{{ completedCount }}</span>
          <span class="stat-label">Selesai</span>
        </div>
        <div class="stat-item pending">
          <span class="stat-number">{{ pendingCount }}</span>
          <span class="stat-label">Pending</span>
        </div>
        <div class="stat-item priority">
          <span class="stat-number">{{ priorityCount }}</span>
          <span class="stat-label">Prioritas Tinggi</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted, watch } from 'vue'
import axios from 'axios'
import TodoForm from './TodoForm.vue'
import TodoList from './TodoList.vue'

// API Base URL - pastikan JSON Server berjalan di port 3000
const API_BASE_URL = 'http://localhost:3000'

export default {
  name: 'TodoApp',
  components: {
    TodoForm,
    TodoList
  },
  setup() {
    const todos = ref([])
    const form = ref({
      title: '',
      description: '',
      priority: false
    })
    const currentFilter = ref('all')
    const editingId = ref(null)
    const searchQuery = ref('')
    const sortBy = ref('newest')

    // Computed Properties
    const completedCount = computed(() => 
      todos.value.filter(todo => todo.completed).length
    )

    const pendingCount = computed(() => 
      todos.value.filter(todo => !todo.completed).length
    )

    const priorityCount = computed(() => 
      todos.value.filter(todo => todo.priority && !todo.completed).length
    )

    const completionPercentage = computed(() => {
      if (todos.value.length === 0) return 0
      return Math.round((completedCount.value / todos.value.length) * 100)
    })

    const filteredTodos = computed(() => {
      let filtered = todos.value

      // Filter berdasarkan status
      switch (currentFilter.value) {
        case 'completed':
          filtered = filtered.filter(todo => todo.completed)
          break
        case 'pending':
          filtered = filtered.filter(todo => !todo.completed)
          break
        default:
          // 'all' - tidak perlu filter
          break
      }

      // Filter berdasarkan search query
      if (searchQuery.value.trim()) {
        const query = searchQuery.value.toLowerCase().trim()
        filtered = filtered.filter(todo => 
          todo.title.toLowerCase().includes(query) ||
          (todo.description && todo.description.toLowerCase().includes(query))
        )
      }

      return filtered
    })

    const sortedAndFilteredTodos = computed(() => {
      const filtered = [...filteredTodos.value]

      switch (sortBy.value) {
        case 'oldest':
          return filtered.sort((a, b) => new Date(a.createdAt) - new Date(b.createdAt))
        case 'title':
          return filtered.sort((a, b) => a.title.localeCompare(b.title))
        case 'priority':
          return filtered.sort((a, b) => {
            if (a.priority && !b.priority) return -1
            if (!a.priority && b.priority) return 1
            return new Date(b.createdAt) - new Date(a.createdAt)
          })
        default: // 'newest'
          return filtered.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt))
      }
    })

    // Methods
    const fetchTodos = async () => {
      try {
        const response = await axios.get(`${API_BASE_URL}/todos`)
        todos.value = response.data
      } catch (error) {
        console.error('Error fetching todos:', error)
        alert('Gagal memuat data. Pastikan JSON Server berjalan di port 3000')
      }
    }

    const addTodo = async () => {
      if (!form.value.title.trim()) return

      try {
        const newTodo = {
          title: form.value.title.trim(),
          description: form.value.description.trim(),
          completed: false,
          priority: form.value.priority || false,
          createdAt: new Date().toISOString()
        }

        const response = await axios.post(`${API_BASE_URL}/todos`, newTodo)
        todos.value.push(response.data)
        resetForm()
      } catch (error) {
        console.error('Error adding todo:', error)
        alert('Gagal menambah tugas')
      }
    }

    const updateTodo = async () => {
      if (!form.value.title.trim() || !editingId.value) return

      try {
        const updatedTodo = {
          ...todos.value.find(t => t.id === editingId.value),
          title: form.value.title.trim(),
          description: form.value.description.trim(),
          priority: form.value.priority || false
        }

        await axios.put(`${API_BASE_URL}/todos/${editingId.value}`, updatedTodo)
        
        const index = todos.value.findIndex(t => t.id === editingId.value)
        if (index !== -1) {
          todos.value[index] = updatedTodo
        }
        
        cancelEdit()
      } catch (error) {
        console.error('Error updating todo:', error)
        alert('Gagal mengupdate tugas')
      }
    }

    const deleteTodo = async (id) => {
      if (!confirm('Apakah Anda yakin ingin menghapus tugas ini?')) return

      try {
        await axios.delete(`${API_BASE_URL}/todos/${id}`)
        todos.value = todos.value.filter(todo => todo.id !== id)
      } catch (error) {
        console.error('Error deleting todo:', error)
        alert('Gagal menghapus tugas')
      }
    }

    const toggleTodo = async (id) => {
      try {
        const todo = todos.value.find(t => t.id === id)
        const updatedTodo = { ...todo, completed: !todo.completed }

        await axios.put(`${API_BASE_URL}/todos/${id}`, updatedTodo)
        
        const index = todos.value.findIndex(t => t.id === id)
        if (index !== -1) {
          todos.value[index] = updatedTodo
        }
      } catch (error) {
        console.error('Error toggling todo:', error)
        alert('Gagal mengubah status tugas')
      }
    }

    const togglePriority = async (id) => {
      try {
        const todo = todos.value.find(t => t.id === id)
        const updatedTodo = { ...todo, priority: !todo.priority }

        await axios.put(`${API_BASE_URL}/todos/${id}`, updatedTodo)
        
        const index = todos.value.findIndex(t => t.id === id)
        if (index !== -1) {
          todos.value[index] = updatedTodo
        }
      } catch (error) {
        console.error('Error toggling priority:', error)
        alert('Gagal mengubah prioritas tugas')
      }
    }

    const editTodo = (todo) => {
      form.value.title = todo.title
      form.value.description = todo.description || ''
      form.value.priority = todo.priority || false
      editingId.value = todo.id
      window.scrollTo({ top: 0, behavior: 'smooth' })
    }

    const cancelEdit = () => {
      resetForm()
      editingId.value = null
    }

    const clearSearch = () => {
      searchQuery.value = ''
    }

    const resetForm = () => {
      form.value.title = ''
      form.value.description = ''
      form.value.priority = false
    }

    // Auto-save search query to localStorage (jika diperlukan)
    watch(searchQuery, (newQuery) => {
      // Bisa ditambahkan untuk menyimpan riwayat pencarian
    })

    // Lifecycle
    onMounted(() => {
      fetchTodos()
    })

    return {
      todos,
      form,
      currentFilter,
      editingId,
      searchQuery,
      sortBy,
      completedCount,
      pendingCount,
      priorityCount,
      completionPercentage,
      filteredTodos,
      sortedAndFilteredTodos,
      addTodo,
      updateTodo,
      deleteTodo,
      toggleTodo,
      togglePriority,
      editTodo,
      cancelEdit,
      clearSearch
    }
  }
}
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.header {
  text-align: center;
  margin-bottom: 30px;
}

.header h1 {
  color: #2c3e50;
  margin-bottom: 10px;
  font-size: 2.5rem;
}

.search-container {
  position: relative;
  max-width: 400px;
  margin: 20px auto;
}

.search-input {
  width: 100%;
  padding: 12px 20px;
  border: 2px solid #e1e8ed;
  border-radius: 25px;
  font-size: 16px;
  transition: all 0.3s ease;
}

.search-input:focus {
  outline: none;
  border-color: #3498db;
  box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.1);
}

.clear-search {
  position: absolute;
  right: 15px;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  cursor: pointer;
  color: #95a5a6;
  font-size: 18px;
}

.filter-controls {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.filter-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 16px;
  border: 2px solid #e1e8ed;
  background: white;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.filter-btn:hover {
  background: #f8f9fa;
}

.filter-btn.active {
  background: #3498db;
  color: white;
  border-color: #3498db;
}

.badge {
  background: #e1e8ed;
  color: #2c3e50;
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: bold;
}

.filter-btn.active .badge {
  background: rgba(255, 255, 255, 0.3);
  color: white;
}

.badge.pending {
  background: #f39c12;
  color: white;
}

.badge.completed {
  background: #27ae60;
  color: white;
}

.sort-controls {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 20px;
}

.sort-select {
  padding: 8px 12px;
  border: 2px solid #e1e8ed;
  border-radius: 8px;
  font-size: 14px;
}

.empty-state {
  text-align: center;
  padding: 60px 20px;
  color: #7f8c8d;
}

.empty-icon {
  font-size: 4rem;
  margin-bottom: 20px;
}

.stats {
  margin-top: 40px;
  padding: 20px;
  background: #f8f9fa;
  border-radius: 15px;
}

.progress-section {
  margin-bottom: 20px;
}

.progress-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
  font-weight: bold;
}

.progress-bar {
  height: 10px;
  background: #e1e8ed;
  border-radius: 5px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #3498db, #27ae60);
  transition: width 0.3s ease;
}

.stat-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 15px;
}

.stat-item {
  text-align: center;
  padding: 15px;
  border-radius: 10px;
  background: white;
}

.stat-item.total {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.stat-item.completed {
  background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
  color: white;
}

.stat-item.pending {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  color: white;
}

.stat-item.priority {
  background: linear-gradient(135deg, #ffecd2 0%, #fcb69f 100%);
  color: #2c3e50;
}

.stat-number {
  display: block;
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 5px;
}

.stat-label {
  font-size: 12px;
  opacity: 0.9;
}

@media (max-width: 768px) {
  .filter-controls {
    flex-direction: column;
  }
  
  .filter-btn {
    justify-content: center;
  }
}
</style>