<script setup>
import { ref, watch, provide, computed } from 'vue'
import Drawer from './components/Drawer.vue'
import Header from './components/Header.vue'

//Corzina
const cart = ref([])
const drawerOpen = ref(false)

//Функций для корзины по типу тотал цены товаров при заказе
//Открывание и закрывание корзины и удаление карты в корзине
const totalPrice = computed(() => cart.value.reduce((acc, item) => acc + item.price, 0))
const vatPrice = computed(() => Math.round((totalPrice.value * 5) / 100))

const closeDrawer = () => {
  drawerOpen.value = false
}

const openDrawer = () => {
  drawerOpen.value = true
}

const addToCart = (item) => {
  cart.value.push(item)
  item.isAdded = true
}

const removeFromCart = (item) => {
  cart.value.splice(cart.value.indexOf(item), 1)
  item.isAdded = false
}

watch(
  cart,
  () => {
    localStorage.setItem('cart', JSON.stringify(cart.value))
  },
  { deep: true }
)

//И с помощью провайд иопортируем в дочерние компоненты эти функций
//просто импортируя через inject в комопненты что бы не было Props Drilling
provide('cart', {
  cart,
  closeDrawer,
  openDrawer,
  addToCart,
  removeFromCart
})
//Corzina
</script>

<template>
  <div class="w-4/5 m-auto bg-white rounded-xl shadow-xl mt-14">
    <Drawer
      v-if="drawerOpen"
      :total-price="totalPrice"
      :vat-price="vatPrice"
      @createOrder="createOrder"
    />

    <Header :total-price="totalPrice" @open-drawer="openDrawer" />

    <div class="p-10">
      <router-view></router-view>
    </div>
  </div>
</template>
