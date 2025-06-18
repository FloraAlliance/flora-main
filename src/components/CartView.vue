<template>
  <div class="cart-container">
    <div class="cart-header">
      <h1>Корзина</h1>
      <router-link to="/catalog" class="browse-catalog">
        <i class="fas fa-arrow-left"></i>
        Продолжить покупки
      </router-link>
    </div>
    
    <!-- Список товаров -->
    <div v-if="cartItems.length > 0" class="cart-content">
      <div class="cart-items">
        <div v-for="item in cartItems" :key="item.id" class="cart-item">
          <div class="item-image-container">
            <img :src="getImageUrl(item.image)" :alt="item.name" class="item-image">
          </div>
          <div class="item-details">
            <div class="item-info">
              <h3 class="item-name">{{ item.name }}</h3>
              <p class="item-price">{{ item.price }} ₽</p>
            </div>
            <div class="item-controls">
              <div class="quantity-controls">
                <button @click="decreaseQuantity(item)" class="quantity-btn" title="Уменьшить количество">
                </button>
                <span class="quantity-value">{{ item.quantity }}</span>
                <button @click="increaseQuantity(item)" class="quantity-btn" title="Увеличить количество">
                </button>
              </div>
              <button class="remove-item" @click="removeItem(item)">
                <i class="fas fa-trash-alt"></i>
              </button>
            </div>
          </div>
        </div>
      </div>
      
      <div class="cart-summary">
        <div class="summary-content">
          <div class="summary-row">
            <span>Количество товаров:</span>
            <span>{{ cartItems.length }}</span>
          </div>
          <div class="summary-row total">
            <span>Итого к оплате:</span>
            <span class="total-price">{{ totalPrice }} ₽</span>
          </div>
          <button class="checkout-button" @click="createOrder">
            <i class="fas fa-shopping-cart"></i>
            Оформить заказ
          </button>
        </div>
      </div>
    </div>
    
    <!-- Пустая корзина -->
    <div v-else class="empty-cart">
      <div class="empty-cart-content">
        <i class="fas fa-shopping-cart"></i>
        <h2>Ваша корзина пуста</h2>
        <p>Добавьте товары из каталога</p>
        <router-link to="/catalog" class="continue-shopping">
          Перейти к покупкам
        </router-link>
      </div>
    </div>

    <!-- Модальное окно с кодом заказа -->
    <div v-if="showOrderCode" class="order-modal">
      <div class="order-modal-content">
        <div class="modal-header">
          <div class="success-icon">
            <i class="fas fa-check-circle"></i>
          </div>
          <h2>Заказ успешно создан!</h2>
          <button class="close-modal-btn" @click="closeOrderModal">
            <i class="fas fa-times"></i>
          </button>
        </div>
        <div class="modal-body">
          <div class="order-code">
            <p class="order-info">Код для получения заказа:</p>
            <div class="code-container">
              <span class="code">{{ orderCode }}</span>
              <button class="copy-btn" @click="copyCode" title="Копировать код">
              </button>
            </div>
          </div>
          <div class="order-details">
            <h3>Информация о получении:</h3>
            <div class="details-grid">
              <div class="detail-item">
                <div class="detail-icon" title="Адрес получения"></div>
                <div class="detail-text">
                  <p class="detail-label">Адрес получения</p>
                  <p class="detail-value">г.Курск, пр-кт. Кулакова, Д-1-В</p>
                </div>
              </div>
              <div class="detail-item">
                <div class="detail-icon" title="Способ оплаты"></div>
                <div class="detail-text">
                  <p class="detail-label">Способ оплаты</p>
                  <p class="detail-value">Оплата при получении</p>
                </div>
              </div>
              <div class="detail-item">
                <div class="detail-icon" title="Статус заказа"></div>
                <div class="detail-text">
                  <p class="detail-label">Статус заказа</p>
                  <p class="detail-value">Код заказа сохранен в вашем профиле</p>
                </div>
              </div>
            </div>
          </div>
          <div class="modal-footer">
            <router-link to="/profile" class="view-order-btn">
              <i class="fas fa-list-alt"></i>
              Перейти к заказам
            </router-link>
            <button class="continue-shopping-btn" @click="closeOrderModal">
              <i class="fas fa-shopping-cart"></i>
              Продолжить покупки
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useStore } from 'vuex'
import axios from 'axios'

