<template>
  <div class="todo-form">
    <div class="form-header">
      <h3 v-if="!editingId" class="form-title">
        ➕ Tambah Tugas Baru
      </h3>
      <h3 v-else class="form-title editing">
        ✏️ Edit Tugas
      </h3>
    </div>

    <div class="form-container">
      <!-- Title Input -->
      <div class="form-group">
        <label class="form-label">
          <span class="label-text">Judul Tugas</span>
          <span class="required">*</span>
        </label>
        <input 
          v-model="form.title" 
          type="text" 
          placeholder="Masukkan judul tugas..." 
          class="form-input"
          :class="{ 'error': titleError }"
          @keyup.enter="handleSubmit"
          @blur="validateTitle"
          @input="clearTitleError"
          maxlength="100"
        >
        <div v-if="titleError" class="error-message">
          {{ titleError }}
        </div>
        <div class="char-counter">
          {{ form.title.length }}/100
        </div>
      </div>

      <!-- Description Input -->
      <div class="form-group">
        <label class="form-label">
          <span class="label-text">Deskripsi</span>
          <span class="optional">(opsional)</span>
        </label>
        <textarea 
          v-model="form.description" 
          placeholder="Tambahkan deskripsi detail untuk tugas ini..." 
          class="form-textarea"
          rows="4"
          maxlength="500"
        ></textarea>
        <div class="char-counter">
          {{ form.description.length }}/500
        </div>
      </div>

      <!-- Priority Toggle -->
      <div class="form-group">
        <label class="priority-label">
          <input 
            v-model="form.priority" 
            type="checkbox" 
            class="priority-checkbox"
          >
          <span class="priority-text">
            <span class="priority-icon">{{ form.priority ? '🔥' : '⭐' }}</span>
            {{ form.priority ? 'Prioritas Tinggi' : 'Tandai sebagai prioritas tinggi' }}
          </span>
        </label>
        <div v-if="form.priority" class="priority-note">
          Tugas dengan prioritas tinggi akan ditampilkan di urutan atas
        </div>
      </div>

      <!-- Category Selection -->
      <div class="form-group">
        <label class="form-label">
          <span class="label-text">Kategori</span>
        </label>
        <select v-model="form.category" class="form-select">
          <option value="">Pilih kategori...</option>
          <option value="work">💼 Pekerjaan</option>
          <option value="personal">👤 Pribadi</option>
          <option value="study">📚 Belajar</option>
          <option value="health">🏥 Kesehatan</option>
          <option value="shopping">🛒 Belanja</option>
          <option value="other">📝 Lainnya</option>
        </select>
      </div>

      <!-- Due Date -->
      <div class="form-group">
        <label class="form-label">
          <span class="label-text">Tenggat Waktu</span>
        </label>
        <input 
          v-model="form.dueDate" 
          type="datetime-local" 
          class="form-input"
          :min="minDate"
        >
        <div v-if="form.dueDate" class="due-date-info">
          📅 {{ formatDueDate(form.dueDate) }}
        </div>
      </div>

      <!-- Action Buttons -->
      <div class="form-actions">
        <button 
          @click="handleSubmit" 
          class="btn btn-primary"
          :disabled="!canSubmit"
          :class="{ 'loading': isSubmitting }"
        >
          <span v-if="!isSubmitting">
            {{ editingId ? '💾 Update Tugas' : '➕ Tambah Tugas' }}
          </span>
          <span v-else class="loading-text">
            {{ editingId ? 'Mengupdate...' : 'Menambahkan...' }}
          </span>
        </button>
        
        <button 
          v-if="editingId" 
          @click="handleCancel" 
          class="btn btn-secondary"
          :disabled="isSubmitting"
        >
          ❌ Batal
        </button>

        <button 
          v-if="!editingId && hasFormData" 
          @click="clearForm" 
          class="btn btn-ghost"
          type="button"
        >
          🗑️ Bersihkan
        </button>
      </div>

      <!-- Form Tips -->
      <div v-if="!editingId" class="form-tips">
        <div class="tip-item">
          💡 <strong>Tips:</strong> Tekan Enter pada judul untuk langsung menambah tugas
        </div>
        <div class="tip-item">
          🎯 Gunakan prioritas tinggi untuk tugas yang mendesak
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue'

