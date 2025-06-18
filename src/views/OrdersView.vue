<template>
  <div class="orders-container">
    <div class="orders-header">
    <h1>Мои заказы</h1>
      <router-link to="/catalog" class="browse-catalog">
        <i class="fas fa-shopping-cart"></i>
        Перейти к покупкам
      </router-link>
    </div>
    
    <div v-if="isLoading" class="loading">
      <div class="spinner"></div>
      <span>Загрузка заказов...</span>
    </div>
    
    <div v-else-if="error" class="error-message">
      <i class="fas fa-exclamation-circle"></i>
      <div class="error-content">
        <p>{{ error }}</p>
        <button @click="fetchOrders" class="retry-button">
          <i class="fas fa-sync-alt"></i>
          Повторить попытку
        </button>
      </div>
    </div>
    
    <div v-else-if="orders.length === 0" class="no-orders">
      <i class="fas fa-box-open"></i>
      <p>У вас пока нет заказов</p>
      <router-link to="/catalog" class="browse-catalog">
        <i class="fas fa-shopping-cart"></i>
        Перейти к покупкам
      </router-link>
    </div>
    
    <div v-else class="orders-list">
      <div v-for="order in orders" :key="order.id" class="order-card">
        <div class="order-header">
          <div class="order-info">
            <div class="order-id">
              <i class="fas fa-shopping-bag"></i>
            <span class="order-number">Заказ #{{ order.id }}</span>
            </div>
            <span class="order-date">
              <i class="far fa-calendar-alt"></i>
              {{ formatDate(order.created_at) }}
            </span>
          </div>
          <div class="order-status" :class="order.status">
            <i :class="getStatusIcon(order.status)"></i>
            {{ getStatusText(order.status) }}
          </div>
        </div>
        
        <div class="order-items">
          <div v-for="item in order.items" :key="item.id" class="order-item">
            <div class="item-image-container">
            <img :src="getImageUrl(item.product.image)" :alt="item.product.name">
            </div>
            <div class="item-details">
              <div class="item-main-info">
              <h3>{{ item.product.name }}</h3>
                <p class="item-article">Артикул: {{ item.product.article || 'Не указан' }}</p>
              </div>
              <div class="item-price-info">
                <p class="item-quantity">
                  <i class="fas fa-times"></i>
                  {{ item.quantity }} шт.
                </p>
              <p class="item-price">{{ item.product.price }} ₽</p>
              </div>
            </div>
          </div>
        </div>
        
        <div class="order-footer">
          <div class="order-total">
            <span>Итого:</span>
            <span class="total-price">{{ calculateTotal(order.items) }} ₽</span>
          </div>
          
          <div v-if="order.code" class="order-code">
            <span>Код получения:</span>
            <span class="code">{{ order.code }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const orders = ref([])
const isLoading = ref(true)
const error = ref(null)
const baseUrl = 'https://floraalliance.pythonanywhere.com'

const fetchOrders = async () => {
  try {
    isLoading.value = true
    error.value = null
    const response = await axios.get(`${baseUrl}/api/orders/`)
    orders.value = response.data
  } catch (err) {
    error.value = 'Не удалось загрузить заказы'
    console.error('Error fetching orders:', err)
  } finally {
    isLoading.value = false
  }
}

const formatDate = (dateString) => {
  return new Date(dateString).toLocaleDateString('ru-RU', {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  })
}

const getStatusText = (status) => {
  const statusMap = {
    'pending': 'Ожидает получения',
    'completed': 'Получен',
    'cancelled': 'Отменён'
  }
  return statusMap[status] || status
}

const getStatusIcon = (status) => {
  const iconMap = {
    'pending': 'fas fa-clock',
    'completed': 'fas fa-check-circle',
    'cancelled': 'fas fa-times-circle'
  }
  return iconMap[status] || 'fas fa-question-circle'
}

const calculateTotal = (items) => {
  return items.reduce((total, item) => total + (item.product.price * item.quantity), 0)
}

const getImageUrl = (imagePath) => {
  if (!imagePath) return require('@/assets/img/no-image.webp')
  if (imagePath.startsWith('http')) return imagePath
  return `${baseUrl}${imagePath}`
}

onMounted(() => {
  fetchOrders()
})
</script>

<style>
.orders-container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 2rem;
  min-height: 100vh;
  background: #EBF9E0;
}

