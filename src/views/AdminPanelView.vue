<template>
  <div class="admin-container">
    <!-- Проверка прав администратора -->
    <div v-if="!authStore.isAdmin" class="access-denied">
      <h2>Доступ запрещен</h2>
      <p>У вас недостаточно прав для просмотра этой страницы</p>
      <p>Текущая роль: {{ authStore.userRole || 'не определена' }}</p>
    </div>

    <!-- Админ-панель -->
    <div v-else class="admin-panel">
      <div class="admin-header">
        <h1>Панель управления</h1>
        <div class="admin-stats">
          <div class="stat-card">
            <i class="fas fa-users"></i>
            <div class="stat-info">
              <span class="stat-value">{{ users.length }}</span>
              <span class="stat-label">Пользователей</span>
            </div>
          </div>
          <div class="stat-card">
            <i class="fas fa-box"></i>
            <div class="stat-info">
              <span class="stat-value">{{ products.length }}</span>
              <span class="stat-label">Товаров</span>
            </div>
          </div>
          <div class="stat-card">
            <i class="fas fa-tags"></i>
            <div class="stat-info">
              <span class="stat-value">{{ categories.length }}</span>
              <span class="stat-label">Категорий</span>
            </div>
          </div>
        </div>
      </div>

      <div class="tabs">
        <button
          v-for="tab in tabs"
          :key="tab"
          @click="currentTab = tab"
          :class="{ active: currentTab === tab }"
        >
          <i :class="getTabIcon(tab)"></i>
          {{ tab }}
        </button>
      </div>

      <!-- Индикатор загрузки -->
      <div v-if="isLoading" class="loading-indicator">
        <div class="spinner"></div>
        <span>Загрузка данных...</span>
      </div>

      <!-- Контент вкладок -->
      <div v-else class="tab-content">
        <!-- Пользователи -->
        <div v-if="currentTab === 'Пользователи'" class="section">
          <div class="section-header">
            <h2>Управление пользователями</h2>
            <button @click="openUserModal(null)" class="add-button">
              <i class="fas fa-plus"></i>
              Добавить пользователя
            </button>
          </div>
          
          <div class="table-container">
            <table class="data-table">
              <thead>
                <tr>
                  <th>ID</th>
                  <th>Имя</th>
                  <th>Email</th>
                  <th>Телефон</th>
                  <th>Роль</th>
                  <th>Действия</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="user in users" :key="user.id">
                  <td>{{ user.id }}</td>
                  <td>{{ user.username }}</td>
                  <td>{{ user.email }}</td>
                  <td>{{ user.phone }}</td>
                  <td>
                    <span :class="['role-badge', user.role]">
                      {{ user.role === 'admin' ? 'Администратор' : 'Покупатель' }}
                    </span>
                  </td>
                  <td class="actions">
                    <button @click="openUserModal(user)" class="action-btn edit">
                      <i class="fas fa-edit"></i>
                      <span>Изменить</span>
                    </button>
                    <button @click="deleteUser(user.id)" class="action-btn delete">
                      <i class="fas fa-trash"></i>
                      <span>Удалить</span>
                    </button>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <!-- Товары -->
        <div v-if="currentTab === 'Товары'" class="section">
          <div class="section-header">
            <h2>Управление товарами</h2>
            <button @click="openProductModal(null)" class="add-button">
              <i class="fas fa-plus"></i>
              Добавить товар
            </button>
          </div>
          
          <div class="table-container">
            <table class="data-table">
              <thead>
                <tr>
                  <th>ID</th>
                  <th>Название</th>
                  <th>Категория</th>
                  <th>Цена</th>
                  <th>Артикул</th>
                  <th>Действия</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="product in products" :key="product.id">
                  <td>{{ product.id }}</td>
                  <td>{{ product.name }}</td>
                  <td>
                    <span class="category-badge">
                      {{ product.category ? product.category.name : 'Не указана' }}
                    </span>
                  </td>
                  <td>{{ product.price }} ₽</td>
                  <td>{{ product.article }}</td>
                  <td class="actions">
                    <button @click="openProductModal(product)" class="action-btn edit">
                      <i class="fas fa-edit"></i>
                      <span>Изменить</span>
                    </button>
                    <button @click="deleteProduct(product.id)" class="action-btn delete">
                      <i class="fas fa-trash"></i>
                      <span>Удалить</span>
                    </button>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <!-- Категории -->
        <div v-if="currentTab === 'Категории'" class="section">
          <div class="section-header">
            <h2>Управление категориями</h2>
            <button @click="openCategoryModal(null)" class="add-button">
              <i class="fas fa-plus"></i>
              Добавить категорию
            </button>
          </div>
          
          <div class="table-container">
            <table class="data-table">
              <thead>
                <tr>
                  <th>ID</th>
                  <th>Название</th>
                  <th>Действия</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="category in categories" :key="category.id">
                  <td>{{ category.id }}</td>
                  <td>{{ category.name }}</td>
                  <td class="actions">
                    <button @click="openCategoryModal(category)" class="action-btn edit">
                      <i class="fas fa-edit"></i>
                      <span>Изменить</span>
                    </button>
                    <button @click="deleteCategory(category.id)" class="action-btn delete">
                      <i class="fas fa-trash"></i>
                      <span>Удалить</span>
                    </button>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- Модальные окна -->
      <UserModal
        v-if="showUserModal"
        :user="selectedUser"
        @save="saveUser"
        @close="showUserModal = false"
      />
      
      <ProductModal
        v-if="showProductModal"
        :product="selectedProduct"
        :categories="categories"
        @save="saveProduct"
        @close="showProductModal = false"
      />
      
      <CategoryModal
        v-if="showCategoryModal"
        :category="selectedCategory"
        @save="saveCategory"
        @close="showCategoryModal = false"
      />

      <!-- Сообщения об ошибках -->
      <div v-if="error" class="error-message">
        <i class="fas fa-exclamation-circle"></i>
        {{ error }}
        <button @click="error = null" class="close-error">
          <i class="fas fa-times"></i>
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import { useAuthStore } from '@/store/auth'
import UserModal from '@/components/admin/UserModal.vue'
import ProductModal from '@/components/admin/ProductModal.vue'
import CategoryModal from '@/components/admin/CategoryModal.vue'

