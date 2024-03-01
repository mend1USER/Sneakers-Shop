<script setup>
//Компонент карточки товаров и его информаций сначала пишем пропсы
//чтобы было легче работать с ними в других дочерних компонентах 

const props = defineProps({
  id: Number,
  title: String,
  imageUrl: String,
  price: Number,
  isFavorite: Boolean,
  isAdded: Boolean,
  onClickAdd: Function,
  onClickFavorite: Function
})
</script>

<template>
  <!--Разметка для компонента карточек, сперва пишем стили и разметки для информаций внутри 
  карточки с иконками с реактивностью данных тайтлов и цены на товары а также директивы для событий на кнопки 
      v-if="onClickFavorite"         v-if="onClickAdd"
а вот эти директивы нужны что бы сайт знал когда именно показывать кнопки для добавления в избранное и когда их дизэйблить
-->
  <div
    class="relative bg-white border-slate-100 rounded-3xl p-8 cursor-pointer transition hover:-translate-y-2 hover:shadow-xl"
  >
    <img
    
      v-if="onClickFavorite"
      class="absolute top-0 left-8"
      :src="!isFavorite ? '/like-1.svg' : '/like-2.svg'"
      alt="Like"
      @click="onClickFavorite"
    />
    <img :src="imageUrl" alt="Sneakers" />
    <p class="mt-2">{{ title }}</p>

    <div class="flex justify-between mt-5">
      <div class="flex flex-col">
        <span class="text-slate-400">Цена:</span>
        <b>{{ price }}$</b>
      </div>
      <img
        v-if="onClickAdd"
        @click="onClickAdd"
        :src="!isAdded ? '/plus.svg' : '/checked.svg'"
        alt="Plus"
      />
    </div>
  </div>
</template>
