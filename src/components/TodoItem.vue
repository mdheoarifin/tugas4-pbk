<template>
  <div 
    class="todo-item"
    :class="{ 
      completed: todo.completed,
      priority: todo.priority && !todo.completed,
      overdue: isOverdue
    }"
  >
    <div class="todo-checkbox">
      <input 
        type="checkbox" 
        :checked="todo.completed"
        @change="handleToggle"
        class="checkbox"
      >
    </div>
    
    <div class="todo-content">
      <div class="todo-header">
        <h3 class="todo-title" :class="{ completed: todo.completed }">
          {{ todo.title }}
        </h3>
        <div class="todo-badges">
          <span v-if="todo.priority && !todo.completed" class="badge priority">🔥</span>
          <span v-if="todo.category" class="badge category">{{ getCategoryIcon(todo.category) }}</span>
          <span v-if="isOverdue && !todo.completed" class="badge overdue">⏰</span>
        </div>
      </div>
      
      <p v-if="todo.description" class="todo-description">
        {{ todo.description }}
      </p>
      
      <div class="todo-meta">
        <span class="todo-date">{{ formatDate(todo.createdAt) }}</span>
        <span v-if="todo.dueDate" class="due-date" :class="{ overdue: isOverdue }">
          {{ formatDueDate(todo.dueDate) }}
        </span>
      </div>
    </div>
    
    <div class="todo-actions">
      <button 
        v-if="todo.priority"
        @click="handleTogglePriority" 
        class="btn btn-sm btn-priority"
        :class="{ active: todo.priority }"
        title="Toggle Priority"
      >
        ⭐
      </button>
      <button 
        @click="handleEdit" 
        class="btn btn-sm btn-edit"
        title="Edit"
      >
        ✏️
      </button>
      <button 
        @click="handleDelete" 
        class="btn btn-sm btn-delete"
        title="Hapus"
      >
        🗑️
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TodoItem',
  props: {
    todo: {
      type: Object,
      required: true
    }
  },
  emits: ['toggle-todo', 'edit-todo', 'delete-todo', 'toggle-priority'],
  computed: {
    isOverdue() {
      if (!this.todo.dueDate || this.todo.completed) return false
      return new Date(this.todo.dueDate) < new Date()
    }
  },
  methods: {
    handleToggle() {
      this.$emit('toggle-todo', this.todo.id)
    },
    handleEdit() {
      this.$emit('edit-todo', this.todo)
    },
    handleDelete() {
      this.$emit('delete-todo', this.todo.id)
    },
    handleTogglePriority() {
      this.$emit('toggle-priority', this.todo.id)
    },
    getCategoryIcon(category) {
      const icons = {
        work: '💼',
        personal: '👤',
        study: '📚',
        health: '🏥',
        shopping: '🛒',
        other: '📝'
      }
      return icons[category] || '📝'
    },
    formatDate(dateString) {
      const date = new Date(dateString)
      const now = new Date()
      const diffTime = Math.abs(now - date)
      const diffDays = Math.floor(diffTime / (1000 * 60 * 60 * 24))
      
      if (diffDays === 0) return 'Hari ini'
      if (diffDays === 1) return 'Kemarin'
      if (diffDays < 7) return `${diffDays} hari lalu`
      
      return date.toLocaleDateString('id-ID', {
        day: 'numeric',
        month: 'short',
        year: 'numeric'
      })
    },
    formatDueDate(dateString) {
      const date = new Date(dateString)
      const now = new Date()
      
      if (date < now) return 'Terlambat'
      if (date.toDateString() === now.toDateString()) return 'Hari ini'
      
      const diffTime = date - now
      const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24))
      if (diffDays === 1) return 'Besok'
      if (diffDays <= 7) return `${diffDays} hari`
      
      return date.toLocaleDateString('id-ID', { day: 'numeric', month: 'short' })
    }
  }
}
</script>

<style scoped>
.todo-item {
  display: flex;
  align-items: flex-start;
  gap: 15px;
  padding: 16px;
  background: white;
  border-radius: 12px;
  border: 2px solid #f1f3f4;
  margin-bottom: 12px;
  transition: all 0.3s ease;
}

.todo-item:hover {
  border-color: #e3f2fd;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

.todo-item.priority {
  border-left: 4px solid #ff6b35;
  background: linear-gradient(90deg, #fff5f5 0%, white 100%);
}

.todo-item.completed {
  opacity: 0.7;
  background: #f8f9fa;
}

.todo-item.overdue {
  border-left: 4px solid #e74c3c;
  background: linear-gradient(90deg, #fdf2f2 0%, white 100%);
}

.checkbox {
  width: 20px;
  height: 20px;
  margin-top: 2px;
}

.todo-content {
  flex: 1;
}

.todo-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 8px;
}

.todo-title {
  margin: 0;
  font-size: 16px;
  font-weight: 600;
  color: #2c3e50;
  line-height: 1.3;
}

.todo-title.completed {
  text-decoration: line-through;
  color: #7f8c8d;
}

.todo-badges {
  display: flex;
  gap: 6px;
  flex-shrink: 0;
}

.badge {
  font-size: 12px;
  padding: 2px 6px;
  border-radius: 12px;
  font-weight: 500;
}

.badge.priority {
  background: #ffe8e0;
  color: #d63031;
}

.badge.category {
  background: #e3f2fd;
  color: #1976d2;
}

.badge.overdue {
  background: #ffebee;
  color: #c62828;
}

.todo-description {
  margin: 0 0 8px 0;
  color: #5a6c7d;
  font-size: 14px;
  line-height: 1.4;
}

.todo-meta {
  display: flex;
  gap: 12px;
  font-size: 12px;
  color: #7f8c8d;
}

.due-date.overdue {
  color: #e74c3c;
  font-weight: 600;
}

.todo-actions {
  display: flex;
  gap: 4px;
  flex-shrink: 0;
}

.btn {
  border: none;
  background: none;
  cursor: pointer;
  padding: 6px;
  border-radius: 6px;
  transition: all 0.2s ease;
}

.btn:hover {
  background: #f1f3f4;
}

.btn-priority.active {
  background: #fff3e0;
  color: #f57c00;
}

.btn-edit:hover {
  background: #e3f2fd;
}

.btn-delete:hover {
  background: #ffebee;
}

@media (max-width: 768px) {
  .todo-item {
    padding: 12px;
    gap: 12px;
  }
  
  .todo-header {
    flex-direction: column;
    gap: 8px;
  }
  
  .todo-badges {
    align-self: flex-start;
  }
}
</style>