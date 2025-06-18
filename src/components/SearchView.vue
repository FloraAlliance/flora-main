<template>
  <div class="search-view-container">
    <!-- Лоадер -->
    <div v-if="isLoading" class="search-view-loader-overlay">
      <div class="search-view-loader"></div>
    </div>

    <!-- Контейнер товаров -->
    <div v-if="!isLoading" class="search-view-products">
      <!-- Карточка товара -->
      <div 
        v-for="(product, index) in displayedProducts"
        :key="product.id"
        class="search-view-card"
        :id="'product-card_' + product.id"
        :style="{ '--delay': index * 0.03 + 's' }"
        :class="{ 'animate-in': visibleProductIds.includes('product-card_' + product.id) }"
        @click="goToProductDetail(product.id)"
      >
        <img 
          :src="getImageUrl(product.image)"
          :alt="product.name"
          class="search-view-image"
          @error="handleImageError"
        >
        <div class="search-view-info">
          <h3 class="search-view-name">{{ product.name }}</h3>
          <p class="search-view-price">{{ product.price }} ₽</p>
          <p class="search-view-article">Артикул: {{ product.article }}</p>
          <p v-if="product.category" class="search-view-category">
            {{ product.category.name }}
          </p>
        </div>
      </div>

      <!-- Сообщение если ничего не найдено -->
      <div v-if="searchQuery && displayedProducts.length === 0" class="search-view-empty">
        Товары по запросу "{{ searchQuery }}" не найдены
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
const baseUrl = 'https://floraalliance.pythonanywhere.com'

// Реактивные данные
const products = ref([])
const categories = ref([])
const searchQuery = ref(route.query.q || '')
const isLoading = ref(true)
const visibleProductIds = ref([])

// Watcher для параметра q в URL
watch(() => route.query.q, (newQuery) => {
  searchQuery.value = newQuery || ''
})

// Получение всех товаров и категорий
const fetchData = async () => {
  try {
    isLoading.value = true
    const [productsRes, categoriesRes] = await Promise.all([
      axios.get(`${baseUrl}/api/products/`),
      axios.get(`${baseUrl}/api/categories/`)
    ])
    products.value = productsRes.data
    categories.value = categoriesRes.data
  } catch (err) {
    console.error('Ошибка загрузки данных:', err)
  } finally {
    isLoading.value = false
    nextTick(() => {
      initProductsObserver()
    })
  }
}

// Функция для нормализации строки
const normalizeString = (str) => {
  return str.toLowerCase()
    .replace(/[^\wа-яё\d\s]/g, '') // Сохраняем пробелы для разделения слов
    .trim()
}

// Функция для разделения строки на слова и поиска
const fullTextSearch = (productName, query) => {
  if (!query.trim()) return false
  
  const normalizedProduct = normalizeString(productName)
  const normalizedQuery = normalizeString(query)
  
  // Разбиваем запрос на отдельные слова
  const queryWords = normalizedQuery.split(/\s+/).filter(word => word.length > 0)
  
  // Если запрос состоит из одного слова - ищем его вхождение
  if (queryWords.length === 1) {
    return normalizedProduct.includes(queryWords[0])
  }
  
  // Для нескольких слов ищем товары, содержащие все слова запроса
  // (можно изменить на some() если нужно, чтобы совпадало хотя бы одно слово)
  return queryWords.every(word => normalizedProduct.includes(word))
  
  // Альтернативный вариант - поиск по точному совпадению фразы
  // return normalizedProduct.includes(normalizedQuery)
}

// Отображаемые товары
const displayedProducts = computed(() => {
  if (!searchQuery.value.trim()) {
    return products.value
  }
  
  // Разбиваем поисковый запрос на отдельные фразы (если разделены запятыми)
  const searchPhrases = searchQuery.value.split(',')
    .map(phrase => phrase.trim())
    .filter(phrase => phrase.length > 0)
  
  return products.value.filter(product => {
    // Ищем товары, которые соответствуют хотя бы одной из поисковых фраз
    return searchPhrases.some(phrase => 
      fullTextSearch(product.name, phrase)
    )
  })
})

const goToProductDetail = (productId) => {
  router.push(`/product/${productId}`)
}

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

  displayedProducts.value.forEach(product => {
    const element = document.getElementById(`product-card_${product.id}`)
    if (element) {
      observer.observe(element)
    }
  })
}

onMounted(() => {
  fetchData()
})
</script>

<style>
.search-view-container {
  padding: 20px;
  min-height: 100vh;
  background: #f7fafc;
}

.search-view-products {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 30px;
  padding: 20px 0;
}

.search-view-card {
  background: white;
  border-radius: 12px;
  overflow: hidden;
  transition: all 0.3s ease;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  opacity: 0;
  transform: translateY(20px);
  animation: searchViewFadeIn 0.5s ease forwards;
  animation-delay: var(--delay);
}

.search-view-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.search-view-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
}

.search-view-info {
  padding: 20px;
}

.search-view-name {
  font-size: 18px;
  font-weight: 600;
  color: #1a202c;
  margin-bottom: 10px;
}

.search-view-price {
  font-size: 20px;
  font-weight: 700;
  color: #6366f1;
  margin-bottom: 8px;
}

.search-view-article {
  font-size: 14px;
  color: #64748b;
  margin-bottom: 8px;
}

.search-view-category {
  font-size: 14px;
  color: #6366f1;
  background: rgba(99, 102, 241, 0.1);
  padding: 4px 8px;
  border-radius: 4px;
  display: inline-block;
}

.search-view-empty {
  grid-column: 1 / -1;
  text-align: center;
  padding: 40px;
  color: #64748b;
  font-size: 18px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.search-view-loader-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.9);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.search-view-loader {
  width: 40px;
  height: 40px;
  border: 3px solid #e2e8f0;
  border-radius: 50%;
  border-top-color: #6366f1;
  animation: searchViewSpin 1s linear infinite;
}

@keyframes searchViewSpin {
  to {
    transform: rotate(360deg);
  }
}

@keyframes searchViewFadeIn {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (max-width: 768px) {
  .search-view-container {
    padding: 10px;
  }

  .search-view-products {
    gap: 15px;
  }

  .search-view-card {
    border-radius: 8px;
  }

  .search-view-image {
    height: 150px;
  }

  .search-view-info {
    padding: 15px;
  }

  .search-view-name {
    font-size: 16px;
  }

  .search-view-price {
    font-size: 18px;
  }
}
</style>