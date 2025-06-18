<template>
    <div class="product-container">
      <!-- Breadcrumbs -->
      <nav class="breadcrumbs">
        <router-link to="/about" class="breadcrumb-link">Все категории</router-link>
        <span class="breadcrumb-separator">/</span>
        <router-link 
          :to="`/about/${product.category?.id || product.category}`" 
          class="breadcrumb-link"
        >
          {{ product.category?.name || `Категория ${product.category}` }}
        </router-link>
        <span class="breadcrumb-separator">/</span>
        <span class="breadcrumb-current">{{ product.name }}</span>
      </nav>
  
      <!-- Main Content -->
      <div v-if="isLoading" class="loader-overlay">
        <div class="loader"></div>
      </div>
  
      <div v-else-if="error" class="error-message">
        {{ error }}
        <button @click="fetchProduct" class="retry-button">Повторить попытку</button>
      </div>
  
      <div v-else class="product-content">
        <!-- Product Images -->
        <div class="product-gallery">
          <img 
            :src="mainImage || getImageUrl(product.image)" 
            :alt="product.name"
            class="main-image"
          >
          <div v-if="product.images && product.images.length > 1" class="thumbnail-grid">
            <img
              v-for="(img, index) in product.images"
              :key="index"
              :src="getImageUrl(img)"
              @click="mainImage = getImageUrl(img)"
              :class="['thumbnail', { active: mainImage === getImageUrl(img) }]"
            >
          </div>
        </div>
  
        <!-- Product Info -->
        <div class="product-info">
          <h1 class="product-title">{{ product.name }}</h1>
          <p class="product-price">{{ product.price }} ₽</p>
          <p class="product-article">Артикул: {{ product.article }}</p>
  
          <div class="product-description" v-html="product.description"></div>
  
          <div class="product-actions">
            <button class="add-to-cart" @click="addToCart">Добавить в корзину</button>
            <button class="buy-now" @click="buyNow">Купить сейчас</button>
          </div>
  
          <div class="product-meta">
            <div class="meta-item" v-if="product.availability">
              <span class="meta-label">Наличие:</span>
              <span class="meta-value">{{ product.availability }}</span>
            </div>
            <div class="meta-item" v-if="product.brand">
              <span class="meta-label">Бренд:</span>
              <span class="meta-value">{{ product.brand }}</span>
            </div>
            <div class="meta-item" v-if="product.weight">
              <span class="meta-label">Вес:</span>
              <span class="meta-value">{{ product.weight }} г</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from 'vue'
  import { useRoute, useRouter } from 'vue-router'
  import axios from 'axios'
  
  const route = useRoute()
  const router = useRouter()
  const product = ref({})
  const isLoading = ref(true)
  const error = ref(null)
  const mainImage = ref(null)
  const baseUrl = 'http://localhost:8000'
  
  const fetchProduct = async () => {
    try {
      isLoading.value = true
      const response = await axios.get(`${baseUrl}/api/products/${route.params.productId}/`)
      product.value = response.data
      mainImage.value = getImageUrl(product.value.image)
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
  
  const addToCart = () => {
    // Implement cart functionality
    console.log('Added to cart:', product.value.id)
    router.push('/cart')  // Example of using router
  }
  
  const buyNow = () => {
    // Implement quick purchase
    console.log('Buy now:', product.value.id)
    router.push('/checkout')  // Example of using router
  }
  
  onMounted(() => {
    fetchProduct()
  })
  </script>
  
  <style scoped>
  .product-container {
    max-width: 1400px;
    margin: 0 auto;
    padding: 2rem;
    min-height: 100vh;
    background: #EBF9E0;
  }
  
  .product-header {
    margin-bottom: 2rem;
    background: #dff6be;
    padding: 1.5rem;
    border-radius: 16px;
    box-shadow: var(--shadow-sm);
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .back-button {
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
  
  .back-button:hover {
    background: #d9e8d0;
    transform: translateX(-2px);
  }
  
  .product-content {
    display: grid;
    grid-template-columns: 1fr 400px;
    gap: 2rem;
  }
  
  .product-gallery {
    background: #dff6be;
    border-radius: 16px;
    padding: 1.5rem;
    box-shadow: var(--shadow-sm);
  }
  
  .main-image {
    width: 100%;
    height: 400px;
    border-radius: 12px;
    overflow: hidden;
    margin-bottom: 1.5rem;
  }
  
  .main-image img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
  
  .thumbnail-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(80px, 1fr));
    gap: 1rem;
  }
  
  .thumbnail {
    width: 100%;
    padding-top: 100%;
    position: relative;
    border-radius: 8px;
    overflow: hidden;
    cursor: pointer;
    transition: all 0.3s ease;
  }
  
  .thumbnail:hover {
    transform: translateY(-2px);
  }
  
  .thumbnail.active {
    border: 2px solid var(--secondary-color);
  }
  
  .thumbnail img {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
  
  .product-info {
    background: #dff6be;
    border-radius: 16px;
    padding: 1.5rem;
    box-shadow: var(--shadow-sm);
  }
  
  .product-title {
    color: #333333;
    font-size: 1.75rem;
    font-weight: 600;
    margin: 0 0 1rem;
  }
  
  .product-article {
    color: #666666;
    font-size: 0.9rem;
    margin: 0 0 1.5rem;
  }
  
  .product-price {
    color: #2C763B;
    font-size: 2rem;
    font-weight: 600;
    margin: 0 0 1.5rem;
  }
  
  .quantity-controls {
    display: flex;
    align-items: center;
    gap: 1rem;
    margin-bottom: 1.5rem;
    background: var(--bg-primary);
    padding: 0.75rem;
    border-radius: 8px;
  }
  
  .quantity-btn {
    background: #2C763B;
    color: #FFFFFF;
    border: none;
    width: 36px;
    height: 36px;
    border-radius: 8px;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    transition: all 0.3s ease;
  }
  
  .quantity-btn:hover {
    background: #1f5429;
  }
  
  .quantity-value {
    font-size: 1.1rem;
    font-weight: 600;
    color: var(--text-dark);
    min-width: 40px;
    text-align: center;
  }
  
  .add-to-cart {
    width: 100%;
    padding: 1rem;
    background: #2C763B;
    color: #FFFFFF;
    border: none;
    border-radius: 8px;
    font-size: 1.1rem;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.3s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.75rem;
  }
  
  .add-to-cart:hover {
    background: #1f5429;
  }
  
  .product-description {
    margin-top: 2rem;
    padding-top: 1.5rem;
    border-top: 1px solid var(--border-dark);
  }
  
  .description-title {
    color: var(--text-dark);
    font-size: 1.2rem;
    font-weight: 600;
    margin: 0 0 1rem;
  }
  
  .description-text {
    color: var(--text-muted);
    line-height: 1.6;
    margin: 0;
  }
  
  .specifications {
    margin-top: 2rem;
    padding-top: 1.5rem;
    border-top: 1px solid var(--border-dark);
  }
  
  .specifications-title {
    color: var(--text-dark);
    font-size: 1.2rem;
    font-weight: 600;
    margin: 0 0 1rem;
  }
  
  .spec-list {
    display: grid;
    gap: 0.75rem;
  }
  
  .spec-item {
    display: flex;
    justify-content: space-between;
    padding: 0.75rem;
    background: var(--bg-primary);
    border-radius: 8px;
  }
  
  .spec-label {
    color: var(--text-muted);
  }
  
  .spec-value {
    color: var(--text-dark);
    font-weight: 500;
  }
  
  .product-actions {
    display: flex;
    gap: 15px;
    margin-bottom: 30px;
  }
  
  .buy-now {
    background: #2C763B;
    color: #FFFFFF;
    border: none;
    padding: 1rem;
    border-radius: 8px;
    font-size: 1.1rem;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.3s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.75rem;
  }
  
  .buy-now:hover {
    background: #1f5429;
  }
  
  .product-meta {
    border-top: 1px solid var(--border-dark);
    padding-top: 20px;
  }
  
  .meta-item {
    margin-bottom: 10px;
    display: flex;
  }
  
  .meta-label {
    font-weight: bold;
    width: 100px;
    color: var(--text-muted);
  }
  
  .meta-value {
    flex: 1;
    color: var(--text-dark);
  }
  
  /* Shared styles from catalog (loader, error message, breadcrumbs) */
  .loader-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.219);
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    z-index: 1000;
  }
  
  .loader {
    border: 5px solid var(--border-light);
    border-top: 5px solid var(--text-dark);
    border-radius: 50%;
    width: 50px;
    height: 50px;
    animation: spin 1s linear infinite;
    margin-bottom: 15px;
  }
  
  .error-message {
    background: #D75480;
    padding: 15px;
    border-radius: 8px;
    margin-bottom: 20px;
    color: #FFFFFF;
    text-align: center;
  }
  
  .retry-button {
    background: #2C763B;
    color: #FFFFFF;
    border: none;
    padding: 8px 16px;
    border-radius: 4px;
    margin-top: 10px;
    cursor: pointer;
    transition: background 0.3s;
  }
  
  .retry-button:hover {
    background: #1f5429;
  }
  
  .breadcrumbs {
    margin-bottom: 20px;
    font-size: 16px;
    display: flex;
    align-items: center;
  }
  
  .breadcrumb-link {
    color: #2C763B;
    text-decoration: none;
    transition: color 0.3s;
  }
  
  .breadcrumb-link:hover {
    color: #1f5429;
    text-decoration: underline;
  }
  
  .breadcrumb-separator {
    margin: 0 8px;
    color: #666666;
  }
  
  .breadcrumb-current {
    color: #333333;
    font-weight: 500;
  }
  
  @keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
  }
  
  @media (max-width: 1024px) {
    .product-content {
      grid-template-columns: 1fr;
    }
  }
  
  @media (max-width: 768px) {
    .product-container {
      padding: 1rem;
    }
  
    .product-header {
      padding: 1rem;
    }
  
    .main-image {
      height: 300px;
    }
  
    .product-title {
      font-size: 1.5rem;
    }
  
    .product-price {
      font-size: 1.75rem;
    }
  
    .quantity-controls {
      justify-content: center;
    }
  }
  </style>