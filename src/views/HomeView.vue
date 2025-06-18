<template>
  <div class="body1">
    
    <div class="logo-container">
      <img :src="logosite" alt="Логотип сайта" class="site-logo">
    </div>

    <div class="fr_1">
      <div class="fr-elements">
        <div class="fr-element" id="fr-element-1" :class="{ 'is-visible': visibleIds.includes('fr-element-1') }">
        <p class="txt-element">Гарантия качества товара</p>
      </div>
        <div class="fr-element" id="fr-element-2" :class="{ 'is-visible': visibleIds.includes('fr-element-2') }">
        <p class="txt-element">Лучшие цены в Регионе</p>
      </div>
        <div class="fr-element" id="fr-element-3" :class="{ 'is-visible': visibleIds.includes('fr-element-3') }">
        <p class="txt-element">Работаем 7 дней в неделю</p>
      </div>
        <div class="fr-element" id="fr-element-4" :class="{ 'is-visible': visibleIds.includes('fr-element-4') }">
        <p class="txt-element">Собственное производство</p>
      </div>
    </div>
      </div>

    <div class="fr-4">
      <p class="txt-big-l" id="gallery-title" :class="{ 'is-visible': visibleIds.includes('gallery-title') }">Фотогалерея магазина</p>
      <div class="carousel-container gallery-carousel" id="gallery-carousel" :class="{ 'is-visible': visibleIds.includes('gallery-carousel') }">
        <button class="carousel-button prev" @click="prevGallerySlide">&lt;</button>
        <div class="carousel-wrapper" :style="{ transform: `translateX(-${galleryIndex * 100}%)` }">
          <div class="carousel-slide" v-for="(slide, index) in gallerySlides" :key="index"
               :class="{ 'active': galleryIndex === index }">
            <img :src="slide.image" :alt="slide.alt">
            <div class="slide-overlay">
              <h3>{{ slide.title }}</h3>
              <p>{{ slide.description }}</p>
      </div>
      </div>
      </div>
        <button class="carousel-button next" @click="nextGallerySlide">&gt;</button>
        
        <div class="carousel-dots">
          <span
            v-for="(slide, index) in gallerySlides"
            :key="index"
            :class="{ 'dot': true, 'active': galleryIndex === index }"
            @click="goToGallerySlide(index)"
          ></span>
      </div>
    </div>
  </div>

  <div class="fr-3">
    <p class="txt-big-r" id="txt-big-r" :class="{ 'is-visible': visibleIds.includes('txt-big-r') }">Примеры применения</p>
    
    <div class="carousel-container" id="first-carousel" :class="{ 'is-visible': visibleIds.includes('first-carousel') }">
      <button class="carousel-button prev" @click="prevSlide">&lt;</button>
      <div class="carousel-wrapper" :style="{ transform: `translateX(-${currentIndex * 100}%)` }">
        <div class="carousel-slide" v-for="(slide, index) in slides" :key="index"
             :class="{ 'active': currentIndex === index }">
          <img :src="slide.image" :alt="slide.alt">
          <div class="slide-overlay">
            <h3>{{ slide.title }}</h3>
            <p>{{ slide.description }}</p>
          </div>
        </div>
      </div>
      <button class="carousel-button next" @click="nextSlide">&gt;</button>
      
      <div class="carousel-dots">
        <span
          v-for="(slide, index) in slides"
          :key="index"
          :class="{ 'dot': true, 'active': currentIndex === index }"
          @click="goToSlide(index)"
        ></span>
      </div>
    </div>
    
    <div class="fr-3-2">
        <p class="txt-small-m" id="txt-small-m" :class="{ 'is-visible': visibleIds.includes('txt-small-m') }">Современные технологии хранения и профессиональный уход за цветами позволяют нам предлагать свежую продукцию высочайшего качества по доступным ценам. Наш склад оборудован специальными холодильными камерами с контролем влажности и температуры, что гарантирует максимальную свежесть и длительное сохранение товарного вида каждого цветка.</p>
        <p class="txt-small-m2" id="txt-small-m2" :class="{ 'is-visible': visibleIds.includes('txt-small-m2') }">Широкий ассортимент включает в себя более 50 сортов роз, множество видов хризантем, гвоздик, лилий и других популярных цветов. Мы также предлагаем сезонные букеты, горшечные растения и все необходимые материалы для флористов - от упаковки до декоративных элементов. Наши опытные специалисты всегда готовы помочь с выбором и обеспечить профессиональную консультацию.</p>
    </div>
  </div>

          </div>

  <!-- Секция "О нас" -->
  <div class="about-section" id="about-section" :class="{ 'is-visible': visibleIds.includes('about-section') }">
    <div class="about-content">
      <h2 class="about-title">О компании "Флора Альянс"</h2>
      <div class="about-grid">
        <div class="about-info">
          <div class="about-text">
            <p>Мы - ведущая оптовая компания по продаже цветов в Курске, работающая с 2010 года. Наша команда состоит из профессиональных флористов и специалистов по уходу за цветами, которые обеспечивают высочайшее качество каждого растения.</p>
            <p>Наши преимущества:</p>
            <ul>
              <li>Прямые поставки свежих цветов из Голландии и Эквадора</li>
              <li>Современное холодильное оборудование для хранения</li>
              <li>Широкий ассортимент цветов и флористических материалов</li>
              <li>Профессиональная консультация для флористов</li>
            </ul>
        </div>
      </div>
        <div class="contact-info">
          <h3>Контакты</h3>
          <div class="contact-details">
            <div class="contact-item">
              <i class="fas fa-map-marker-alt"></i>
              <span>г. Курск, ул. Парашютная, 1</span>
      </div>
            <div class="contact-item">
              <i class="fas fa-phone"></i>
              <span>+7 (4712) 73-46-01</span>
    </div>
            <div class="contact-item">
              <i class="fas fa-envelope"></i>
              <span>info@flora-alliance.ru</span>
  </div>
            <div class="contact-item">
              <i class="fas fa-clock"></i>
              <span>Пн-Вс: 9:00 - 18:00</span>
      </div>
      </div>
      </div>
    </div>
  </div>