const authStore = useAuthStore()
const tabs = ['Пользователи', 'Товары', 'Категории']
const currentTab = ref('Пользователи')

// Данные
const users = ref([])
const products = ref([])
const categories = ref([])

// Состояние
const isLoading = ref(false)
const error = ref(null)

// Модальные окна
const showUserModal = ref(false)
const showProductModal = ref(false)
const showCategoryModal = ref(false)

const selectedUser = ref(null)
const selectedProduct = ref(null)
const selectedCategory = ref(null)

// Получение иконки для вкладки
const getTabIcon = (tab) => {
  switch (tab) {
    case 'Пользователи':
      return 'fas fa-users'
    case 'Товары':
      return 'fas fa-box'
    case 'Категории':
      return 'fas fa-tags'
    default:
      return 'fas fa-folder'
  }
}


// Загрузка данных
const fetchData = async () => {
  try {
    isLoading.value = true
    error.value = null
    
    const api = authStore.getApiClient()
    
    console.log('Начинаем загрузку данных...')
    
    const [usersRes, productsRes, categoriesRes] = await Promise.all([
      api.get('users/'),
      api.get('products/'),
      api.get('categories/')
    ])

    console.log('Ответ users:', usersRes.data)
    console.log('Ответ products:', productsRes.data)
    console.log('Ответ categories:', categoriesRes.data)

    users.value = usersRes.data
    products.value = productsRes.data
    categories.value = categoriesRes.data

    console.log('Данные после присвоения:')
    console.log('users:', users.value)
    console.log('products:', products.value)
    console.log('categories:', categories.value)

  } catch (err) {
    console.error('Ошибка загрузки данных:', err)
    error.value = err.response?.data?.message || 'Ошибка при загрузке данных'

    if (err.response?.status === 401) {
      authStore.logout()
    }
  } finally {
    isLoading.value = false
  }
}

// Методы для пользователей
const openUserModal = (user) => {
  selectedUser.value = user ? { ...user } : {
    username: '',
    email: '',
    phone: '',
    role: 'customer'
  }
  showUserModal.value = true
}

const saveUser = async (userData) => {
  try {
    const api = authStore.getApiClient()
    
    if (userData.id) {
      await api.put(`users/${userData.id}/`, userData)
    } else {
      await api.post('users/', userData)
    }
    
    showUserModal.value = false
    fetchData()
  } catch (err) {
    error.value = err.response?.data?.message || 'Ошибка при сохранении пользователя'
  }
}

const deleteUser = async (userId) => {
  if (!confirm('Вы уверены, что хотите удалить этого пользователя?')) return
  
  try {
    const api = authStore.getApiClient()
    await api.delete(`users/${userId}/`)
    fetchData()
  } catch (err) {
    error.value = err.response?.data?.message || 'Ошибка при удалении пользователя'
  }
}

// Методы для товаров
const openProductModal = (product) => {
  selectedProduct.value = product ? { ...product } : {
    name: '',
    price: 0,
    article: '',
    category: null
  }
  showProductModal.value = true
}

