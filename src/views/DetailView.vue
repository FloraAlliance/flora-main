<template>
  <div class="detail-container fade-in">
    <!-- Хлебные крошки -->
    <nav class="breadcrumbs">
      <router-link to="/catalog" class="breadcrumb-link">Все категории</router-link>
      <span class="breadcrumb-separator">/</span>
      <router-link 
        :to="`/catalog/${product.category?.id || product.category}`" 
        class="breadcrumb-link"
      >
        {{ product.category?.name || `Категория ${product.category}` }}
      </router-link>
      <span class="breadcrumb-separator">/</span>
      <span class="breadcrumb-current">{{ product.name }}</span>
    </nav>

    <!-- Лоадер -->
    <div v-if="isLoading" class="loading-overlay">
      <div class="spinner"></div>
      <span>Загрузка товара...</span>
    </div>

   <!-- Основное содержимое -->
    <div v-if="!isLoading && product" class="product-detail">
      <div class="product-gallery">
        <div class="image-container" @click="openModal(mainImage || product.image)">
          <!-- Размытый фон -->
          <img 
            :src="getImageUrl(mainImage || product.image)" 
            :alt="product.name + ' background'"
            class="image-background"
            @error="handleImageError"
          >
          <!-- Основное изображение -->
          <img 
            :src="getImageUrl(mainImage || product.image)" 
            :alt="product.name" 
            class="main-image"
            @error="handleImageError"
          >
        </div>
        
        <div v-if="product.images && product.images.length > 1" class="thumbnail-container">
          <img
            v-for="(img, index) in product.images"
            :key="index"
            :src="getImageUrl(img)"
            @click="mainImage = img"
            class="thumbnail"
            :class="{ 'active-thumbnail': mainImage === img }"
          >
        </div>
      </div>

      <div class="product-info">
        <h1 class="product-title">{{ product.name }}</h1>
        <p class="product-price">{{ product.price }} ₽</p>
        <p class="product-article">Артикул: {{ product.article }}</p>

        <div v-if="product.description" class="product-description">
          <h3>Описание:</h3>
          <p>{{ product.description }}</p>
        </div>

        <div v-if="product.specifications" class="product-specs">
          <h3>Характеристики:</h3>
          <p>{{ product.specifications }}</p>
        </div>

        <button 
          class="add-to-cart" 
          :class="{ 'in-cart': isInCart }"
          @click="toggleCart"
        >
          <span>{{ isInCart ? 'В корзине' : 'Добавить в корзину' }}</span>
        </button>
      </div>
    </div>

    
    <div v-if="error" class="error-message">
      {{ error }}
      <button @click="fetchProduct" class="retry-button">Повторить попытку</button>
    </div>

    <div v-if="showImageModal" class="image-modal-overlay" @click.self="closeModal">
      <div class="image-modal-content">
        <button class="close-modal-btn" @click="closeModal">×</button>
        <img 
          :src="getImageUrl(modalImageUrl)" 
          :alt="'Увеличенное изображение ' + product.name"
          class="modal-image"
          @error="handleModalImageError"
        >
      </div>
    </div>
  </div>
  
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useStore } from 'vuex'
import { useAuthStore } from '@/store/auth'
import axios from 'axios'

const route = useRoute()
const router = useRouter()
const store = useStore()
const authStore = useAuthStore()
const product = ref({})
const isLoading = ref(true)
const error = ref(null)
const mainImage = ref(null)
const baseUrl = 'http://localhost:8000'

// Добавляем состояния для модального окна
const showImageModal = ref(false)
const modalImageUrl = ref('')

// Проверяем, находится ли товар в корзине
const isInCart = computed(() => {
  return store.state.cart.items.some(item => item.id === product.value.id)
})

const productId = computed(() => parseInt(route.params.productId))

const fetchProduct = async () => {
  try {
    isLoading.value = true
    const response = await axios.get(`${baseUrl}/api/products/${productId.value}/`)
    product.value = response.data
    mainImage.value = product.value.image
  } catch (err) {
    error.value = 'Ошибка загрузки товара'
    console.error('Fetch error:', err)
  } finally {
    isLoading.value = false
  }
}

