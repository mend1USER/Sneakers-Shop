<script setup>
defineProps({
  imageUrl: String,
  title: String,
  description: String
})
</script>
<template>
  <div class="flex flex-col items-center text-center max-w-72 mx-autos">
    <img height="70" width="70" :src="imageUrl" alt="Info Image" />
    <h2 class="text-2 mt-4 font-medium">{{ title }}</h2>
    <p class="text-gray-400 mt-2">{{ description }}</p>
  </div>
</template>