</div>
</template>

<script>
import { ref } from 'vue';
import image1 from '@/assets/carousel/1_1.jpg';
import image3 from '@/assets/carousel/1_3.jpg';
import gallery2 from '@/assets/carousel/photo/1.png';
import gallery1 from '@/assets/carousel/photo/2.webp';
import gallery3 from '@/assets/carousel/photo/3.webp';
import logosite from '@/assets/img/logosite.png';

export default {
  setup() {
    const slide = ref(0);
    return { slide };
  },
  data() {
    return {
      logosite,
      visibleIds: [],
      latestProducts: [
        {
          id: 1,
          name: 'Букет "Весеннее настроение"',
          price: 3500,
          image: '/images/products/bouquet1.jpg'
        },
        {
          id: 2,
          name: 'Композиция "Нежность"',
          price: 4200,
          image: '/images/products/bouquet2.jpg'
        },
        {
          id: 3,
          name: 'Букет "Яркий день"',
          price: 3800,
          image: '/images/products/bouquet3.jpg'
        }
      ],
      slides: [
        { 
          image: image1, 
          alt: 'Цветы оптом',
          title: 'Свежие цветы оптом',
          description: 'Широкий ассортимент свежесрезанных цветов'
        },
        { 
          image: image3, 
          alt: 'Флористика',
          title: 'Профессиональные флористы',
          description: 'Все для флористов и цветочных магазинов'
        }
      ],
      gallerySlides: [
        {
          image: gallery1,
          alt: 'Склад цветов',
          title: 'Наш склад',
          description: 'Современное оборудование для хранения цветов'
        },
        {
          image: gallery2,
          alt: 'Ассортимент',
          title: 'Большой выбор',
          description: 'Более 1000 наименований цветов и растений'
        },
        {
          image: gallery3,
          alt: 'Упаковка',
          title: 'Упаковочные материалы',
          description: 'Все для оформления букетов'
        }
      ],
      currentIndex: 0,
      galleryIndex: 0,
      autoPlayInterval: null,
      galleryInterval: null
    }
  },
  methods: {
    goToSlide(index) {
      this.currentIndex = index;
    },
    goToGallerySlide(index) {
      this.galleryIndex = index;
    },
    nextSlide() {
      this.currentIndex = (this.currentIndex + 1) % this.slides.length;
    },
    nextGallerySlide() {
      this.galleryIndex = (this.galleryIndex + 1) % this.gallerySlides.length;
    },
    prevSlide() {
      this.currentIndex = this.currentIndex === 0 
        ? this.slides.length - 1 
        : this.currentIndex - 1;
    },
    prevGallerySlide() {
      this.galleryIndex = this.galleryIndex === 0 
        ? this.gallerySlides.length - 1 
        : this.galleryIndex - 1;
    },
    startAutoPlay() {
      this.autoPlayInterval = setInterval(() => {
        this.nextSlide();
      }, 5000);
      this.galleryInterval = setInterval(() => {
        this.nextGallerySlide();
      }, 6000);
    },
    stopAutoPlay() {
      if (this.autoPlayInterval) {
        clearInterval(this.autoPlayInterval);
      }
      if (this.galleryInterval) {
        clearInterval(this.galleryInterval);
      }
    }
  },
  mounted() {
    const idsToObserve = [
      'fr-element-1',
      'fr-element-2',
      'fr-element-3',
      'fr-element-4',
      'section1',
      'section2',
      'section3',
      'section4',
      'section5',
      'txt-big-r',
      'txt-small-m',
      'txt-small-m2',
      'first-carousel',
      'gallery-title',
      'gallery-carousel',
      'about-section'
    ];
    
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            const id = entry.target.id;
            if (!this.visibleIds.includes(id)) {
              this.visibleIds.push(id);
            }
            observer.unobserve(entry.target);
          }
        });
      },
      {
        root: null,
        rootMargin: '0px',
        threshold: 0.1,
      }
    );

    idsToObserve.forEach((id) => {
      const element = document.getElementById(id);
      if (element) {
        observer.observe(element);
      }
    });

    this.startAutoPlay();
  },
  beforeUnmount() {
    this.stopAutoPlay();
  }
}
</script>

