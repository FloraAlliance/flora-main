<template>
  <div class="profile-container fade-in">
    <div class="profile-card">
      <div class="profile-header">
        <div class="profile-avatar">
          <span class="avatar-placeholder">{{ user?.username?.charAt(0).toUpperCase() || '?' }}</span>
        </div>
        <div class="profile-title">
          <h2>{{ user?.username || 'Загрузка...' }}</h2>
          <span class="user-role">{{ authStore.isAdmin ? 'Администратор' : 'Пользователь' }}</span>
        </div>
      </div>

      <div v-if="isLoading" class="loading-overlay">
        <div class="spinner"></div>
        <span>Загрузка профиля...</span>
      </div>
      
      <div v-if="user" class="profile-content">
        <div class="profile-section">
          <div class="section-header">
            <i class="fas fa-user-circle"></i>
            <h3>Контактная информация</h3>
        </div>
          <div class="profile-fields">
        <div class="profile-field">
              <span class="field-label">Email</span>
          <span class="field-value">{{ user.email || 'Не указан' }}</span>
        </div>
        
        <div class="profile-field">
              <span class="field-label">Телефон</span>
          <span class="field-value">{{ user.phone || 'Не указан' }}</span>
            </div>
          </div>
        </div>
        
        <!-- Секция заказов -->
        <div v-if="user.orders" class="orders-section">
          <div class="section-header">
            <i class="fas fa-shopping-bag"></i>
            <h3>История заказов</h3>
      </div>

        <div class="orders-tabs">
          <button 
              v-for="tab in ['pending', 'completed', 'cancelled']"
              :key="tab"
            class="tab-button" 
              :class="{ active: currentTab === tab }"
              @click="currentTab = tab"
              :data-tab="tab"
            >
              {{ getTabName(tab) }}
              <span class="order-count">{{ getOrderCount(tab) }}</span>
          </button>
        </div>

        <div v-if="filteredOrders.length === 0" class="no-orders">
            <i class="fas fa-box-open"></i>
            <p>{{ getEmptyMessage(currentTab) }}</p>
        </div>
        
        <div v-else class="orders-list">
          <div v-for="order in filteredOrders" :key="order.id" class="order-card">
            <div class="order-header">
              <div class="order-info">
                  <span class="order-code">Заказ #{{ order.code }}</span>
                <span class="order-date">{{ formatDate(order.created_at) }}</span>
              </div>
              <div class="order-status" :class="order.status">
                {{ getStatusText(order.status) }}
              </div>
            </div>
            
            <div v-if="order.status === 'cancelled' && order.cancel_reason" class="cancel-reason">
                <i class="fas fa-exclamation-circle"></i>
                <div>
                  <h4>Причина отмены</h4>
              <p>{{ order.cancel_reason }}</p>
                </div>
            </div>

            <div class="order-items">
              <div v-for="item in order.items" :key="item.product.id" class="order-item">
                <img 
                  :src="getImageUrl(item.product.image)" 
                  :alt="item.product.name"
                  class="item-image"
                  @error="handleImageError"
                >
                <div class="item-details">
                  <span class="item-name">{{ item.product.name }}</span>
                  <span class="item-article">Артикул: {{ item.product.article }}</span>
                  <div class="item-price-qty">
                    <span class="item-quantity">{{ item.quantity }} шт.</span>
                    <span class="item-price">{{ item.price }} ₽</span>
                  </div>
                </div>
              </div>
            </div>
            
            <div class="order-footer">
              <span class="order-total">Итого: {{ order.total_price }} ₽</span>
            </div>
          </div>
        </div>
      </div>
      </div>

      <div class="button-group">
        <router-link 
          v-if="authStore.isAdmin" 
          to="/admin-panel"
          class="admin-btn"
        >
          <i class="fas fa-cog"></i>
          <span>Админ-панель</span>
        </router-link>
      
      <button 
        @click="showLogoutConfirm = true" 
        class="logout-btn"
        :disabled="isLoading"
      >
          <i class="fas fa-sign-out-alt"></i>
          <span>Выйти</span>
      </button>
      </div>
    </div>

    <!-- Модальное окно подтверждения выхода -->
    <div v-if="showLogoutConfirm" class="modal-overlay" @click.self="showLogoutConfirm = false">
      <div class="confirm-modal">
        <div class="modal-header">
          <i class="fas fa-exclamation-triangle"></i>
        <h3>Подтверждение выхода</h3>
        </div>
        <p>Вы действительно хотите выйти из аккаунта?</p>
        
        <div class="modal-buttons">
          <button @click="showLogoutConfirm = false" class="cancel-btn">
            <i class="fas fa-times"></i>
            <span>Отмена</span>
          </button>
          <button @click="confirmLogout" class="confirm-btn">
            <i class="fas fa-sign-out-alt"></i>
            <span>Выйти</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import axios from 'axios'
