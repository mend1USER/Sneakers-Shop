<script setup>
//Компонент отдельных карточек в пропсами что бы их импортировать и
//экспортированные емиты что бы их испортировать было легче
import Card from './Card.vue'

//Пропсы для прокидывания через компоненты
defineProps({
  items: Array,
  isFavorites: Boolean
})

//Функций которые будем импортироваться через Emit
const emit = defineEmits(['addToFavorite', 'addToCart'])

//Функция через Emit
const onClickAdd = () => {
  emit('addToFavorite', item)
}
</script>

<template>
  <!--разметка для отдлеьных карточексо всеми забинденными данными внутри них и тоже закрепленные функций-->
  <div class="p-10">
    <!--Проходимся по всем массивам через директиву v-for и с помощью него рендерим все товары
    и пишем динамические данные-->
    <div class="grid grid-cols-4 gap-5" v-auto-animate>
      <Card
        v-for="item in items"
        :id="item.id"
        :key="item.id"
        :image-url="item.imageUrl"
        :title="item.title"
        :price="item.price"
        :is-favorite="item.isFavorite"
        :on-click-favorite="isFavorites ? null : () => emit('addToFavorite', item)"
        :on-click-add="isFavorites ? null : () => emit('addToCart', item)"
        :is-added="item.isAdded"
      />
    </div>
  </div>
</template>