<style scoped>
.body1 {
  width: 100%;
  min-height: 100vh;
  background: #FFFFFF;
}

.logo-container {
  width: 100%;
  display: flex;
  justify-content: center;
  padding: 2rem 0;
  background: #FFFFFF;
}

.site-logo {
  max-width: 473px;
  height: auto;
}

.fr_1 {
  width: 100%;
  padding: 2rem 1rem;
  background: #FFFFFF;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2rem;
}

.fr-elements {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 2rem;
  width: 100%;
  max-width: 1200px;
  margin-top: 2rem;
}

.fr-element {
  background: #F8F9FA;
  padding: 2rem;
  border-radius: 15px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
  opacity: 0;
  transform: translateY(20px);
  transition: all 0.3s ease;
}

.fr-element.is-visible {
  opacity: 1;
  transform: translateY(0);
}

.fr-element:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.15);
}

.txt-element {
  font-size: 1.5rem;
  font-weight: 500;
  text-align: center;
  color: #212529;
  line-height: 1.4;
}

.fr-4 {
  width: 100%;
  padding: 4rem 1rem;
  background: #FFFFFF;
  text-align: center;
}

.txt-big-l {
  font-size: 2rem;
  margin-bottom: 3rem;
  color: #4CAF50;
  opacity: 0;
  transform: translateX(40px);
}

.txt-big-l.is-visible {
  opacity: 1;
  transform: translateX(0);
  transition: all 0.6s ease;
}

.gallery-carousel {
  margin-top: 2rem;
}

/* Текстовый блок */
.fr-2 {
  width: 100%;
  padding: 6rem 2rem;
  background: #FFFFFF;
  position: relative;
}

