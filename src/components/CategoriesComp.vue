<template>
  <div class="cards-container">
     
    <!-- Общий лоадер -->
     <div v-if="isLoading" class="loader-overlay">
        <div class="loader"></div>
        <p class="card_txt">Загрузка данных...</p>
      </div>

    <router-link
  v-for="(category, index) in categories"
  :key="category.id"
  :to="{ name: 'catalog', params: { categoryId: category.id } }"
  class="card-link"
>
  <div 
    class="card_fr" 
    :id="'card_fr_' + category.id"
    :style="{ '--delay': index * 0.03 + 's' }"
    :class="{ 'animate-in': visibleIds.includes('card_fr_' + category.id) }"
  >
    <div class="card_img_fr">
      <img
        class="card_img"
        :src="getImageUrl(category.image)"
        :alt="category.name"
        @error="handleImageError"
      >
    </div>
    <p class="card_txt" ref="textElements">{{ category.name }}</p>
  </div>
</router-link>
  </div>
</template>

<script setup>
import axios from 'axios'
import { ref, onMounted, nextTick, watch } from 'vue'

// Реактивные данные
const categories = ref([])
const textElements = ref([])
const isLoading = ref(true)
const visibleIds = ref([]) // Добавляем для Intersection Observer

// Константы
const baseUrl = 'https://floraalliance.pythonanywhere.com'

// Методы
const adjustTextSize = () => {
  nextTick(() => {
    textElements.value.forEach(el => {
      if (!el) return
      
      el.style.fontSize = '' // Сброс размера для перерасчета
      const containerHeight = el.clientHeight
      let fontSize = 30
      
      // Проверяем, помещается ли текст
      el.style.fontSize = `${fontSize}px`
      while (el.scrollHeight > containerHeight && fontSize > 12) {
        fontSize -= 1
        el.style.fontSize = `${fontSize}px`
      }
    })
  })
}

const fetchCategories = async () => {
  try {
    isLoading.value = true
    const response = await axios.get(`${baseUrl}/api/categories/`)
    categories.value = response.data
    console.log('Raw products data:', categories.value)
  } catch (error) {
    console.error('Ошибка загрузки категорий:', error)
  } finally {
    isLoading.value = false
  }
}

const getImageUrl = (imagePath) => {
  return imagePath ? `${baseUrl}${imagePath}` : require('@/assets/img/1.webp')
}

const handleImageError = (event) => {
  event.target.src = require('@/assets/img/1.webp')
}

const initIntersectionObserver = () => {
  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          const id = entry.target.id;
          if (!visibleIds.value.includes(id)) {
            visibleIds.value.push(id);
          }
          // Не прекращаем наблюдение, чтобы анимация работала при повторном появлении
        }
      });
    },
    {
      root: null,
      rootMargin: '0px',
      threshold: 0.1,
    }
  );

  // Наблюдаем за всеми карточками
  categories.value.forEach(category => {
    const element = document.getElementById(`card_fr_${category.id}`);
    if (element) {
      observer.observe(element);
    }
  });
};

// Хуки жизненного цикла
onMounted(() => {
  fetchCategories()
  nextTick(() => {
    initIntersectionObserver()
    adjustTextSize()
  })
})

watch(categories, (newCategories) => {
  if (newCategories.length) {
    nextTick(() => {
      initIntersectionObserver();
      adjustTextSize();
    });
  }
}, { deep: true });

// Watcher для категорий
watch(categories, () => {
  nextTick(() => {
    adjustTextSize()
  })
}, { deep: true })
</script>

<style scoped>
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
    border: 5px solid #f3f3f37e;
    border-top: 5px solid #4d4d4d;
    border-radius: 50%;
    width: 50px;
    height: 50px;
    animation: spin 1s linear infinite;
    margin-bottom: 15px;
  }
  
  .cards-container {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
    gap: 1.5rem;
    padding: 20px;
    width: 955px;
}

.card-link {
  text-decoration: none;
  color: inherit;
  margin: 0 !important;
}

.card_img {
  width: 215.41px;
  height: 149px;
  object-fit: cover;
  margin: 0 auto;
  background-color: #dff6be;
  transition: transform 0.3s ease;
}

.card_img_fr {
  box-sizing: border-box;
  width: 220px;
  height: 138px;
  border: 4px solid #2C763B;
  border-radius: 6px;
  margin: -4px 0 0 -4px;
  overflow: hidden;
  transition: box-shadow 0.3s ease;
  background-color: #dff6be;
}

.card_txt {
  width: 248px;
  height: 137px;
  font-family: 'Manrope', sans-serif;
  font-style: normal;
  font-weight: 400;
  font-size: 30px;
  line-height: 1.2;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  color: #2C763B;
  margin-top: -4px;
  transition: text-shadow 0.3s ease;
  overflow: hidden;
  word-break: break-word;
}

.card_fr {
  display: flex;
  box-sizing: border-box;
  width: 100%;
  height: 138px;
  background: #EBF9E0;
  border: 4px solid #2C763B;
  border-radius: 6px;
  cursor: pointer;
  opacity: 0;
  transform: translateY(20px);
  transition: opacity 0.5s ease-out, transform 0.5s ease-out;
}

.card_fr:hover {
  background: #dff6be;
  box-shadow: 0 4px 12px rgba(44, 118, 59, 0.2);
}

.card_fr:hover .card_img {
  transform: scale(1.05);
}

.card_fr:hover .card_txt {
  color: #1f5429;
}

.card_fr.animate-in {
  opacity: 1;
  transform: translateY(0);
  transition-delay: var(--delay);
}
</style>