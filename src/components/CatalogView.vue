<template>
    <!-- Основной контейнер -->
    <div class="catalog-container">
      <!-- Верхняя панель -->
      <div class="catalog-header">
        <div class="header-left">
          <router-link to="/catalog" class="back-button">
            <i class="fas fa-arrow-left"></i>
            <span>Назад к категориям</span>
          </router-link>
          <h1 class="catalog-title">{{ currentCategoryName }}</h1>
        </div>
      </div>
  
      <!-- Индикатор загрузки -->
      <div v-if="isLoading" class="loading">
        <div class="spinner"></div>
        <span>Загрузка товаров...</span>
      </div>
  
      <!-- Сообщение об ошибке -->
      <div v-if="error" class="error-message">
        <i class="fas fa-exclamation-circle"></i>
        <div class="error-content">
          <p>{{ error }}</p>
          <button @click="fetchProducts" class="retry-button">
            <i class="fas fa-sync-alt"></i>
            Повторить попытку
          </button>
        </div>
      </div>
  
      <!-- Сетка товаров -->
      <div v-if="!isLoading && !error" class="products-grid">
        <div 
          v-for="(product, index) in filteredProducts"
          :key="product.id"
          class="product-card"
          :id="'product-card_' + product.id"
          :style="{ '--delay': index * 0.03 + 's' }"
          :class="{ 'animate-in': visibleProductIds.includes('product-card_' + product.id) }"
          @click="goToProductDetail(product.id)"
        >
          <div class="product-image-container">
            <img 
              :src="getImageUrl(product.image)"
              :alt="product.name"
              class="product-image"
              @error="handleImageError"
            >
            <div class="product-overlay">
              <span class="view-details">
                <i class="fas fa-eye"></i>
                Подробнее
              </span>
            </div>
          </div>
          <div class="product-info">
            <h3 class="product-name">{{ product.name }}</h3>
            <p class="product-article">Артикул: {{ product.article }}</p>
            <p v-if="product.category" class="product-category">
              {{ product.category.name }}
            </p>
            <div class="product-footer">
              <span class="price-value">{{ product.price }} ₽</span>
              <button class="details-button">
                <i class="fas fa-chevron-right"></i>
              </button>
            </div>
          </div>
        </div>
  
        <!-- Сообщение при отсутствии товаров -->
        <div v-if="filteredProducts.length === 0" class="empty-catalog">
          <i class="fas fa-box-open"></i>
          <p>В этой категории пока нет товаров</p>
          <router-link to="/catalog" class="browse-categories">
            Посмотреть другие категории
          </router-link>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, computed, onMounted, nextTick, watch } from 'vue'
  import { useRoute, useRouter } from 'vue-router'
  import axios from 'axios'
  
  const router = useRouter()
  const route = useRoute()
  
  const products = ref([])
  const isLoading = ref(true)
  const error = ref(null)
  const baseUrl = 'https://floraalliance.pythonanywhere.com'
  const visibleProductIds = ref([])
  
  const categoryId = computed(() => parseInt(route.params.categoryId))
  
  const goToProductDetail = (productId) => {
    router.push(`/product/${productId}`)
  }
  
  const fetchProducts = async () => {
    try {
      isLoading.value = true
      const response = await axios.get(`${baseUrl}/api/products/`)
      products.value = response.data
    } catch (err) {
      error.value = 'Ошибка загрузки товаров'
      console.error('Fetch error:', err)
    } finally {
      isLoading.value = false
      nextTick(() => {
        initProductsObserver()
      })
    }
  }
  
  const filteredProducts = computed(() => {
    return products.value.filter(product => {
      const catId = product.category?.id ?? product.category
      return catId == categoryId.value
    })
  })
  
  const currentCategoryName = computed(() => {
    const product = products.value.find(p => {
      const catId = p.category?.id ?? p.category
      return catId == categoryId.value
    })
    return product?.category?.name ?? `Категория ${categoryId.value}`
  })
  
  const getImageUrl = (imagePath) => {
    if (!imagePath) return require('@/assets/img/no-image.webp')
    if (imagePath.startsWith('http')) return imagePath
    return `${baseUrl}${imagePath}`
  }
  
  const handleImageError = (event) => {
    event.target.src = require('@/assets/img/no-image.webp')
  }
  
  const initProductsObserver = () => {
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            const id = entry.target.id
            if (!visibleProductIds.value.includes(id)) {
              visibleProductIds.value.push(id)
            }
          }
        })
      },
      {
        root: null,
        rootMargin: '0px',
        threshold: 0.1,
      }
    )
  
    filteredProducts.value.forEach(product => {
      const element = document.getElementById(`product-card_${product.id}`)
      if (element) {
        observer.observe(element)
      }
    })
  }
  
  onMounted(() => {
    fetchProducts()
  })
  
  watch(filteredProducts, () => {
    nextTick(() => {
      initProductsObserver()
    })
  }, { deep: true })
  </script>
  
  <style>
  .catalog-container {
    max-width: 1400px;
    margin: 0 auto;
    padding: 2rem;
    min-height: 100vh;
    background: #EBF9E0;
  }
  
  .catalog-header {
    margin-bottom: 2rem;
    background: #dff6be;
    padding: 1.5rem;
    border-radius: 16px;
    box-shadow: var(--shadow-sm);
  }
  
  .header-left {
    display: flex;
    align-items: center;
    gap: 1.5rem;
  }
  
  .back-button {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    color: #333333;
    text-decoration: none;
    font-weight: 500;
    transition: all 0.3s ease;
    padding: 0.5rem 1rem;
    border-radius: 8px;
    background: #EBF9E0;
  }
  
  .back-button:hover {
    background: #d9e8d0;
    transform: translateX(-2px);
  }
  
  .catalog-title {
    font-size: 1.75rem;
    color: var(--text-dark);
    margin: 0;
    font-weight: 600;
  }
  
  .loading {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 4rem;
    color: var(--text-muted);
    gap: 1rem;
  }
  
  .spinner {
    width: 40px;
    height: 40px;
    border: 4px solid var(--border-dark);
    border-top-color: var(--secondary-color);
    border-radius: 50%;
    animation: spin 1s linear infinite;
  }
  
  .error-message {
    background: var(--bg-neutral);
    border-radius: 12px;
    padding: 2rem;
    margin: 2rem 0;
    display: flex;
    align-items: center;
    gap: 1.5rem;
    color: var(--accent-color);
  }
  
  .error-message i {
    font-size: 1.75rem;
  }
  
  .error-content {
    flex: 1;
  }
  
  .error-content p {
    margin: 0 0 1rem;
    font-size: 1.1rem;
  }
  
  .retry-button {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    background: var(--bg-secondary);
    color: var(--text-light);
    border: none;
    padding: 0.75rem 1.5rem;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.3s ease;
    margin-top: 1rem;
  }
  
  .retry-button:hover {
    background: var(--hover-secondary);
  }
  
  .products-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 2rem;
    padding: 1rem 0;
  }
  
  .product-card {
    background: #dff6be;
    border: 2px solid #2C763B;
    border-radius: 16px;
    overflow: hidden;
    transition: all 0.3s ease;
    cursor: pointer;
    opacity: 0;
    transform: translateY(20px);
    box-shadow: 0 2px 8px rgba(44, 118, 59, 0.1);
  }
  
  .product-card.animate-in {
    animation: fadeInUp 0.6s ease forwards;
    animation-delay: var(--delay);
  }
  
  .product-card:hover {
    transform: translateY(-4px);
    box-shadow: 0 4px 12px rgba(44, 118, 59, 0.2);
    background: #EBF9E0;
  }
  
  .product-image-container {
    position: relative;
    padding-top: 100%;
    overflow: hidden;
    background: #dff6be;
  }
  
  .product-image {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    transition: transform 0.3s ease;
  }
  
  .product-card:hover .product-image {
    transform: scale(1.05);
  }
  
  .product-overlay {
    position: absolute;
    inset: 0;
    background: rgba(0, 0, 0, 0.3);
    display: flex;
    align-items: center;
    justify-content: center;
    opacity: 0;
    transition: opacity 0.3s ease;
  }
  
  .product-card:hover .product-overlay {
    opacity: 1;
  }
  
  .view-details {
    background: #2C763B;
    color: #FFFFFF;
    padding: 0.75rem 1.5rem;
    border-radius: 8px;
    display: flex;
    align-items: center;
    gap: 0.5rem;
    font-weight: 500;
    transform: translateY(10px);
    transition: all 0.3s ease;
  }
  
  .product-card:hover .view-details {
    transform: translateY(0);
  }
  
  .product-info {
    padding: 1.5rem;
  }
  
  .product-name {
    font-size: 1.1rem;
    color: var(--text-dark);
    margin: 0 0 0.5rem;
    font-weight: 600;
    line-height: 1.4;
  }
  
  .product-article {
    color: var(--text-muted);
    font-size: 0.9rem;
    margin: 0 0 0.5rem;
  }
  
  .product-category {
    font-size: 0.9rem;
    color: #2C763B;
    background: rgba(44, 118, 59, 0.1);
    padding: 4px 8px;
    border-radius: 4px;
    display: inline-block;
    margin: 0 0 1rem;
  }
  
  .product-footer {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
  
  .price-value {
    font-size: 1.25rem;
    font-weight: 600;
    color: #2C763B;
  }
  
  .details-button {
    background: #2C763B;
    color: #FFFFFF;
    border: none;
    width: 36px;
    height: 36px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    transition: all 0.3s ease;
    position: relative;
  }
  
  .details-button::before {
    content: '';
    position: absolute;
    width: 20px;
    height: 20px;
    background-image: url('@/assets/img/12button2.png');
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
    filter: brightness(0) invert(1);
  }
  
  .details-button:hover {
    background: #1f5429;
    transform: translateX(2px);
  }
  
  .empty-catalog {
    grid-column: 1 / -1;
    text-align: center;
    padding: 4rem 2rem;
    color: #666666;
    background: #dff6be;
    border-radius: 16px;
    margin-top: 2rem;
  }
  
  .empty-catalog i {
    font-size: 3rem;
    margin-bottom: 1rem;
  }
  
  .empty-catalog p {
    margin: 0 0 1.5rem;
    font-size: 1.2rem;
    color: #fff;
  }
  
  .browse-categories {
    display: inline-block;
    margin-top: 1rem;
    padding: 0.75rem 1.5rem;
    background: #2C763B;
    color: #FFFFFF;
    border-radius: 8px;
    text-decoration: none;
    transition: all 0.3s ease;
  }
  
  .browse-categories:hover {
    background: #1f5429;
  }
  
  @keyframes fadeInUp {
    from {
      opacity: 0;
      transform: translateY(20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
  
  @keyframes spin {
    to {
      transform: rotate(360deg);
    }
  }
  
  /* Адаптивность */
  @media (max-width: 768px) {
    .catalog-container {
      padding: 1rem;
    }
  
    .catalog-header {
      padding: 1rem;
      margin-bottom: 1.5rem;
    }
  
    .header-left {
      flex-direction: column;
      align-items: flex-start;
      gap: 1rem;
    }
  
    .catalog-title {
      font-size: 1.5rem;
    }
  
    .products-grid {
      gap: 1rem;
      grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
    }
  
    .product-info {
      padding: 1rem;
    }
  }
  </style>