const saveProduct = async (productData) => {
  try {
    console.log('Отправляемые данные товара:', productData)
    console.log('Тип изображения:', productData.image ? typeof productData.image : 'нет изображения')
    if (productData.image) {
      console.log('Информация об изображении:', {
        name: productData.image.name,
        type: productData.image.type,
        size: productData.image.size
      })
    }
    
    const api = authStore.getApiClient()
    const formData = new FormData()
    
    // Добавляем все поля в FormData
    formData.append('name', productData.name)
    formData.append('category_id', productData.category)
    formData.append('article', productData.article)
    formData.append('price', productData.price)
    if (productData.specifications) {
      formData.append('specifications', productData.specifications)
    }
    if (productData.description) {
      formData.append('description', productData.description)
    }
    if (productData.image) {
      formData.append('image', productData.image)
    }

    // Логируем содержимое FormData
    console.log('Содержимое FormData:')
    for (let pair of formData.entries()) {
      console.log(`${pair[0]}: ${pair[1] instanceof File ? 
        `File(${pair[1].name}, ${pair[1].type}, ${pair[1].size} bytes)` : 
        pair[1]}`)
    }
    
    if (productData.id) {
      await api.put(`products/${productData.id}/`, formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      })
    } else {
      const response = await api.post('products/', formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      })
      console.log('Ответ сервера:', response.data)
    }
    
    showProductModal.value = false
    fetchData()
  } catch (err) {
    console.error('Полная ошибка:', err)
    console.error('Данные ошибки:', err.response?.data)
    if (err.response?.data?.image) {
      console.error('Ошибки валидации изображения:', err.response.data.image)
    }
    error.value = err.response?.data?.message || 'Ошибка при сохранении товара'
  }
}

const deleteProduct = async (productId) => {
  if (!confirm('Вы уверены, что хотите удалить этот товар?')) return
  
  try {
    const api = authStore.getApiClient()
    await api.delete(`products/${productId}/`)
    fetchData()
  } catch (err) {
    error.value = err.response?.data?.message || 'Ошибка при удалении товара'
  }
}

// Методы для категорий
const openCategoryModal = (category) => {
  selectedCategory.value = category ? { ...category } : {
    name: ''
  }
  showCategoryModal.value = true
}

const saveCategory = async (categoryData) => {
  try {
    const api = authStore.getApiClient()
    const formData = new FormData()
    formData.append('name', categoryData.name)
    if (categoryData.image) {
      formData.append('image', categoryData.image)
    }
    
    if (categoryData.id) {
      await api.put(`categories/${categoryData.id}/`, formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      })
    } else {
      await api.post('categories/', formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      })
    }
    
    showCategoryModal.value = false
    fetchData()
  } catch (err) {
    error.value = err.response?.data?.message || 'Ошибка при сохранении категории'
  }
}

const deleteCategory = async (categoryId) => {
  if (!confirm('Вы уверены, что хотите удалить эту категорию?')) return
  
  try {
    const api = authStore.getApiClient()
    await api.delete(`categories/${categoryId}/`)
    fetchData()
  } catch (err) {
    error.value = err.response?.data?.message || 'Ошибка при удалении категории'
  }
}

// Загрузка данных при монтировании и смене вкладки
onMounted(() => {
  console.log('Проверка прав:')
  console.log('authStore.isAdmin:', authStore.isAdmin)
  console.log('authStore.userRole:', authStore.userRole)
  console.log('authStore.isAuthenticated:', authStore.isAuthenticated)
  
  if (authStore.isAdmin) {
    fetchData()
  }
})

watch(currentTab, fetchData)
</script>

<style>
/* Основные стили контейнера */
.admin-container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 2rem;
  min-height: 100vh;
  background: var(--bg-primary);
  color: var(--text-dark);
}

.admin-panel {
  animation: fadeIn 0.3s ease-out;
}

/* Заголовок и статистика */
.admin-header {
  margin-bottom: 2rem;
}

.admin-header h1 {
  font-size: 2rem;
  margin-bottom: 1.5rem;
  color: var(--text-dark);
  font-weight: 600;
}

.admin-stats {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
  margin-bottom: 2rem;
}

.stat-card {
  background: var(--bg-neutral);
  padding: 1.5rem;
  border-radius: 12px;
  display: flex;
  align-items: center;
  gap: 1rem;
  transition: transform 0.2s ease;
}

.stat-card:hover {
  transform: translateY(-2px);
}

.stat-card i {
  font-size: 2rem;
  color: var(--accent-color);
}

.stat-info {
  display: flex;
  flex-direction: column;
}

.stat-value {
  font-size: 1.5rem;
  font-weight: 600;
  color: var(--text-dark);
}

.stat-label {
  color: var(--text-muted);
  font-size: 0.9rem;
}

/* Вкладки */
.tabs {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 2rem;
  border-bottom: 1px solid var(--border-dark);
  padding-bottom: 1rem;
}

