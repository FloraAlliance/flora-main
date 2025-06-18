<template>
    <div class="modal-overlay">
      <div class="modal">
        <h2>{{ category.id ? 'Редактировать категорию' : 'Добавить категорию' }}</h2>
        
        <form @submit.prevent="handleSubmit">
          <div class="form-group">
            <label>Название:</label>
            <input v-model="formData.name" required>
          </div>
          
          <div class="form-group">
            <label>Изображение:</label>
            <input type="file" @change="handleImageUpload">
            <img v-if="formData.image && typeof formData.image === 'string'" :src="formData.image" class="preview-image">
          </div>
          
          <div class="form-actions">
            <button type="button" @click="$emit('close')">Отмена</button>
            <button type="submit">Сохранить</button>
          </div>
        </form>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, watch } from 'vue'
  import { defineProps, defineEmits } from 'vue'
  
  const props = defineProps({
    category: {
      type: Object,
      required: true
    }
  })
  
  const emit = defineEmits(['close', 'save'])
  
  const formData = ref({
    name: '',
    image: null
  })
  
  watch(() => props.category, (newVal) => {
    formData.value = { ...newVal }
  }, { immediate: true })
  
  const handleImageUpload = (e) => {
    formData.value.image = e.target.files[0]
  }
  
  const handleSubmit = () => {
    emit('save', formData.value)
  }
  </script>
  
<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding-top: 15vh;
  z-index: 1000;
  backdrop-filter: blur(5px);
  animation: fadeIn 0.3s ease-out forwards;
}

.modal {
  background: var(--bg-primary);
  padding: 30px;
  border-radius: 16px;
  width: 500px;
  max-width: 90%;
  border: 1px solid var(--border-dark);
  box-shadow: var(--shadow-lg);
  animation: modalSlideIn 0.4s cubic-bezier(0.22, 1, 0.36, 1) forwards;
  transform: translateY(-20px);
  opacity: 0;
}

h2 {
  margin: 0;
  color: var(--text-dark);
  font-family: 'Manrope', sans-serif;
  font-weight: 500;
  font-size: 1.5rem;
  margin-bottom: 1.5rem;
}

.form-group {
  margin-bottom: 20px;
}

label {
  display: block;
  margin-bottom: 8px;
  color: var(--text-muted);
  font-family: 'Manrope', sans-serif;
  font-size: 0.9rem;
}

input, 
select,
textarea {
  width: 100%;
  padding: 12px 15px;
  background: var(--bg-neutral);
  border: 1px solid var(--border-dark);
  border-radius: 8px;
  color: var(--text-dark);
  font-family: 'Manrope', sans-serif;
  transition: all 0.3s ease;
}

input:focus,
select:focus,
textarea:focus {
  outline: none;
  border-color: var(--secondary-color);
  box-shadow: 0 0 0 2px rgba(44, 118, 59, 0.2);
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 30px;
}

button {
  padding: 12px 24px;
  border-radius: 8px;
  font-family: 'Manrope', sans-serif;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
}

button[type="button"] {
  background: var(--bg-neutral);
  border: 1px solid var(--border-dark);
  color: var(--text-dark);
}

button[type="button"]:hover {
  background: var(--hover-primary);
}

button[type="submit"] {
  background: var(--bg-secondary);
  border: none;
  color: var(--text-light);
}

button[type="submit"]:hover {
  background: var(--hover-secondary);
}

.preview-image {
  max-width: 100%;
  max-height: 200px;
  margin-top: 15px;
  border-radius: 8px;
  border: 1px solid var(--border-dark);
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes modalSlideIn {
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

@media (max-width: 600px) {
  .modal {
    padding: 20px;
  }
  
  .form-actions {
    flex-direction: column-reverse;
    gap: 10px;
  }
  
  button {
    width: 100%;
  }
}
</style>