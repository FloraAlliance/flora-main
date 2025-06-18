<template>
    <div class="modal-overlay">
      <div class="modal">
        <h2>{{ user.id ? 'Редактировать пользователя' : 'Добавить пользователя' }}</h2>
        
        <form @submit.prevent="handleSubmit">
          <div class="form-group">
            <label>Имя пользователя:</label>
            <input v-model="formData.username" required>
          </div>
          
          <div class="form-group">
            <label>Email:</label>
            <input v-model="formData.email" type="email">
          </div>
          
          <div class="form-group" v-if="!user.id">
            <label>Пароль:</label>
            <input v-model="formData.password" type="password" required>
          </div>
          
          <div class="form-group">
            <label>Телефон:</label>
            <input v-model="formData.phone">
          </div>
          
          <div class="form-group">
            <label>Роль:</label>
            <select v-model="formData.role" required>
              <option value="admin">Администратор</option>
              <option value="customer">Покупатель</option>
            </select>
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
    user: {
      type: Object,
      required: true
    }
  })
  
  const emit = defineEmits(['close', 'save'])
  
  const formData = ref({
    username: '',
    email: '',
    phone: '',
    role: 'customer',
    password: ''
  })
  
  watch(() => props.user, (newVal) => {
    formData.value = { ...newVal }
  }, { immediate: true })
  
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
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding-top: 15vh;
  z-index: 1000;
  backdrop-filter: blur(5px);
  animation: fadeIn 0.3s ease-out forwards;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.modal {
  background: #EBF9E0;
  padding: 30px;
  border-radius: 16px;
  width: 500px;
  max-width: 90%;
  border: 2px solid #2C763B;
  box-shadow: 0 10px 30px rgba(44, 118, 59, 0.2);
  animation: modalSlideIn 0.4s cubic-bezier(0.22, 1, 0.36, 1) forwards;
  transform: translateY(-20px);
  opacity: 0;
}

@keyframes modalSlideIn {
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
}

h2 {
  margin: 0;
  color: #2C763B;
  font-family: 'Manrope', sans-serif;
  font-weight: 500;
  font-size: 1.5rem;
}

.close-btn {
  background: none;
  border: none;
  color: #2C763B;
  font-size: 1.5rem;
  cursor: pointer;
  transition: color 0.2s;
}

.close-btn:hover {
  color: #1f5429;
}

.form-group {
  margin-bottom: 20px;
}

label {
  display: block;
  margin-bottom: 8px;
  color: #2C763B;
  font-family: 'Manrope', sans-serif;
  font-size: 0.9rem;
}

input, 
select,
textarea {
  width: 100%;
  padding: 12px 15px;
  background: #dff6be;
  border: 1px solid #2C763B;
  border-radius: 8px;
  color: #2C763B;
  font-family: 'Manrope', sans-serif;
  transition: all 0.3s ease;
}

input:focus,
select:focus,
textarea:focus {
  outline: none;
  border-color: #2C763B;
  box-shadow: 0 0 0 2px rgba(44, 118, 59, 0.2);
}

textarea {
  min-height: 100px;
  resize: vertical;
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

.cancel-btn {
  background: #dff6be;
  border: 1px solid #2C763B;
  color: #2C763B;
}

.cancel-btn:hover {
  background: #EBF9E0;
  transform: translateY(-1px);
}

.submit-btn {
  background: #2C763B;
  border: 1px solid #2C763B;
  color: #EBF9E0;
}

.submit-btn:hover {
  background: #1f5429;
  transform: translateY(-1px);
}

.image-preview {
  max-width: 100%;
  max-height: 200px;
  margin-top: 15px;
  border-radius: 8px;
  border: 1px solid #4a4a4a;
}

.file-input {
  padding: 10px;
  background: rgba(50, 50, 50, 0.5);
  border-radius: 8px;
  margin-top: 5px;
}

/* Анимация ошибки */
.error-message {
  color: #d75480;
  font-size: 0.85rem;
  margin-top: 5px;
  animation: fadeIn 0.3s ease-out;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-5px); }
  to { opacity: 1; transform: translateY(0); }
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