.fr-2::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(0, 0, 0, 0.1), transparent);
}

.text-sections {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2.5rem;
  max-width: 1200px;
  margin: 0 auto;
}

.text-section {
  background: #F8F9FA;
  border-radius: 20px;
  padding: 2.5rem;
  border: 1px solid rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  opacity: 0;
  transform: translateY(20px);
}

.text-section.is-visible {
  opacity: 1;
  transform: translateY(0);
}

.text-section:hover {
  background: #FFFFFF;
  transform: translateY(-5px);
  border-color: rgba(0, 0, 0, 0.15);
}

.section-title {
  font-size: 1.8rem;
  margin-bottom: 1.5rem;
  color: #4CAF50;
  font-weight: 600;
  position: relative;
  padding-bottom: 1rem;
}

.section-title::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  width: 60px;
  height: 2px;
  background: linear-gradient(90deg, #4CAF50, transparent);
}

.section-text {
  font-size: 1.1rem;
  line-height: 1.7;
  color: #495057;
}

/* Блок с примерами */
.fr-3 {
  width: 100%;
  padding: 2rem 1rem;
  text-align: center;
  background: #F8F9FA;
}

.txt-big-r {
  font-size: 2rem;
  margin-bottom: 1rem;
  opacity: 0;
  transform: translateX(-40px);
  color: #4CAF50;
}

.txt-big-r.is-visible {
  opacity: 1;
  transform: translateX(0);
  transition: all 0.6s ease;
}

.txt-small-m, .txt-small-m2 {
  font-size: 1rem;
  color: #495057;
  margin-bottom: 2rem;
  max-width: 800px;
  margin-left: auto;
  margin-right: auto;
  opacity: 0;
  transform: translateY(20px);
}

.txt-small-m.is-visible, .txt-small-m2.is-visible {
  opacity: 1;
  transform: translateY(0);
  transition: all 0.6s ease;
}

/* Карусель */
.carousel-container {
  position: relative;
  width: 100%;
  max-width: 1200px;
  margin: 4rem auto;
  border-radius: 24px;
  overflow: hidden;
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.3);
  opacity: 0;
  transform: translateY(40px);
}

.carousel-container.is-visible {
  opacity: 1;
  transform: translateY(0);
  transition: all 0.8s ease;
}

.carousel-wrapper {
  display: flex;
  transition: transform 0.7s ease;
  height: 600px;
  width: 100%;
}

.carousel-slide {
  flex: 0 0 100%;
  position: relative;
  overflow: hidden;
  width: 100%;
}

.carousel-slide img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.7s ease;
  display: block;
}

.carousel-slide:hover img {
  transform: scale(1.05);
}

.slide-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.8));
  padding: 3rem 2rem;
  transform: translateY(0);
  transition: transform 0.5s ease;
}

.slide-overlay h3 {
  font-size: 2.2rem;
  margin: 0 0 1rem;
  font-weight: 700;
  background: linear-gradient(135deg, #ffffff, #dedede);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.slide-overlay p {
  font-size: 1.2rem;
  color: #ffffff;
  opacity: 0.9;
  max-width: 600px;
}

.carousel-button {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(76, 175, 80, 0.2);
  border: none;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  cursor: pointer;
  color: #4CAF50;
  font-size: 1.8rem;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
  backdrop-filter: blur(5px);
}

.carousel-button:hover {
  background: rgba(76, 175, 80, 0.3);
  transform: translateY(-50%) scale(1.1);
}

.carousel-button.prev {
  left: 1rem;
}

.carousel-button.next {
  right: 1rem;
}

.carousel-dots {
  position: absolute;
  bottom: 1rem;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 0.5rem;
}

.dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.5);
  cursor: pointer;
  transition: all 0.3s ease;
}

.dot.active {
  background: #4CAF50;
  transform: scale(1.2);
}

