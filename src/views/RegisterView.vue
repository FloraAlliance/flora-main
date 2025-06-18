<template>
  <div class="auth-page">
    <div class="register-container" :class="{ 'fade-in': show }">
      <div class="auth-header">
        <h2>Регистрация</h2>
      </div>

      <form @submit.prevent="handleRegister" class="auth-form">
        <div class="form-group">
          <label for="username" class="form-label">Имя пользователя</label>
          <div class="input-wrapper">
            <i class="fas fa-user input-icon"></i>
            <input
              id="username"
              v-model="form.username"
              type="text"
              required
              placeholder="Введите логин"
              class="input-field"
            />
          </div>
        </div>

        <div class="form-group">
          <label for="password" class="form-label">Пароль</label>
          <div class="input-wrapper">
            <i class="fas fa-lock input-icon"></i>
            <input
              id="password"
              v-model="form.password"
              type="password"
              required
              placeholder="Введите пароль (минимум 8 символов)"
              class="input-field"
              @input="validatePassword"
            />
          </div>
        </div>

        <div class="form-group">
          <label for="confirmPassword" class="form-label">Подтверждение пароля</label>
          <div class="input-wrapper">
            <i class="fas fa-lock input-icon"></i>
            <input
              id="confirmPassword"
              v-model="form.confirmPassword"
              type="password"
              required
              placeholder="Подтвердите пароль"
              class="input-field"
              @input="validatePassword"
            />
          </div>
        </div>

        <div class="form-group">
          <label for="email" class="form-label">Email</label>
          <div class="input-wrapper">
            <i class="fas fa-envelope input-icon"></i>
            <input
              id="email"
              v-model="form.email"
              type="email"
              required
              placeholder="Введите email"
              class="input-field"
            />
          </div>
        </div>

        <div class="form-group">
          <label for="phone" class="form-label">Телефон</label>
          <div class="input-wrapper">
            <i class="fas fa-phone input-icon"></i>
            <input
              id="phone"
              v-model="form.phone"
              type="tel"
              placeholder="Введите телефон"
              class="input-field"
            />
          </div>
        </div>

        <button type="submit" :disabled="loading || !isFormValid" class="submit-btn">
          <span v-if="loading" class="btn-content">
            <span class="spinner"></span>
            <span>Регистрация...</span>
          </span>
          <span v-else class="btn-content">
            <i class="fas fa-user-plus"></i>
            Зарегистрироваться
          </span>
        </button>

        <transition name="fade">
          <div v-if="error" class="error-message">
            <i class="fas fa-exclamation-circle"></i>
            {{ error }}
          </div>
        </transition>

        <transition name="fade">
          <div v-if="passwordError" class="error-message">
            <i class="fas fa-exclamation-circle"></i>
            {{ passwordError }}
          </div>
        </transition>

        <div class="auth-footer">
          <p>Уже есть аккаунт? <router-link to="/login" class="auth-link">Войти</router-link></p>
        </div>
      </form>
    </div>
  </div>
</template>
  
<script setup>
import { ref, computed, onMounted } from 'vue'
import { useAuthStore } from '@/store/auth.js'

const authStore = useAuthStore()
const form = ref({
  username: '',
  password: '',
  confirmPassword: '',
  email: '',
  phone: ''
})

const loading = ref(false)
const error = ref(null)
const passwordError = ref('')
const show = ref(false)

onMounted(() => {
  setTimeout(() => {
    show.value = true
  }, 100)
})

const isFormValid = computed(() => {
  return (
    form.value.username &&
    form.value.password &&
    form.value.password.length >= 8 &&
    form.value.password === form.value.confirmPassword &&
    form.value.email &&
    !passwordError.value
  )
})

const validatePassword = () => {
  if (form.value.password.length < 8) {
    passwordError.value = 'Пароль должен содержать минимум 8 символов'
  } else if (form.value.password !== form.value.confirmPassword) {
    passwordError.value = 'Пароли не совпадают'
  } else {
    passwordError.value = ''
  }
}

const handleRegister = async () => {
  try {
    loading.value = true
    error.value = null
    await authStore.register(form.value)
  } catch (err) {
    error.value = err.response?.data || 'Ошибка регистрации'
  } finally {
    loading.value = false
  }
}
</script>
  
<style scoped>
.auth-page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #ffffff 0%, #f8f9fa 100%);
  padding: 20px;
}

.register-container {
  width: 100%;
  max-width: 400px;
  background: white;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  padding: 40px;
  opacity: 0;
  transform: translateY(20px);
  transition: all 0.5s ease-out;
}

.auth-header {
  text-align: center;
  margin-bottom: 40px;
}

h2 {
  font-family: 'Manrope', sans-serif;
  color: #333;
  font-size: 28px;
  font-weight: 600;
  margin: 0;
}

.auth-form {
  margin-top: 30px;
}

.form-group {
  margin-bottom: 20px;
}

.form-label {
  display: block;
  margin-bottom: 8px;
  color: #666;
  font-size: 14px;
  font-weight: 500;
}

.input-wrapper {
  position: relative;
}

.input-icon {
  position: absolute;
  left: 12px;
  top: 50%;
  transform: translateY(-50%);
  color: #4CAF50;
  font-size: 16px;
}

.input-field {
  width: 100%;
  padding: 12px 12px 12px 40px;
  border: 2px solid #e0e0e0;
  border-radius: 12px;
  font-size: 15px;
  color: #333;
  background: white;
  transition: all 0.3s ease;
}

.input-field:focus {
  outline: none;
  border-color: #4CAF50;
  box-shadow: 0 0 0 3px rgba(76, 175, 80, 0.1);
}

.input-field:hover {
  border-color: #4CAF50;
}

.submit-btn {
  width: 100%;
  padding: 14px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 12px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.submit-btn:hover:not(:disabled) {
  background: #43a047;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(76, 175, 80, 0.2);
}

.submit-btn:active:not(:disabled) {
  transform: translateY(0);
}

.submit-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
  transform: none;
}

.btn-content {
  display: flex;
  align-items: center;
  gap: 8px;
}

.spinner {
  width: 20px;
  height: 20px;
  border: 3px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  border-top-color: white;
  animation: spin 1s linear infinite;
}

.error-message {
  margin-top: 20px;
  padding: 12px;
  background: #ffebee;
  color: #d32f2f;
  border-radius: 12px;
  font-size: 14px;
  display: flex;
  align-items: center;
  gap: 8px;
  animation: shake 0.5s ease;
}

.auth-footer {
  margin-top: 30px;
  text-align: center;
  color: #666;
  font-size: 14px;
}

.auth-link {
  color: #4CAF50;
  text-decoration: none;
  font-weight: 500;
  transition: color 0.3s ease;
}

.auth-link:hover {
  color: #43a047;
  text-decoration: underline;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20%, 60% { transform: translateX(-5px); }
  40%, 80% { transform: translateX(5px); }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.fade-in {
  opacity: 1;
  transform: translateY(0);
}

@media (max-width: 480px) {
  .register-container {
    padding: 30px 20px;
  }

  .auth-logo {
    width: 100px;
  }

  h2 {
    font-size: 20px;
  }

  .input-field {
    font-size: 14px;
  }

  .submit-btn {
    padding: 12px;
    font-size: 15px;
  }
}
</style>