export default {
  name: 'TodoForm',
  props: {
    form: {
      type: Object,
      required: true
    },
    editingId: {
      type: [Number, String, null],
      default: null
    }
  },
  emits: ['add-todo', 'update-todo', 'cancel-edit'],
  setup(props, { emit }) {
    const titleError = ref('')
    const isSubmitting = ref(false)

    // Computed properties
    const canSubmit = computed(() => {
      return props.form.title.trim().length > 0 && !titleError.value && !isSubmitting.value
    })

    const hasFormData = computed(() => {
      return props.form.title || props.form.description || props.form.priority || 
             props.form.category || props.form.dueDate
    })

    const minDate = computed(() => {
      const now = new Date()
      const year = now.getFullYear()
      const month = String(now.getMonth() + 1).padStart(2, '0')
      const day = String(now.getDate()).padStart(2, '0')
      const hours = String(now.getHours()).padStart(2, '0')
      const minutes = String(now.getMinutes()).padStart(2, '0')
      return `${year}-${month}-${day}T${hours}:${minutes}`
    })

    // Methods
    const validateTitle = () => {
      const title = props.form.title.trim()
      if (!title) {
        titleError.value = 'Judul tugas wajib diisi'
      } else if (title.length < 3) {
        titleError.value = 'Judul tugas minimal 3 karakter'
      } else if (title.length > 100) {
        titleError.value = 'Judul tugas maksimal 100 karakter'
      } else {
        titleError.value = ''
      }
    }

    const clearTitleError = () => {
      if (titleError.value && props.form.title.trim()) {
        titleError.value = ''
      }
    }

    const formatDueDate = (dateString) => {
      if (!dateString) return ''
      
      const date = new Date(dateString)
      const now = new Date()
      
      const options = {
        weekday: 'long',
        year: 'numeric',
        month: 'long',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      }
      
      const formatted = date.toLocaleDateString('id-ID', options)
      
      // Check if it's overdue
      if (date < now) {
        return `${formatted} (Terlambat!)`
      } else if (date.toDateString() === now.toDateString()) {
        return `${formatted} (Hari ini)`
      } else {
        const diffTime = date - now
        const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24))
        if (diffDays === 1) {
          return `${formatted} (Besok)`
        } else if (diffDays <= 7) {
          return `${formatted} (${diffDays} hari lagi)`
        }
      }
      
      return formatted
    }

    const handleSubmit = async () => {
      validateTitle()
      if (!canSubmit.value) return

      isSubmitting.value = true
      
      try {
        // Simulate network delay
        await new Promise(resolve => setTimeout(resolve, 500))
        
        if (props.editingId) {
          emit('update-todo')
        } else {
          emit('add-todo')
        }
      } finally {
        isSubmitting.value = false
      }
    }

    const handleCancel = () => {
      emit('cancel-edit')
    }

    const clearForm = () => {
      if (confirm('Apakah Anda yakin ingin membersihkan form?')) {
        Object.keys(props.form).forEach(key => {
          if (typeof props.form[key] === 'boolean') {
            props.form[key] = false
          } else {
            props.form[key] = ''
          }
        })
        titleError.value = ''
      }
    }

    // Initialize form with default values if needed
    if (!props.form.category) {
      props.form.category = ''
    }
    if (!props.form.dueDate) {
      props.form.dueDate = ''
    }

    // Watch for form changes to provide real-time validation
    watch(() => props.form.title, (newTitle) => {
      if (titleError.value && newTitle.trim()) {
        validateTitle()
      }
    })

    return {
      titleError,
      isSubmitting,
      canSubmit,
      hasFormData,
      minDate,
      validateTitle,
      clearTitleError,
      formatDueDate,
      handleSubmit,
      handleCancel,
      clearForm
    }
  }
}
</script>

<style scoped>
.todo-form {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 20px;
  padding: 25px;
  margin-bottom: 30px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
}

.form-header {
  text-align: center;
  margin-bottom: 25px;
}

.form-title {
  color: white;
  margin: 0;
  font-size: 1.5rem;
  font-weight: 600;
}

