<template>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=M+PLUS+Rounded+1c&family=Noto+Sans&display=swap" rel="stylesheet">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  
  <div class="page-container">
    <header class="header">
      <nav class="nav-container">
        <!-- Левая группа -->
        <div class="nav-group">
          <router-link to="/" class="nav-item">
            <div class="nav-bg"></div>
            <span class="nav-text">Главная</span>
          </router-link>
          
          <router-link to="/catalog" class="nav-item catalog">
            <div class="nav-bg catalog-bg"></div>
            <span class="nav-text">Каталог</span>
          </router-link>
        </div>

        <!-- Центральная группа -->
        <div class="nav-group search-group">
          <div class="search-container">
            <div class="nav-bg"></div>
            <div class="search-content">
          <input 
    type="text"
                placeholder="Поиск" 
    v-model="searchQuery"
    @focus="redirectToSearch"
    @keyup.enter="performSearch"
  >
              <button class="search-button">
                Поиск
              </button>
            </div>
          </div>
        </div>

        <!-- Правая группа -->
        <div class="nav-group">
          <router-link v-if="!authStore.isAuthenticated" to="/register" class="nav-item profile">
            <div class="nav-bg profile-bg"></div>
            <span class="nav-text">Регистрация</span>
          </router-link>

          <router-link v-if="!authStore.isAuthenticated" to="/login" class="nav-item profile">
            <div class="nav-bg profile-bg"></div>
            <span class="nav-text">Войти</span>
          </router-link>
        
          <router-link v-if="authStore.isAuthenticated" to="/profile" class="nav-item profile">
            <div class="nav-bg profile-bg"></div>
            <span class="nav-text">Профиль</span>
          </router-link>

          <router-link 
      v-if="authStore.isAuthenticated && authStore.isAdmin" 
      to="/admin-panel"
            class="nav-item profile"
    >
            <div class="nav-bg profile-bg"></div>
            <span class="nav-text">Админ-панель</span>
    </router-link>

          <router-link 
      v-if="authStore.isAuthenticated && authStore.isAdmin" 
      to="/admin/orders"
            class="nav-item profile"
          >
            <div class="nav-bg profile-bg"></div>
            <span class="nav-text">Заказы</span>
          </router-link>

          <router-link v-if="authStore.isAuthenticated" to="/cart" class="nav-item cart">
            <div class="nav-bg cart-bg"></div>
            <div class="cart-content">
          <svg 
            class="cart-icon" 
            viewBox="0 0 24 24" 
            fill="none" 
            stroke="currentColor" 
            stroke-width="2"
          >
            <path d="M9 22a1 1 0 100-2 1 1 0 000 2zM20 22a1 1 0 100-2 1 1 0 000 2z" />
            <path d="M1 1h4l2.68 13.39a2 2 0 002 1.61h9.72a2 2 0 002-1.61L23 6H6" />
          </svg>
              <span class="nav-text">Корзина</span>
              <div v-if="cartItemsCount > 0" class="cart-badge">
            {{ cartItemsCount }}
              </div>
        </div>
      </router-link>
        </div>
      </nav>
    </header>

    <router-view></router-view>
  </div>
</template>
<script>
import { computed, ref, watch } from 'vue'
import { useAuthStore } from '@/store/auth.js'
import { useRouter } from 'vue-router'
import { useStore } from 'vuex'

export default {
  name: 'App',
  setup() {
    const router = useRouter()
    const store = useStore()
    const searchQuery = ref('')
    const authStore = useAuthStore()

    const isAuthenticated = computed(() => {
      return !!localStorage.getItem('access_token')
    })

    // Добавляем вычисляемое свойство для количества товаров в корзине
    const cartItemsCount = computed(() => {
      return store.getters['cart/itemCount']
    })

    const redirectToSearch = () => {
      router.push('/catalogs')
    }
    
    const performSearch = () => {
      if (searchQuery.value.trim()) {
        router.push({ path: '/catalogs', query: { q: searchQuery.value } })
        searchQuery.value = ''
      }
    }

    watch(searchQuery, (newVal) => {
      if (newVal.trim()) {
        router.push({ path: '/catalogs', query: { q: newVal } })
      }
    })

    return {
      isAuthenticated,
      authStore,
      searchQuery,
      redirectToSearch,
      performSearch,
      cartItemsCount
    }
  }
}
</script>
<style>
/* Базовые стили */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body, html {
  margin: 0;
  padding: 0;
  background: #F8F9FA;
  font-family: 'M PLUS Rounded 1c', 'Noto Sans', sans-serif;
  overflow-x: hidden;
  width: 100%;
}

.page-container {
  width: 100%;
  min-height: 100vh;
  background: #F8F9FA;
  overflow-x: hidden;
}

/* Контейнер навигации */
.header {
  padding: 25px 20px;
  background: #F8F9FA;
  position: sticky;
  top: 0;
  z-index: 100;
  width: 100%;
}

.nav-container {
    width: 100%;
  max-width: 1394px;
  margin: 0 auto;
  height: 85px;
  background: #FFFFFF;
  border: 1px solid rgba(0, 0, 0, 0.1);
  border-radius: 30px;
  box-shadow: 0px 4px 12px rgba(0, 0, 0, 0.1);
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 20px;
  padding: 0 25px;
  box-sizing: border-box;
}

/* Группы элементов */
.nav-group {
    display: flex;
    align-items: center;
  gap: 10px;
}

