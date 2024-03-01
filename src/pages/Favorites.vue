<script setup>
import { onMounted, ref } from 'vue'
import axios from 'axios'
import CardList from '../components/CardList.vue'

const favorites = ref([])

//Функция для отправления и рендера товаров на закладках
//при первом рендере пишем что будем получать данные с бека с помощью одной фичи сайта где он сам будет фильтровать и искать товары по айдике
//Из за этого мы дали имя обьекту item_id и с помощью relations метода в сайте Mokky dev сайт будет выдавать нам массив с данным соответсвующего товара по айди
onMounted(async () => {
  try {
    const { data } = await axios.get(
      'https://e59ff3ed5d4be313.mokky.dev/favourites?_relations=sneakersMockData'
    )
    console.log(data)
    favorites.value = data.map((obj) => obj.item)
  } catch (err) {
    console.log(err)
  }
})
</script>

<template>
  <div>
    <h2 class="text-3xl font-bold mb-8">Мой Закладки</h2>

    <CardList :items="favorites" is-favorites />
  </div>
</template>