const store = useStore()
const showOrderCode = ref(false)
const orderCode = ref('')
const baseUrl = 'http://localhost:8000'

const cartItems = computed(() => store.state.cart.items)
const totalPrice = computed(() => store.getters['cart/totalPrice'])

const getImageUrl = (imagePath) => {
  if (!imagePath) return require('@/assets/img/no-image.webp')
  if (imagePath.startsWith('http')) return imagePath
  return `${baseUrl}${imagePath}`
}

const increaseQuantity = (item) => {
  store.commit('cart/updateItemQuantity', {
    id: item.id,
    quantity: item.quantity + 1
  })
}

const decreaseQuantity = (item) => {
  if (item.quantity > 1) {
    store.commit('cart/updateItemQuantity', {
      id: item.id,
      quantity: item.quantity - 1
    })
  }
}

const removeItem = (item) => {
  store.commit('cart/removeItem', item.id)
}

const createOrder = async () => {
  try {
    const response = await axios.post(`${baseUrl}/api/orders/`, {
      items: cartItems.value.map(item => ({
        product_id: item.id,
        quantity: item.quantity
      }))
    })
    
    orderCode.value = response.data.code
    showOrderCode.value = true
    store.commit('cart/clearCart')
  } catch (error) {
    console.error('Error creating order:', error)
    // Здесь можно добавить обработку ошибок
  }
}

const closeOrderModal = () => {
  showOrderCode.value = false
  orderCode.value = ''
}

const copyCode = () => {
  navigator.clipboard.writeText(orderCode.value)
    .then(() => {
      // Можно добавить уведомление об успешном копировании
      alert('Код скопирован в буфер обмена')
    })
    .catch(err => {
      console.error('Ошибка при копировании:', err)
    })
}
</script>

<style scoped>
.cart-container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 2rem;
  min-height: 100vh;
  background: #EBF9E0;
}