.tabs button {
  background: var(--bg-neutral);
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 8px;
  color: var(--text-muted);
  font-size: 0.9rem;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.tabs button i {
  font-size: 1rem;
}

.tabs button:hover {
  background: var(--hover-primary);
  color: var(--text-dark);
}

.tabs button.active {
  background: var(--bg-secondary);
  color: var(--text-light);
}

/* Секции */
.section {
  background: var(--bg-neutral);
  border-radius: 12px;
  padding: 1.5rem;
  margin-bottom: 2rem;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
}

.section-header h2 {
  font-size: 1.25rem;
  color: var(--text-dark);
  font-weight: 500;
}

/* Кнопки */
.add-button {
  background: var(--bg-accent);
  color: var(--text-light);
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.9rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  transition: all 0.2s ease;
}

.add-button:hover {
  background: var(--hover-accent);
  transform: translateY(-1px);
}

.add-button i {
  font-size: 0.9rem;
}

/* Таблицы */
.table-container {
  overflow-x: auto;
  border-radius: 8px;
  background: var(--bg-primary);
}

.data-table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  font-size: 0.9rem;
}

.data-table th {
  background: var(--bg-secondary);
  color: var(--text-light);
  font-weight: 500;
  text-align: left;
  padding: 1rem;
  border-bottom: 1px solid var(--border-dark);
}

.data-table td {
  padding: 1rem;
  color: var(--text-dark);
  border-bottom: 1px solid var(--border-dark);
}

.data-table tbody tr:hover {
  background: var(--hover-primary);
}

/* Бейджи */
.role-badge {
  padding: 0.25rem 0.75rem;
  border-radius: 20px;
  font-size: 0.8rem;
  font-weight: 500;
}

.role-badge.admin {
  background: rgba(44, 118, 59, 0.2);
  color: var(--secondary-color);
}

.role-badge.customer {
  background: rgba(215, 84, 128, 0.2);
  color: var(--accent-color);
}

.category-badge {
  background: rgba(44, 118, 59, 0.2);
  color: var(--secondary-color);
  padding: 0.25rem 0.75rem;
  border-radius: 20px;
  font-size: 0.8rem;
}

/* Кнопки действий */
.actions {
  display: flex;
  gap: 0.5rem;
}

.action-btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  background: none;
  border: none;
  padding: 0.5rem 1rem;
  cursor: pointer;
  border-radius: 6px;
  transition: all 0.2s ease;
  color: var(--text-light);
}

.action-btn i {
  font-size: 0.9rem;
}

.action-btn span {
  font-size: 0.9rem;
}

.action-btn.edit {
  background: var(--secondary-color);
}

.action-btn.edit:hover {
  background: var(--hover-secondary);
  transform: translateY(-1px);
}

.action-btn.delete {
  background: var(--accent-color);
}

.action-btn.delete:hover {
  background: var(--hover-accent);
  transform: translateY(-1px);
}

/* Загрузчик */
.loading-indicator {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 3rem;
  color: var(--text-muted);
}

.spinner {
  width: 24px;
  height: 24px;
  border: 3px solid var(--border-dark);
  border-top-color: var(--accent-color);
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-right: 1rem;
}

/* Сообщения об ошибках */
.error-message {
  position: fixed;
  bottom: 2rem;
  right: 2rem;
  background: var(--bg-neutral);
  color: var(--accent-color);
  padding: 1rem 1.5rem;
  border-radius: 8px;
  display: flex;
  align-items: center;
  gap: 0.75rem;
  box-shadow: var(--shadow-md);
  animation: slideIn 0.3s ease-out;
  max-width: 400px;
  z-index: 1000;
}

.error-message i {
  font-size: 1.25rem;
}

.close-error {
  background: none;
  border: none;
  color: var(--text-muted);
  cursor: pointer;
  padding: 0.25rem;
  margin-left: auto;
  transition: color 0.2s ease;
}

.close-error:hover {
  color: var(--text-dark);
}

/* Анимации */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
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

@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

/* Адаптивность */
@media (max-width: 768px) {
  .admin-container {
    padding: 1rem;
  }

  .admin-stats {
    grid-template-columns: 1fr;
  }

  .tabs {
    flex-wrap: wrap;
  }

  .tabs button {
    flex: 1 1 calc(50% - 0.25rem);
  }

  .section {
    padding: 1rem;
  }

  .section-header {
    flex-direction: column;
    gap: 1rem;
    align-items: stretch;
  }

  .add-button {
    width: 100%;
    justify-content: center;
  }

  .data-table {
    font-size: 0.8rem;
  }

  .data-table th,
  .data-table td {
    padding: 0.75rem;
  }
}
</style>