.search-group {
  flex: 1;
  max-width: 600px;
}

/* Поиск */
.search-container {
  width: 100%;
  position: relative;
}

.search-content {
    position: relative;
  z-index: 2;
  display: flex;
  gap: 10px;
}

.search-content input {
  flex: 1;
    height: 40px;
  background: #FFFFFF;
  border: 1px solid rgba(0, 0, 0, 0.1);
    border-radius: 6px;
  color: #333333;
  font-size: 16px;
    padding: 8px 12px;
    outline: none;
    transition: border-color 0.3s ease;
}

.search-content input:focus,
.search-content input:hover {
  border-color: rgba(0, 0, 0, 0.2);
}

.search-button {
    height: 40px;
  padding: 0 20px;
  background: #4CAF50;
  border: none;
  border-radius: 6px;
  color: #FFFFFF;
  font-weight: 700;
    cursor: pointer;
  transition: all 0.3s ease;
}

.search-button:hover {
  background: #45a049;
}

/* Навигационные элементы */
.nav-item {
  position: relative;
  height: 50px;
  padding: 0 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  text-decoration: none;
  border-radius: 10px;
  transition: transform 0.2s;
  color: #333333;
}

.nav-item:hover {
  transform: scale(1.05);
}

.nav-bg {
  position: absolute;
  inset: 0;
  border-radius: inherit;
  z-index: 1;
}

.nav-text {
    position: relative;
  z-index: 2;
  font-weight: 600;
  font-size: 16px;
  color: #333333;
}

/* Специфичные стили для кнопок */
.profile {
  background: #F8F9FA;
  border: 1px solid rgba(0, 0, 0, 0.1);
    }

.profile:hover {
  background: #e9ecef;
    }

.profile .nav-text {
  color: #333333;
}

/* Корзина */
.cart {
  background: #4CAF50;
  border: none;
}

.cart:hover {
  background: #45a049;
}

.cart .nav-text {
  color: #FFFFFF;
}

.cart-content {
  display: flex;
  align-items: center;
  gap: 8px;
}

.cart-icon {
  width: 24px;
  height: 24px;
  color: #FFFFFF;
}

.cart-badge {
  position: absolute;
  top: -8px;
  right: -8px;
  background: #e53e3e;
  color: #FFFFFF;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  font-size: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  border: 2px solid #FFFFFF;
}

/* Медиа-запросы */
@media (max-width: 768px) {
  .nav-container {
    height: auto;
    padding: 15px;
    flex-wrap: wrap;
  }

  .nav-group {
    width: 100%;
    justify-content: center;
    margin: 5px 0;
  }

  .search-group {
    order: -1;
  }

  .nav-item {
    font-size: 14px;
    padding: 0 12px;
    height: 40px;
  }

  .search-content {
    flex-direction: column;
  }

  .search-button {
    width: 100%;
    margin-top: 5px;
  }

  .cart-icon {
    width: 20px;
    height: 20px;
  }

  .cart-badge {
    width: 18px;
    height: 18px;
    font-size: 11px;
  }
}

#app {
  font-family: 'Manrope', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: var(--text-dark);
  background-color: var(--bg-primary);
  min-height: 100vh;
}

/* Импорт переменных */
@import './assets/variables.css';

/* Общие стили */
body {
  margin: 0;
  padding: 0;
  background-color: var(--bg-primary);
}

/* Кнопки */
.btn {
  padding: 0.75rem 1.5rem;
  border-radius: 8px;
  font-weight: 500;
  transition: all 0.3s ease;
  cursor: pointer;
  border: none;
}

.btn-primary {
  background-color: var(--bg-secondary);
  color: var(--text-light);
}

.btn-primary:hover {
  background-color: var(--hover-secondary);
}

.btn-accent {
  background-color: var(--bg-accent);
  color: var(--text-light);
}

.btn-accent:hover {
  background-color: var(--hover-accent);
}

/* Карточки */
.card {
    background-color: #dff6be;
    border: 2px solid #2C763B;
  border-radius: 12px;
  padding: 1.5rem;
    box-shadow: 0 2px 8px rgba(44, 118, 59, 0.1);
    transition: all 0.3s ease;
}

.card:hover {
    background-color: #EBF9E0;
    box-shadow: 0 4px 12px rgba(44, 118, 59, 0.2);
}

/* Формы */
input, 
select, 
textarea {
  background-color: var(--bg-neutral);
  border: 1px solid var(--border-dark);
  color: var(--text-dark);
  padding: 0.75rem 1rem;
  border-radius: 8px;
  transition: all 0.3s ease;
}

input:focus, 
select:focus, 
textarea:focus {
  outline: none;
  border-color: var(--secondary-color);
  box-shadow: 0 0 0 2px rgba(44, 118, 59, 0.2);
}

/* Заголовки */
h1, h2, h3, h4, h5, h6 {
  color: var(--text-dark);
}

/* Ссылки */
a {
  color: var(--secondary-color);
  text-decoration: none;
  transition: color 0.3s ease;
}

a:hover {
  color: var(--hover-secondary);
}

/* Таблицы */
table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  margin: 1rem 0;
}

th {
  background-color: var(--bg-secondary);
  color: var(--text-light);
  padding: 1rem;
  text-align: left;
}

td {
  padding: 1rem;
  border-bottom: 1px solid var(--border-dark);
}

tr:hover {
  background-color: var(--hover-primary);
}
</style>