.form-title.editing {
  background: linear-gradient(45deg, #f093fb 0%, #f5576c 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.form-container {
  background: white;
  border-radius: 15px;
  padding: 25px;
}

.form-group {
  margin-bottom: 20px;
}

.form-label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #2c3e50;
}

.label-text {
  margin-right: 5px;
}

.required {
  color: #e74c3c;
}

.optional {
  color: #7f8c8d;
  font-size: 0.9em;
  font-weight: normal;
}

.form-input,
.form-textarea,
.form-select {
  width: 100%;
  padding: 12px 16px;
  border: 2px solid #e1e8ed;
  border-radius: 10px;
  font-size: 16px;
  transition: all 0.3s ease;
  box-sizing: border-box;
}

.form-input:focus,
.form-textarea:focus,
.form-select:focus {
  outline: none;
  border-color: #3498db;
  box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.1);
}

.form-input.error {
  border-color: #e74c3c;
  box-shadow: 0 0 0 3px rgba(231, 76, 60, 0.1);
}

.form-textarea {
  resize: vertical;
  min-height: 80px;
}

.char-counter {
  text-align: right;
  font-size: 12px;
  color: #7f8c8d;
  margin-top: 5px;
}

.error-message {
  color: #e74c3c;
  font-size: 14px;
  margin-top: 5px;
  display: flex;
  align-items: center;
}

.error-message::before {
  content: '⚠️';
  margin-right: 5px;
}

.priority-label {
  display: flex;
  align-items: center;
  cursor: pointer;
  padding: 12px;
  border: 2px solid #e1e8ed;
  border-radius: 10px;
  transition: all 0.3s ease;
}

.priority-label:hover {
  background: #f8f9fa;
  border-color: #3498db;
}

.priority-checkbox {
  margin-right: 12px;
  transform: scale(1.2);
}

.priority-text {
  display: flex;
  align-items: center;
  font-weight: 500;
}

.priority-icon {
  margin-right: 8px;
  font-size: 1.2em;
}

.priority-note {
  background: #fff3cd;
  color: #856404;
  padding: 8px 12px;
  border-radius: 6px;
  font-size: 12px;
  margin-top: 8px;
  border-left: 4px solid #ffc107;
}

.due-date-info {
  background: #d1ecf1;
  color: #0c5460;
  padding: 8px 12px;
  border-radius: 6px;
  font-size: 14px;
  margin-top: 8px;
  border-left: 4px solid #17a2b8;
}

.form-actions {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
  margin-top: 25px;
}

.btn {
  padding: 12px 24px;
  border: none;
  border-radius: 10px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 48px;
}

.btn-primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  flex: 1;
}

.btn-primary:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
}

.btn-primary:disabled {
  background: #bdc3c7;
  cursor: not-allowed;
  transform: none;
}

.btn-primary.loading {
  background: #95a5a6;
}

.btn-secondary {
  background: #e74c3c;
  color: white;
}

.btn-secondary:hover:not(:disabled) {
  background: #c0392b;
  transform: translateY(-1px);
}

.btn-ghost {
  background: transparent;
  color: #7f8c8d;
  border: 2px solid #e1e8ed;
}

.btn-ghost:hover {
  background: #f8f9fa;
  border-color: #bdc3c7;
}

.loading-text {
  display: flex;
  align-items: center;
}

.loading-text::after {
  content: '';
  width: 16px;
  height: 16px;
  margin-left: 8px;
  border: 2px solid transparent;
  border-top: 2px solid currentColor;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.form-tips {
  background: #f8f9fa;
  border-radius: 10px;
  padding: 15px;
  margin-top: 20px;
  border-left: 4px solid #3498db;
}

.tip-item {
  color: #5a6c7d;
  font-size: 14px;
  line-height: 1.5;
  margin-bottom: 5px;
}

.tip-item:last-child {
  margin-bottom: 0;
}

@media (max-width: 768px) {
  .todo-form {
    margin: 0 -10px 30px -10px;
    border-radius: 15px;
  }
  
  .form-actions {
    flex-direction: column;
  }
  
  .btn {
    width: 100%;
  }
}
</style>