const getImageUrl = (imagePath) => {
  if (!imagePath) return require('@/assets/img/no-image.webp')
  if (imagePath.startsWith('http')) return imagePath
  return `${baseUrl}${imagePath}`
}

const handleImageError = (e) => {
  e.target.src = require('@/assets/img/no-image.webp')
}

// Функции для работы с модальным окном
const openModal = (imgUrl) => {
  modalImageUrl.value = imgUrl
  showImageModal.value = true
  document.body.style.overflow = 'hidden' // Блокируем скролл страницы
}

const closeModal = () => {
  showImageModal.value = false
  document.body.style.overflow = '' // Восстанавливаем скролл
}

const handleModalImageError = (e) => {
  e.target.src = require('@/assets/img/no-image.webp')
}

// Функция для добавления/удаления товара из корзины
const toggleCart = () => {
  if (!authStore.isAuthenticated) {
    // Если пользователь не авторизован, перенаправляем на страницу регистрации
    router.push('/register')
    return
  }

  if (isInCart.value) {
    store.commit('cart/removeItem', product.value.id)
  } else {
    store.commit('cart/addItem', {
      id: product.value.id,
      name: product.value.name,
      price: product.value.price,
      image: product.value.image
    })
  }
}

onMounted(() => {
  fetchProduct()
})
</script>

<style scoped>
/* Стили для модального окна */
.image-modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.95);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  backdrop-filter: blur(10px);
  animation: fadeIn 0.3s ease-out;
}

.image-modal-content {
  position: relative;
  max-width: 95vw;
  max-height: 95vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-image {
  max-height: 95vh;
  max-width: 95vw;
  object-fit: contain;
  border-radius: 8px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
  animation: zoomIn 0.3s ease-out;
}

.close-modal-btn {
  position: absolute;
  top: -3rem;
  right: -3rem;
  background: rgba(255, 255, 255, 0.1);
  color: white;
  border: none;
  width: 3rem;
  height: 3rem;
  border-radius: 50%;
  font-size: 1.5rem;
  cursor: pointer;
  transition: all 0.3s ease;
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
}

.close-modal-btn:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: rotate(90deg);
}

/* Основные стили */
.detail-container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 2rem;
  min-height: 100vh;
  background: #EBF9E0;
}

.fade-in {
  animation: fadeIn 0.5s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* Хлебные крошки */
.breadcrumbs {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 30px;
  flex-wrap: wrap;
}

.breadcrumb-link {
  color: #2C763B;
  text-decoration: none;
  font-size: 14px;
  transition: color 0.3s;
}

.breadcrumb-link:hover {
  color: #1f5429;
}

.breadcrumb-separator {
  color: #2C763B;
  opacity: 0.5;
}

.breadcrumb-current {
  color: #2C763B;
  font-weight: 500;
}

/* Основной контент */
.product-detail {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 3rem;
  margin-top: 2rem;
  background: #dff6be;
  border-radius: 24px;
  padding: 2rem;
  box-shadow: 0 8px 32px rgba(44, 118, 59, 0.1);
  border: 2px solid #2C763B;
}

/* Галерея изображений */
.product-gallery {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.image-container {
  position: relative;
  width: 100%;
  padding-top: 100%;
  border-radius: 20px;
  overflow: hidden;
  cursor: zoom-in;
  background: #EBF9E0;
  box-shadow: 0 4px 20px rgba(44, 118, 59, 0.1);
  transition: transform 0.3s ease;
  border: 2px solid #2C763B;
}

.image-container:hover {
  transform: scale(1.02);
}

.image-background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  filter: blur(20px);
  opacity: 0.3;
  transform: scale(1.2);
}

.main-image {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: contain;
  z-index: 1;
  transition: transform 0.3s ease;
}

.thumbnail-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(80px, 1fr));
  gap: 1rem;
  padding: 0.5rem;
  background: #EBF9E0;
  border-radius: 16px;
  border: 1px solid #2C763B;
}

.thumbnail {
  width: 100%;
  aspect-ratio: 1;
  object-fit: cover;
  border-radius: 12px;
  cursor: pointer;
  opacity: 0.6;
  transition: all 0.3s ease;
  border: 2px solid transparent;
}

.thumbnail:hover {
  opacity: 1;
  transform: translateY(-2px);
}