.orders-header {
  margin-bottom: 2rem;
  background: #dff6be;
  padding: 1.5rem;
  border-radius: 16px;
  box-shadow: var(--shadow-sm);
}

.orders-header h1 {
  color: #333333;
  margin: 0;
  font-size: 1.75rem;
  font-weight: 600;
}

.orders-list {
  display: grid;
  gap: 1.5rem;
}

.order-card {
  background: #dff6be;
  border-radius: 16px;
  padding: 1.5rem;
  box-shadow: var(--shadow-sm);
  transition: all 0.3s ease;
}

.order-card:hover {
  transform: translateY(-4px);
  box-shadow: var(--shadow-md);
}

.order-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid var(--border-dark);
}

.order-number {
  color: #333333;
  font-size: 1.2rem;
  font-weight: 600;
}

.order-date {
  color: #666666;
  font-size: 0.9rem;
}

.order-status {
  padding: 0.5rem 1rem;
  border-radius: 20px;
  font-size: 0.9rem;
  font-weight: 500;
}

.status-pending {
  background: rgba(44, 118, 59, 0.1);
  color: #2C763B;
}

.status-completed {
  background: rgba(44, 118, 59, 0.2);
  color: #2C763B;
}

.status-cancelled {
  background: rgba(215, 84, 128, 0.1);
  color: #D75480;
}

.order-items {
  display: grid;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.order-item {
  display: flex;
  gap: 1rem;
  padding: 1rem;
  background: #EBF9E0;
  border-radius: 12px;
}

.item-image {
  width: 80px;
  height: 80px;
  border-radius: 8px;
  overflow: hidden;
}

.item-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.item-details {
  flex: 1;
}

.item-name {
  color: #333333;
  font-weight: 500;
  margin: 0 0 0.5rem;
}

.item-price {
  color: #2C763B;
  font-weight: 600;
}

.item-quantity {
  color: #666666;
  font-size: 0.9rem;
}

.order-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 1rem;
  border-top: 1px solid var(--border-dark);
}

.total-amount {
  color: var(--text-dark);
  font-size: 1.2rem;
  font-weight: 600;
}

.total-amount span {
  color: var(--secondary-color);
}

.order-actions {
  display: flex;
  gap: 1rem;
}

.action-button {
  padding: 0.75rem 1.5rem;
  border-radius: 8px;
  font-weight: 500;
  border: none;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.primary-action {
  background: var(--bg-secondary);
  color: var(--text-light);
}

.primary-action:hover {
  background: var(--hover-secondary);
}

.secondary-action {
  background: var(--bg-neutral);
  color: var(--text-dark);
}

.secondary-action:hover {
  background: var(--hover-primary);
}

.danger-action {
  background: var(--bg-accent);
  color: var(--text-light);
}

.danger-action:hover {
  background: var(--hover-accent);
}

.empty-orders {
  text-align: center;
  padding: 4rem 2rem;
  background: #dff6be;
  border-radius: 16px;
  margin-top: 2rem;
}

.empty-orders i {
  font-size: 3rem;
  color: var(--text-muted);
  margin-bottom: 1rem;
}

.empty-orders h2 {
  color: var(--text-dark);
  margin: 0 0 1rem;
}

.empty-orders p {
  color: var(--text-muted);
  margin: 0 0 2rem;
}

.browse-catalog-btn {
  display: inline-flex;
  align-items: center;
  gap: 0.75rem;
  background: #2C763B;
  color: #FFFFFF;
  padding: 1rem 2rem;
  border-radius: 8px;
  text-decoration: none;
  font-weight: 500;
  transition: all 0.3s ease;
}

.browse-catalog-btn:hover {
  background: #1f5429;
}

@media (max-width: 768px) {
  .orders-container {
    padding: 1rem;
  }

  .orders-header {
    padding: 1rem;
  }

  .order-card {
    padding: 1rem;
  }

  .order-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 1rem;
  }

  .order-item {
    flex-direction: column;
    align-items: center;
    text-align: center;
  }

  .item-image {
    width: 120px;
    height: 120px;
  }

  .order-footer {
    flex-direction: column;
    gap: 1rem;
    text-align: center;
  }

  .order-actions {
    width: 100%;
    flex-direction: column;
  }

  .action-button {
    width: 100%;
    justify-content: center;
  }
}
</style> 