import { useAuthStore } from '@/store/auth.js'

const authStore = useAuthStore()
const user = ref(null)
const isLoading = ref(false)
const showLogoutConfirm = ref(false)
const baseUrl = 'http://localhost:8000'
const currentTab = ref('pending')

// Вспомогательные функции для UI
const getTabName = (tab) => {
  const names = {
    'pending': 'Ожидают получения',
    'completed': 'Завершенные',
    'cancelled': 'Отмененные'
  }
  return names[tab]
}

const getOrderCount = (tab) => {
  if (!user.value?.orders) return 0
  return user.value.orders.filter(order => order.status === tab).length
}

const getEmptyMessage = (tab) => {
  const messages = {
    'pending': 'У вас нет заказов, ожидающих получения',
    'completed': 'У вас нет завершенных заказов',
    'cancelled': 'У вас нет отмененных заказов'
  }
  return messages[tab]
}

// Остальной код без изменений
const filteredOrders = computed(() => {
  if (!user.value?.orders) return []
  return user.value.orders.filter(order => order.status === currentTab.value)
})

const confirmLogout = () => {
  showLogoutConfirm.value = false
  authStore.logout()
}

const formatDate = (dateString) => {
  const options = { 
    year: 'numeric', 
    month: 'long', 
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  }
  return new Date(dateString).toLocaleDateString('ru-RU', options)
}

const getImageUrl = (imagePath) => {
  if (!imagePath) return require('@/assets/img/no-image.webp')
  if (imagePath.startsWith('http')) return imagePath
  return `${baseUrl}${imagePath}`
}

const handleImageError = (e) => {
  e.target.src = require('@/assets/img/no-image.webp')
}

const getStatusText = (status) => {
  const statusMap = {
    'pending': 'Ожидает получения',
    'completed': 'Получен',
    'cancelled': 'Отменён'
  }
  return statusMap[status] || status
}

onMounted(async () => {
  try {
    isLoading.value = true
    const response = await axios.get('http://localhost:8000/api/auth/user/', {
      headers: {
        'Authorization': `Bearer ${authStore.accessToken}`,
        'Content-Type': 'application/json'
      },
      withCredentials: true
    })
    user.value = response.data
  } catch (error) {
    console.error('Ошибка загрузки профиля:', error)
    if (error.response?.status === 403) {
      alert('Сессия истекла. Пожалуйста, войдите снова.')
      authStore.logout()
    }
  } finally {
    isLoading.value = false
  }
})
</script>

<style scoped>
.profile-container {
  min-height: calc(100vh - 160px);
  background: #f7fafc;
  padding: 2rem 0;
}