/* Медиа-запросы для адаптивности */
@media (max-width: 768px) {
  .carousel-wrapper {
    height: 400px;
  }

  .slide-overlay h3 {
    font-size: 1.8rem;
  }

  .slide-overlay p {
    font-size: 1rem;
  }

  .txt1 {
    font-size: 2rem;
  }

  .section-title {
    font-size: 1.5rem;
  }

  .fr-elements {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 480px) {
  .carousel-wrapper {
    height: 300px;
  }

  .carousel-button {
    width: 40px;
    height: 40px;
    font-size: 1.5rem;
  }

  .txt1 {
    font-size: 1.8rem;
  }

  .txt-element {
    font-size: 1.2rem;
  }
}

.text-center {
  text-align: center;
}

.products-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 2rem;
    padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
}

.product-card {
  background: #FFFFFF;
  border-radius: 15px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  opacity: 0;
  transform: translateY(20px);
}

.product-card.is-visible {
  opacity: 1;
  transform: translateY(0);
  transition: all 0.6s ease;
}

.product-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.product-image {
  width: 100%;
  height: 250px;
  object-fit: cover;
}

.product-info {
    padding: 1.5rem;
  }

.product-name {
  font-size: 1.2rem;
  font-weight: 600;
  color: #333;
  margin-bottom: 0.5rem;
}

.product-price {
  font-size: 1.4rem;
  color: #4CAF50;
  font-weight: 700;
  margin-bottom: 1rem;
}

.view-button {
  background: #4CAF50;
  color: white;
  border: none;
  padding: 0.8rem 1.5rem;
  border-radius: 8px;
  font-size: 1rem;
  cursor: pointer;
  transition: background 0.3s ease;
  width: 100%;
}

.view-button:hover {
  background: #45a049;
}

@media (max-width: 768px) {
  .products-grid {
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    padding: 1rem;
  }
}

/* Секция "О нас" */
.about-section {
  width: 100%;
  padding: 5rem 2rem;
  background: linear-gradient(to bottom, #FFFFFF, #F8F9FA);
  opacity: 0;
  transform: translateY(40px);
  transition: all 0.8s ease;
}

.about-section.is-visible {
  opacity: 1;
  transform: translateY(0);
}

.about-content {
  max-width: 1200px;
  margin: 0 auto;
}

.about-title {
  font-size: 2.5rem;
  color: #333;
  text-align: center;
  margin-bottom: 3rem;
  font-weight: 600;
}

.about-grid {
  display: grid;
  grid-template-columns: 1.5fr 1fr;
  gap: 4rem;
  align-items: start;
}

.about-info {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.about-text {
  color: #666;
  font-size: 1.1rem;
  line-height: 1.6;
}

.about-text p {
  margin-bottom: 1.5rem;
}

.about-text ul {
  list-style: none;
  padding-left: 1.5rem;
}

.about-text ul li {
  position: relative;
  margin-bottom: 0.75rem;
  padding-left: 1.5rem;
}

.about-text ul li::before {
  content: "•";
  color: #4CAF50;
  font-size: 1.5rem;
  position: absolute;
  left: 0;
  top: -0.25rem;
}

.contact-info {
  background: #FFFFFF;
  padding: 2.5rem;
  border-radius: 20px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.1);
}

.contact-info h3 {
  font-size: 1.8rem;
  color: #333;
  margin-bottom: 2rem;
}

.contact-details {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.contact-item {
  display: flex;
  align-items: center;
  gap: 1rem;
  color: #666;
  font-size: 1.1rem;
}

.contact-item i {
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #4CAF50;
  color: white;
  border-radius: 50%;
  font-size: 0.9rem;
}

/* Адаптивность */
@media (max-width: 1024px) {
  .about-grid {
    grid-template-columns: 1fr;
    gap: 3rem;
  }
}

@media (max-width: 768px) {
  .about-section {
    padding: 3rem 1.5rem;
  }

  .about-title {
    font-size: 2rem;
    margin-bottom: 2rem;
  }
}

@media (max-width: 480px) {
  .about-section {
    padding: 2rem 1rem;
  }

  .contact-info {
    padding: 1.5rem;
  }
}
</style>