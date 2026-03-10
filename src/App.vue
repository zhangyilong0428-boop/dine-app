<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';

interface MenuItem {
  name: string;
  price: number;
  image: string;
}

interface CartItem extends MenuItem {
  quantity: number;
}

const menu = ref<Record<string, MenuItem[]>>({});

onMounted(async () => {
  const response = await fetch('/menu.json');
  menu.value = await response.json();
});

const categories = [
  { key: 'all', name: '全部' },
  { key: 'stir-fried', name: '炒' },
  { key: 'boiled', name: '煮' },
  { key: 'fried', name: '煎' },
  { key: 'soup', name: '汤' },
  { key: 'carbohydrates', name: '主食' },
  { key: 'drinks', name: '饮料' }
];

const activeCategory = ref('all');
const cart = ref<CartItem[]>([]);

const menuItems = computed(() => {
  if (!menu.value || Object.keys(menu.value).length === 0) {
    return [];
  }
  if (activeCategory.value === 'all') {
    return Object.values(menu.value).flat();
  }
  return menu.value[activeCategory.value] || [];
});

const totalPrice = computed(() => {
  return cart.value.reduce((total, item) => total + item.price * item.quantity, 0);
});

function addToCart(item: MenuItem) {
  const existingItem = cart.value.find(cartItem => cartItem.name === item.name);
  if (existingItem) {
    existingItem.quantity++;
  } else {
    cart.value.push({ ...item, quantity: 1 });
  }
}

function checkout() {
  if (cart.value.length > 0) {
    alert('结算成功！');
    cart.value = [];
  } else {
    alert('购物车是空的！');
  }
}

function handleImageError(e: Event) {
  const target = e.target as HTMLImageElement;
  target.src = 'https://placehold.co/200x150?text=No+Image';
}
</script>

<template>
  <header>
    <h1>Dine餐厅</h1>
  </header>
  <main>
    <div class="categories">
      <button
        v-for="category in categories"
        :key="category.key"
        class="category-btn"
        :class="{ active: activeCategory === category.key }"
        @click="activeCategory = category.key"
      >
        {{ category.name }}
      </button>
    </div>
    <div class="menu">
      <div v-for="item in menuItems" :key="item.name" class="menu-item">
        <img :src="'/images/' + item.image" :alt="item.name" @error="handleImageError" />
        <h3>{{ item.name }}</h3>
        <p>￥{{ item.price.toFixed(2) }}</p>
        <button class="add-to-cart-btn" @click="addToCart(item)">加入购物车</button>
      </div>
    </div>
  </main>
  <aside class="cart">
    <h2>购物车</h2>
    <ul class="cart-items">
      <li v-for="item in cart" :key="item.name" class="cart-item">
        <span>{{ item.name }} x {{ item.quantity }}</span>
        <span>￥{{ (item.price * item.quantity).toFixed(2) }}</span>
      </li>
    </ul>
    <div class="cart-total">
      <p>总计: <span id="total-price">￥{{ totalPrice.toFixed(2) }}</span></p>
      <button id="checkout-btn" @click="checkout">结算</button>
    </div>
  </aside>
</template>

<style scoped>
body {
    font-family: sans-serif;
    display: flex;
    flex-direction: column;
    margin: 20px;
}

header {
    width: 100%;
    text-align: center;
    margin-bottom: 20px;
}

main {
    width: 100%;
}

.categories {
    margin-bottom: 20px;
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
}

.category-btn {
    padding: 10px 20px;
    border: 1px solid #ccc;
    background-color: #f0f0f0;
    cursor: pointer;
}

.category-btn.active {
    background-color: #ddd;
}

.menu {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
    gap: 20px;
}

.menu-item {
    border: 1px solid #ccc;
    padding: 15px;
    text-align: center;
}

.menu-item img {
    max-width: 100%;
    height: auto;
    margin-bottom: 10px;
}

.menu-item h3 {
    margin-top: 0;
}

.add-to-cart-btn {
    background-color: #4CAF50;
    color: white;
    padding: 10px 15px;
    border: none;
    cursor: pointer;
}

.cart {
    width: 100%;
    border-left: none;
    padding-left: 0;
    margin-top: 20px;
}

.cart-items {
    list-style: none;
    padding: 0;
}

.cart-item {
    display: flex;
    justify-content: space-between;
    margin-bottom: 10px;
}

#checkout-btn {
    background-color: #008CBA;
    color: white;
    padding: 15px 20px;
    border: none;
    width: 100%;
    cursor: pointer;
}

@media (min-width: 768px) {
    body {
        flex-direction: row;
        justify-content: space-between;
    }

    main {
        width: 70%;
    }

    .cart {
        width: 25%;
        border-left: 1px solid #ccc;
        padding-left: 20px;
        margin-top: 0;
    }

    .menu {
        grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    }
}
</style>