.profile-card {
  width: 100%;
  background: white;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.profile-header {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
  display: flex;
  align-items: center;
  gap: 2rem;
  border-bottom: 1px solid #e2e8f0;
}

.profile-avatar {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  background: linear-gradient(135deg, #6366f1 0%, #4f46e5 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 6px rgba(99, 102, 241, 0.2);
}

.avatar-placeholder {
  font-size: 3rem;
  color: white;
  font-weight: 600;
}

.profile-title {
  flex: 1;
}

.profile-title h2 {
  font-size: 2rem;
  color: #1a202c;
  margin: 0 0 0.5rem 0;
  font-weight: 700;
}

.user-role {
  color: #6366f1;
  font-size: 0.875rem;
  font-weight: 500;
  background: rgba(99, 102, 241, 0.1);
  padding: 0.25rem 0.75rem;
  border-radius: 9999px;
}

.profile-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
}

.section-header {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.section-header i {
  font-size: 1.5rem;
  color: #6366f1;
}

.section-header h3 {
  font-size: 1.25rem;
  color: #1a202c;
  margin: 0;
  font-weight: 600;
}

.profile-section {
  background: white;
  border-radius: 1rem;
  padding: 2rem;
  margin-bottom: 2rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.profile-fields {
  display: grid;
  gap: 1.5rem;
}

.profile-field {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  padding: 1rem;
  background: #f8fafc;
  border-radius: 0.5rem;
  transition: all 0.2s;
}

.profile-field:hover {
  background: #f1f5f9;
}

.field-label {
  color: #64748b;
  font-size: 0.875rem;
  font-weight: 500;
}

.field-value {
  color: #1a202c;
  font-size: 1rem;
  font-weight: 500;
}

.orders-section {
  background: white;
  border-radius: 1rem;
  padding: 2rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.orders-tabs {
  display: flex;
  gap: 1rem;
  margin-bottom: 2rem;
  border-bottom: 1px solid #e2e8f0;
  padding-bottom: 1rem;
  }

.tab-button {
  padding: 0.75rem 1.5rem;
  border: none;
  background: none;
  color: #64748b;
  font-weight: 500;
  cursor: pointer;
  position: relative;
  transition: all 0.2s;
  }
  
.tab-button.active {
  color: #6366f1;
  }

.tab-button.active::after {
  content: '';
  position: absolute;
  bottom: -1rem;
  left: 0;
  right: 0;
  height: 2px;
  background: #6366f1;
}

.order-count {
  margin-left: 0.5rem;
  background: #e2e8f0;
  padding: 0.25rem 0.5rem;
  border-radius: 9999px;
  font-size: 0.75rem;
  color: #64748b;
}

.tab-button.active .order-count {
  background: rgba(99, 102, 241, 0.1);
  color: #6366f1;
}

.no-orders {
  text-align: center;
  padding: 3rem;
  color: #64748b;
}

.no-orders i {
  font-size: 3rem;
  color: #e2e8f0;
  margin-bottom: 1rem;
}

.orders-list {
  display: grid;
  gap: 1.5rem;
}

.order-card {
  background: #f8fafc;
  border-radius: 1rem;
  padding: 1.5rem;
  transition: all 0.2s;
}

.order-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
}

.order-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid #e2e8f0;
}

.order-info {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.order-code {
  font-weight: 600;
  color: #1a202c;
}

.order-date {
  color: #64748b;
  font-size: 0.875rem;
}

.order-status {
  padding: 0.5rem 1rem;
  border-radius: 9999px;
  font-size: 0.875rem;
  font-weight: 500;
}

.order-status.pending {
  background: rgba(234, 179, 8, 0.1);
  color: #b45309;
}

.order-status.completed {
  background: rgba(34, 197, 94, 0.1);
  color: #16a34a;
}

.order-status.cancelled {
  background: rgba(239, 68, 68, 0.1);
  color: #dc2626;
}

.cancel-reason {
  display: flex;
  gap: 1rem;
  padding: 1rem;
  background: rgba(239, 68, 68, 0.05);
  border-radius: 0.5rem;
  margin-bottom: 1.5rem;
}

.cancel-reason i {
  color: #dc2626;
  font-size: 1.25rem;
}

.cancel-reason h4 {
  color: #dc2626;
  margin: 0 0 0.25rem 0;
  font-size: 0.875rem;
  font-weight: 600;
}

.cancel-reason p {
  color: #64748b;
  margin: 0;
  font-size: 0.875rem;
}

.order-items {
  display: grid;
  gap: 1rem;
}

.order-item {
  display: flex;
  gap: 1.5rem;
  padding: 1rem;
  background: white;
  border-radius: 0.5rem;
}

.item-image {
  width: 80px;
  height: 80px;
  object-fit: cover;
  border-radius: 0.5rem;
}

.item-details {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.item-name {
  font-weight: 600;
  color: #1a202c;
}

.item-article {
  color: #64748b;
  font-size: 0.875rem;
}

.item-price-qty {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: auto;
}

.item-quantity {
  color: #64748b;
  font-size: 0.875rem;
}

.item-price {
  font-weight: 600;
  color: #1a202c;
}

.order-footer {
  margin-top: 1.5rem;
  padding-top: 1rem;
  border-top: 1px solid #e2e8f0;
  text-align: right;
}

.order-total {
  font-weight: 600;
  color: #1a202c;
  font-size: 1.125rem;
}

.button-group {
  max-width: 1200px;
  margin: 2rem auto;
  padding: 0 2rem;
  display: flex;
  gap: 1rem;
}

.admin-btn, .logout-btn {
  flex: 1;
  padding: 1rem;
  border-radius: 0.5rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  transition: all 0.2s;
}

.admin-btn {
  background: #6366f1;
  color: white;
  text-decoration: none;
}

.admin-btn:hover {
  background: #4f46e5;
  transform: translateY(-1px);
}

.logout-btn {
  background: #ef4444;
  color: white;
  border: none;
  cursor: pointer;
}

.logout-btn:hover {
  background: #dc2626;
  transform: translateY(-1px);
}

.logout-btn:disabled {
  background: #e2e8f0;
  cursor: not-allowed;
  transform: none;
}

/* Модальное окно */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(4px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  animation: fadeIn 0.3s ease-out forwards;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.confirm-modal {
  background: #EBF9E0;
  border-radius: 1rem;
  padding: 2rem;
  width: 90%;
  max-width: 400px;
  box-shadow: 0 10px 30px rgba(44, 118, 59, 0.2);
  border: 2px solid #2C763B;
  animation: modalSlideIn 0.4s cubic-bezier(0.22, 1, 0.36, 1) forwards;
}

@keyframes modalSlideIn {
  from { transform: translateY(-20px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

.modal-header {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1rem;
}

.modal-header i {
  color: #d75480;
  font-size: 1.5rem;
}

.modal-header h3 {
  color: #2C763B;
  margin: 0;
  font-size: 1.25rem;
  font-weight: 600;
}

.confirm-modal p {
  color: #1f5429;
  margin-bottom: 2rem;
}

.modal-buttons {
  display: flex;
  gap: 1rem;
}

.cancel-btn, .confirm-btn {
  flex: 1;
  padding: 0.75rem;
  border-radius: 0.5rem;
  font-weight: 500;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  transition: all 0.2s;
}

.cancel-btn {
  background: #dff6be;
  color: #2C763B;
  border: 1px solid #2C763B;
}

.cancel-btn:hover {
  background: #EBF9E0;
}

.confirm-btn {
  background: #2C763B;
  color: #EBF9E0;
  border: none;
}

.confirm-btn:hover {
  background: #1f5429;
}

/* Адаптивность */
@media (max-width: 768px) {
  .profile-header {
    flex-direction: column;
    text-align: center;
    gap: 1rem;
  }

  .profile-avatar {
    width: 100px;
    height: 100px;
  }

  .avatar-placeholder {
    font-size: 2.5rem;
  }

  .profile-title h2 {
    font-size: 1.5rem;
  }

  .orders-tabs {
    flex-wrap: wrap;
  }

  .tab-button {
    flex: 1;
    min-width: 150px;
  }

  .order-header {
    flex-direction: column;
    gap: 1rem;
    align-items: flex-start;
  }

  .order-item {
    flex-direction: column;
  }

  .item-image {
    width: 100%;
    height: 200px;
  }

  .button-group {
    flex-direction: column;
  }

  .admin-btn, .logout-btn {
    width: 100%;
}
}

/* Анимации */
@keyframes fadeIn {
  from {
  opacity: 0;
    transform: translateY(20px);
}
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.loading-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.9);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 1rem;
  z-index: 10;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 3px solid #e2e8f0;
  border-radius: 50%;
  border-top-color: #6366f1;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>