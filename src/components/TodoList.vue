<template>
  <div class="todo-container">
    <!-- Empty State dengan animasi -->
    <transition name="fade">
      <div v-if="todos.length === 0" class="empty-state-wrapper">
        <div class="empty-icon">
          <svg width="64" height="64" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M9 11H3v5a2 2 0 0 0 2 2h6l-2-7Z"/>
            <path d="M13 11h8l-2 7h-6v-7Z"/>
            <path d="M9 7v1a3 3 0 0 0 6 0V7a3 3 0 0 0-6 0Z"/>
          </svg>
        </div>
        <div class="empty-content">
          <h3 class="empty-title">{{ getEmptyMessage() }}</h3>
          <p class="empty-description">{{ getEmptyDescription() }}</p>
        </div>
      </div>
    </transition>

    <!-- Todo Items dengan animasi list -->
    <transition-group 
      v-if="todos.length > 0"
      name="todo-list" 
      tag="div" 
      class="todo-items-container"
    >
      <div 
        v-for="todo in todos" 
        :key="todo.id"
        class="todo-item-wrapper"
      >
        <TodoItem 
          :todo="todo"
          @toggle-todo="handleToggleTodo"
          @edit-todo="handleEditTodo"
          @delete-todo="handleDeleteTodo"
        />
      </div>
    </transition-group>

    <!-- Summary info -->
    <div v-if="todos.length > 0" class="todo-summary">
      <span class="summary-text">
        {{ getTodoSummary() }}
      </span>
    </div>
  </div>
</template>

<script>
import TodoItem from './TodoItem.vue'

export default {
  name: 'TodoList',
  components: {
    TodoItem
  },
  props: {
    todos: {
      type: Array,
      required: true,
      default: () => []
    },
    currentFilter: {
      type: String,
      required: true,
      default: 'all'
    }
  },
  emits: ['toggle-todo', 'edit-todo', 'delete-todo'],
  computed: {
    completedCount() {
      return this.todos.filter(todo => todo.completed).length
    },
    pendingCount() {
      return this.todos.filter(todo => !todo.completed).length
    }
  },
  methods: {
    getEmptyMessage() {
      const messages = {
        completed: 'Belum ada tugas yang diselesaikan',
        pending: 'Hebat! Semua tugas telah selesai! 🎉',
        all: 'Daftar tugas masih kosong'
      }
      return messages[this.currentFilter] || messages.all
    },
    
    getEmptyDescription() {
      const descriptions = {
        completed: 'Tugas yang telah diselesaikan akan ditampilkan di sini',
        pending: 'Fantastis! Tidak ada tugas yang tertunda',
        all: 'Tambahkan tugas baru untuk memulai'
      }
      return descriptions[this.currentFilter] || descriptions.all
    },

    getTodoSummary() {
      const total = this.todos.length
      if (this.currentFilter === 'completed') {
        return `${this.completedCount} tugas selesai`
      } else if (this.currentFilter === 'pending') {
        return `${this.pendingCount} tugas belum selesai`
      } else {
        return `${this.completedCount} dari ${total} tugas selesai`
      }
    },

    handleToggleTodo(todoId) {
      this.$emit('toggle-todo', todoId)
    },

    handleEditTodo(payload) {
      this.$emit('edit-todo', payload)
    },

    handleDeleteTodo(todoId) {
      this.$emit('delete-todo', todoId)
    }
  }
}
</script>

<style scoped>
.todo-container {
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
  padding: 1rem;
}

/* Empty State Styles */
.empty-state-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 3rem 1rem;
  text-align: center;
  background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
  border-radius: 16px;
  border: 2px dashed #dee2e6;
  margin: 2rem 0;
}

.empty-icon {
  margin-bottom: 1.5rem;
  color: #6c757d;
  opacity: 0.7;
}

.empty-content {
  max-width: 400px;
}

.empty-title {
  font-size: 1.5rem;
  font-weight: 600;
  color: #495057;
  margin-bottom: 0.5rem;
}

.empty-description {
  font-size: 1rem;
  color: #6c757d;
  margin: 0;
  line-height: 1.5;
}

/* Todo Items Container */
.todo-items-container {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  margin: 1rem 0;
}

.todo-item-wrapper {
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  border: 1px solid #e9ecef;
  transition: all 0.3s ease;
}

.todo-item-wrapper:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
}

/* Summary Styles */
.todo-summary {
  display: flex;
  justify-content: center;
  margin-top: 1.5rem;
  padding: 1rem;
  background: #f8f9fa;
  border-radius: 8px;
  border-left: 4px solid #007bff;
}

.summary-text {
  font-size: 0.9rem;
  color: #495057;
  font-weight: 500;
}

/* Animations */
.fade-enter-active, .fade-leave-active {
  transition: all 0.5s ease;
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
  transform: translateY(20px);
}

.todo-list-enter-active, .todo-list-leave-active {
  transition: all 0.4s ease;
}

.todo-list-enter-from, .todo-list-leave-to {
  opacity: 0;
  transform: translateX(-20px);
}

.todo-list-move {
  transition: transform 0.4s ease;
}

/* Responsive Design */
@media (max-width: 768px) {
  .todo-container {
    padding: 0.5rem;
  }
  
  .empty-state-wrapper {
    padding: 2rem 1rem;
    margin: 1rem 0;
  }
  
  .empty-title {
    font-size: 1.25rem;
  }
  
  .empty-description {
    font-size: 0.9rem;
  }
  
  .todo-items-container {
    gap: 0.5rem;
  }
}
</style>