.cart-header {
  margin-bottom: 2rem;
  background: #dff6be;
  padding: 1.5rem;
  border-radius: 16px;
  box-shadow: var(--shadow-sm);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.cart-header h1 {
  color: #333333;
  margin: 0;
  font-size: 1.75rem;
  font-weight: 600;
}

.browse-catalog {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  color: #333333;
  text-decoration: none;
  padding: 0.75rem 1.5rem;
  background: #EBF9E0;
  border-radius: 8px;
  transition: all 0.3s ease;
}

.browse-catalog:hover {
  background: #d9e8d0;
  transform: translateX(-2px);
}

.cart-content {
  display: grid;
  grid-template-columns: 1fr 300px;
  gap: 2rem;
}

.cart-items {
  background: #dff6be;
  border-radius: 16px;
  padding: 1.5rem;
}

.cart-item {
  display: flex;
  gap: 1.5rem;
  padding: 1.5rem;
  border-bottom: 1px solid var(--border-dark);
}

.cart-item:last-child {
  border-bottom: none;
}

.item-image-container {
  width: 120px;
  height: 120px;
  border-radius: 12px;
  overflow: hidden;
}

.item-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.item-details {
  flex: 1;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.item-name {
  color: var(--text-dark);
  font-size: 1.1rem;
  margin: 0 0 0.5rem;
}

.item-price {
  color: #2C763B;
  font-size: 1.25rem;
  font-weight: 600;
  margin: 0;
}

.quantity-controls {
  display: flex;
  align-items: center;
  gap: 1rem;
  background: var(--bg-primary);
  padding: 0.5rem;
  border-radius: 8px;
}

.quantity-btn {
  background: #2C763B;
  color: #FFFFFF;
  border: none;
  width: 32px;
  height: 32px;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
}

.quantity-btn:first-child::before {
  content: '';
  position: absolute;
  width: 12px;
  height: 2px;
  background-color: #FFFFFF;
  border-radius: 1px;
}

.quantity-btn:last-child::before {
  content: '';
  position: absolute;
  width: 12px;
  height: 2px;
  background-color: #FFFFFF;
  border-radius: 1px;
}

.quantity-btn:last-child::after {
  content: '';
  position: absolute;
  width: 2px;
  height: 12px;
  background-color: #FFFFFF;
  border-radius: 1px;
}

.quantity-btn:hover {
  background: #1f5429;
  transform: scale(1.1);
}

.quantity-btn:active {
  transform: scale(0.95);
}

.quantity-value {
  font-weight: 600;
  color: var(--text-dark);
  min-width: 40px;
  text-align: center;
}

.remove-item {
  background: #D75480;
  color: #FFFFFF;
  border: none;
  width: 32px;
  height: 32px;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  margin-left: 1rem;
  position: relative;
}

.remove-item::before {
  content: '';
  position: absolute;
  width: 20px;
  height: 20px;
  background-image: url('@/assets/img/remove.png');
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  filter: brightness(0) invert(1);
}

.remove-item:hover {
  background: #c44670;
  transform: scale(1.1);
}

.cart-summary {
  background: #dff6be;
  border-radius: 16px;
  padding: 1.5rem;
  height: fit-content;
}

.summary-content {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.summary-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 0;
  color: var(--text-dark);
}

.summary-row.total {
  border-top: 1px solid var(--border-dark);
  margin-top: 1rem;
  padding-top: 1.5rem;
}

.total-price {
  color: #2C763B;
  font-size: 1.5rem;
  font-weight: 600;
}

.checkout-button {
  width: 100%;
  background: #2C763B;
  color: #FFFFFF;
  border: none;
  padding: 1rem;
  border-radius: 8px;
  font-size: 1.1rem;
  font-weight: 500;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  cursor: pointer;
  transition: all 0.3s ease;
  margin-top: 1.5rem;
}

.checkout-button:hover {
  background: #1f5429;
}

.empty-cart {
  background: #dff6be;
  border-radius: 16px;
  padding: 4rem 2rem;
  text-align: center;
  margin-top: 2rem;
}

.empty-cart i {
  font-size: 4rem;
  color: var(--text-muted);
  margin-bottom: 1.5rem;
}

.empty-cart h2 {
  color: var(--text-dark);
  margin: 0 0 1rem;
}

.empty-cart p {
  color: var(--text-muted);
  margin: 0 0 2rem;
}

.continue-shopping {
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

.continue-shopping:hover {
  background: #1f5429;
}

/* Модальное окно */
.order-modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 2rem;
  z-index: 1000;
}

.order-modal-content {
  background: #EBF9E0;
  border-radius: 16px;
  width: 100%;
  max-width: 600px;
  position: relative;
  box-shadow: 0 10px 30px rgba(44, 118, 59, 0.2);
  border: 2px solid #2C763B;
}

.modal-header {
  background: #dff6be;
  padding: 2rem;
  border-radius: 16px 16px 0 0;
  text-align: center;
  position: relative;
  border-bottom: 2px solid #2C763B;
}

.success-icon {
  width: 64px;
  height: 64px;
  background: #2C763B;
  color: #EBF9E0;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 1rem;
  position: relative;
}

.success-icon::before {
  content: '';
  position: absolute;
  width: 32px;
  height: 32px;
  background-image: url('@/assets/img/email.png');
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  filter: brightness(0) invert(1);
}

.success-icon i {
  display: none;
}

.close-modal-btn {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: none;
  border: none;
  color: #2C763B;
  cursor: pointer;
  padding: 0.5rem;
  transition: all 0.3s ease;
}

.close-modal-btn:hover {
  color: #1f5429;
}

.modal-body {
  padding: 2rem;
  background: #EBF9E0;
}

.order-code {
  background: #dff6be;
  padding: 1.5rem;
  border-radius: 12px;
  margin-bottom: 2rem;
  border: 1px solid #2C763B;
}

.code-container {
  display: flex;
  align-items: center;
  gap: 1rem;
  background: #EBF9E0;
  padding: 1rem;
  border-radius: 8px;
  margin-top: 1rem;
  border: 1px solid #2C763B;
}

.code {
  font-size: 1.5rem;
  font-weight: 600;
  color: #2C763B;
  letter-spacing: 2px;
}

.copy-btn {
  background: #2C763B;
  color: #EBF9E0;
  border: none;
  width: 36px;
  height: 36px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
}

.copy-btn::before {
  content: '';
  position: absolute;
  width: 24px;
  height: 24px;
  background-image: url('@/assets/img/add.png');
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  filter: brightness(0) invert(1);
}

.copy-btn:hover {
  background: #1f5429;
  transform: scale(1.1);
}

.copy-btn:active {
  transform: scale(0.95);
}

.copy-btn i {
  display: none;
}

.details-grid {
  display: grid;
  gap: 1.5rem;
  margin-top: 1.5rem;
}

.detail-item {
  display: flex;
  gap: 1rem;
  padding: 1rem;
  background: #dff6be;
  border-radius: 12px;
  border: 1px solid #2C763B;
}

.detail-icon {
  width: 40px;
  height: 40px;
  background: #2C763B;
  color: #EBF9E0;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

/* Иконка для адреса (первая) */
.detail-item:nth-child(1) .detail-icon::before {
  content: '';
  position: absolute;
  width: 24px;
  height: 24px;
  background-image: url('@/assets/img/home.png');
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  filter: brightness(0) invert(1);
}

/* Иконка для способа оплаты (вторая) */
.detail-item:nth-child(2) .detail-icon::before {
  content: '';
  position: absolute;
  width: 24px;
  height: 24px;
  background-image: url('@/assets/img/shopping.png');
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  filter: brightness(0) invert(1);
}

/* Иконка для статуса заказа (третья) */
.detail-item:nth-child(3) .detail-icon::before {
  content: '';
  position: absolute;
  width: 24px;
  height: 24px;
  background-image: url('@/assets/img/info.png');
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  filter: brightness(0) invert(1);
}

.detail-icon i {
  display: none;
}

.detail-text {
  flex: 1;
}

.detail-label {
  color: #2C763B;
  margin: 0 0 0.25rem;
  font-size: 0.9rem;
}

.detail-value {
  color: #1f5429;
  margin: 0;
  font-weight: 500;
}

.modal-footer {
  display: flex;
  gap: 1rem;
  margin-top: 2rem;
}

.view-order-btn,
.continue-shopping-btn {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  padding: 1rem;
  border-radius: 8px;
  font-weight: 500;
  transition: all 0.3s ease;
  text-decoration: none;
}

.view-order-btn {
  background: #2C763B;
  color: #EBF9E0;
  border: none;
}

.view-order-btn:hover {
  background: #1f5429;
}

.continue-shopping-btn {
  background: #dff6be;
  color: #2C763B;
  border: 1px solid #2C763B;
}

.continue-shopping-btn:hover {
  background: #EBF9E0;
}

@media (max-width: 768px) {
  .cart-container {
    padding: 1rem;
  }

  .cart-content {
    grid-template-columns: 1fr;
  }

  .cart-item {
    flex-direction: column;
    align-items: center;
    text-align: center;
  }

  .item-details {
    flex-direction: column;
    gap: 1rem;
  }

  .item-controls {
    width: 100%;
    display: flex;
    justify-content: center;
    gap: 1rem;
  }

  .modal-footer {
    flex-direction: column;
  }
}
</style> 