.active-thumbnail {
  opacity: 1;
  border-color: #2C763B;
  box-shadow: 0 0 12px rgba(44, 118, 59, 0.3);
}

/* Информация о товаре */
.product-info {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  padding: 1rem;
}

.product-title {
  font-size: 2.5rem;
  color: #2C763B;
  font-weight: 600;
  line-height: 1.2;
  margin: 0;
  font-family: 'Manrope', sans-serif;
}

.product-price {
  font-size: 2rem;
  color: #2C763B;
  font-weight: 700;
  margin: 0;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.product-price::after {
  content: '₽';
  font-size: 1.5rem;
  opacity: 0.8;
}

.product-article {
  color: #1f5429;
  font-size: 0.9rem;
  margin: 0;
  padding: 0.5rem 1rem;
  background: #EBF9E0;
  border-radius: 8px;
  display: inline-block;
  border: 1px solid #2C763B;
}

.product-description,
.product-specs {
  background: #EBF9E0;
  padding: 1.5rem;
  border-radius: 16px;
  border: 1px solid #2C763B;
}

.product-description h3,
.product-specs h3 {
  color: #2C763B;
  font-size: 1.2rem;
  margin: 0 0 1rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.product-description h3::before,
.product-specs h3::before {
  content: '';
  display: block;
  width: 4px;
  height: 1.2rem;
  background: #6366f1;
  border-radius: 2px;
}

.product-description p,
.product-specs p {
  color: #1f5429;
  line-height: 1.6;
  margin: 0;
}

/* Кнопки действий */
.product-actions {
  display: flex;
  gap: 20px;
  margin-top: 20px;
}

.add-to-cart,
.buy-now {
  flex: 1;
  padding: 15px 30px;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
}

.add-to-cart {
  margin-top: auto;
  padding: 1rem 2rem;
  border: none;
  border-radius: 12px;
  font-size: 1.1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  background: #2C763B;
  color: #EBF9E0;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  position: relative;
  padding-left: 3.5rem;
}

.add-to-cart::before {
  content: '';
  position: absolute;
  left: 1rem;
  top: 50%;
  transform: translateY(-50%);
  width: 24px;
  height: 24px;
  background-image: url('@/assets/img/12button2.png');
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
}

.add-to-cart:hover {
  background: #1f5429;
  transform: translateY(-2px);
}

.add-to-cart.in-cart {
  background: #dff6be;
  color: #2C763B;
  border: 2px solid #2C763B;
  padding-left: 3.5rem;
}

.add-to-cart.in-cart::before {
  content: '';
  position: absolute;
  left: 1rem;
  top: 50%;
  transform: translateY(-50%);
  width: 24px;
  height: 24px;
  background-image: url('@/assets/img/12button2.png');
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  filter: invert(1);
}

.add-to-cart.in-cart:hover {
  background: #EBF9E0;
}

/* Лоадер */
.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 20px;
  color: #DEDEDE;
  z-index: 1000;
}

.spinner {
  width: 50px;
  height: 50px;
  border: 5px solid #f3f3f3;
  border-top: 5px solid #3498db;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* Сообщение об ошибке */
.error-message {
  text-align: center;
  color: #e74c3c;
  padding: 20px;
}

.retry-button {
  background: #e74c3c;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  margin-top: 10px;
  cursor: pointer;
  transition: background 0.3s;
}

.retry-button:hover {
  background: #c0392b;
}

/* Адаптивность */
@media (max-width: 1024px) {
  .product-detail {
    grid-template-columns: 1fr;
    gap: 2rem;
  }

  .product-title {
    font-size: 2rem;
  }

  .product-price {
    font-size: 1.75rem;
  }
}

@media (max-width: 768px) {
  .detail-container {
    padding: 1rem;
  }

  .product-detail {
    padding: 1.5rem;
    border-radius: 16px;
  }

  .thumbnail-container {
    grid-template-columns: repeat(auto-fit, minmax(60px, 1fr));
  }

  .close-modal-btn {
    top: 1rem;
    right: 1rem;
  }
}

@media (max-width: 480px) {
  .product-title {
    font-size: 1.5rem;
  }

  .product-price {
    font-size: 1.5rem;
  }

  .add-to-cart {
    padding: 0.875rem;
    font-size: 1rem;
  }
}
</style>