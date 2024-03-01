<script setup>
//Компонент для главной страницы
import CardList from '../components/CardList.vue'
import axios from 'axios'
import debounce from 'lodash.debounce'
import { onMounted } from 'vue'
import { inject } from 'vue'
import { reactive, ref, watch } from 'vue'

const { cart, addToCart, removeFromCart } = inject('cart')
const items = ref([])

const filters = reactive({
  sortBy: 'title',
  searchQuery: ''
})

const onChangeList = (event) => {
  filters.sortBy = event.target.value
}
//Это можно сказать таргет цель для пойска инпута с функцией что бы пойск началься через какое то милисекундное время что бы не было перегрузок на сервер
const onChangeSearchInput = debounce((event) => {
  filters.searchQuery = event.target.value
}, 800)

const onClickAddPlus = (item) => {
  if (!item.isAdded) {
    addToCart(item)
  } else {
    removeFromCart(item)
  }
}

//Функци для добавления в закладки
const addToFavorite = async (item) => {
  try {
    //Сначала проверяем является ли элемент избранным если нет то создается обьект который содержит информацию

    if (!item.isFavorite) {
      const obj = {
        favorites_id: item.id,
        item
      }
      //О добавляемом элементе и отправляет POST запрос и после успешного добавления начальное состояние элемента обновляется и его статус IsFavorite из False на true
      //Также обновляется item.favoriteId с идентификатором избранного, который возвращается сервером
      item.isFavorite = true

      const { data } = await axios.post('https://e59ff3ed5d4be313.mokky.dev/favourites', obj)

      item.favoriteId = data.id
    } else {
      //Если элемент уже является избранным, то отправляется DELETE-запрос на сервер для удаления элемента из избранного
      item.isFavorite = false
      await axios.delete(`https://e59ff3ed5d4be313.mokky.dev/favourites/${item.favoriteId}`)
      item.favoriteId = null
    }
  } catch (err) {
    console.log(err)
  }
}

//здесь в этой функций мы пишем что есть ли в нашем данных какой то айтем соответстующим с айди фаворит то стваим ему тру
const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get('https://e59ff3ed5d4be313.mokky.dev/favourites')
    items.value = items.value.map((item) => {
      const favorite = favorites.find((favorite) => favorite.favorites_id === item.id)
      //и если нету таких закладок то вернем тот же товар ничего не меняя
      if (!favorite) {
        return item
      }
      //А если есть то вернем новый обьект в котором есть все свойство и даем тру с закладкам и дам им айди новый
      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id
      }
    })
  } catch (e) {
    console.log(err)
  }
}

//Асинхронная Функция что бы вытащить данные с бекенда с помощью Аксиоса
//сначала пишем параметры функций и пробуем ее если высетиться ошибка то сразу же ловим
//а в самой функций пишем переменную дата и деструктизируем его потом через get запрос берем данные с бекенда

//В эту же функция написал методы для пойска товаров по заголовку и фильтрацию по ценам
//в начале sortBy и searchQuery закручиваем в реактив что бы была динамическая функция  затем пишем в переменную
//параметр значение и оно будет искать по тайтлу товара а товары будет смотреть в нашем массиве данных и через значение в разметке в HTML
//через price и -price оно будет давать товары по найменьшей и найбольшей цене
//searchQuery это метод что бы искать товары по имени и если в инпут есть соответстующее слово то функция начнет искать соответствующий товар

//еще в этой функций есть метод при которым мы можем давать товарам начальные параметры для закладок товаров
const fetchItems = async () => {
  try {
    const params = {
      sortBy: filters.sortBy
    }
    // Если есть строка поиска, добавляем параметр title
    if (filters.searchQuery) {
      params.title = `*${filters.searchQuery}*`
    }

    const { data } = await axios.get('https://e59ff3ed5d4be313.mokky.dev/sneakersMockData', {
      params
    })
    //После получения данных мы вшываем их в реактивный массив items
    //с помощью метода map мы получим весь список перед тем как их сохранить

    items.value = data.map((obj) => ({
      //берем все что есть в данных

      ...obj,
      //и даем начальное состояние для закладок
      isFavorite: false,

      favoriteId: null,
      isAdded: false
    }))
  } catch (eкк) {
    console.log(err)
  }
}

//В этом хуке мы делаем все асинхнронно при рендере
//сначала мы запросим все кроссовки и потом мы рендерим товары для закладок и устанавливаем начальное состояние
//и после этого делается функция fecthFavorites
onMounted(async () => {
  const localCart = localStorage.getItem('cart')
  cart.value = localCart ? JSON.parse(localCart) : []
  await fetchItems()
  await fetchFavorites()

  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cart.value.some((cartItem) => cartItem.id === item.id)
  }))
})

//Если корзина изменилось то все айтемы делаем начальное состояние галочки в фолс
watch(cart, () => {
  items.value = items.value.map((item) => ({
    ...item,
    isAdded: false
  }))
})

//И при каждом изменений с помощью хука watch мы следим за этим и выполняем запрос на бекенд
watch(filters, fetchItems)
</script>
<template>
  <div>
    <div class="flex justify-between items-center">
      <h2 class="text-3xl font-bold mb-8">Все Кроссовки</h2>

      <div class="flex gap-4">
        <select @click="onChangeList" class="py-2 px-3 border rounded-md outline-none">
          <option value="name">По Названию</option>
          <option value="-price">По Цене(Дороже:)</option>
          <option value="price">По Цене(Дешевле:)</option>
        </select>

        <div class="relative">
          <img class="absolute left-3 top-3" src="/search.svg" />
          <input
            @input="onChangeSearchInput"
            class="border rounded-md py-2 pl-11 pr-4 outline-none focus:border-gray-400"
            type="text"
            placeholder="Пойск ..."
          />
        </div>
      </div>
    </div>

    <div class="mt-10">
      <!--Передаем товары на главную страницу через в-бинд-->
      <CardList :items="items" @add-to-favorite="addToFavorite" @add-to-cart="onClickAddPlus" />
    </div>
  </div>
</template>
