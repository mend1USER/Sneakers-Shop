<script setup>
//Компонент корзины для товаров
import DrawerHead from './DrawerHead.vue'
import InfoBlock from './InfoBlock.vue'
import CardListItem from './CardListItem.vue'
import axios from 'axios'
import { provide, ref, watch, computed, inject } from 'vue'

const emit = defineEmits(['createOrder'])
const props = defineProps({
  totalPrice: Number,
  vatPrice: Number,
  buttonDisabled: Boolean
})
const isCreating = ref(false)
const orderId = ref(null)

const { cart, closeDrawer } = inject('cart')

//Устанавливает isCreating.value в true перед отправкой запроса на создание заказа.
const createOrder = async () => {
  try {
    isCreating.value = true
    //Использует Axios для отправки POST-запроса по URL 'https://e59ff3ed5d4be313.mokky.dev/orders'
    //с данными { items: cart.value, totalPrice: props.totalPrice.value }
    const { data } = await axios.post('https://e59ff3ed5d4be313.mokky.dev/orders', {
      items: cart.value,
      totalPrice: props.totalPrice.value
    })

    //Затем устанавливает cart.value в пустой массив и присваивает id из ответа переменной orderId.value.
    cart.value = []

    //Ожидает, что ответ содержит свойство data и присваивает его переменной.
    orderId.value = data.id
  } catch (err) {
    console.log(err)
  } finally {
    //Блок finally устанавливает isCreating.value обратно в false, независимо от успешности или неуспешности запроса.
    //Вернет начальное состояние корзины без товаров
    isCreating.value = false
  }
}
const cartIsEmpty = computed(() => cart.value.length === 0)

//Вычисляемое свойство если корзина пусто то кнопка должна будет отключиться
const buttonDisabled = computed(() => isCreating.value || cartIsEmpty.value)
</script>

<template>
  <div>
    <div class="fixed top-0 left-0 h-full w-full bg-black z-10 opacity-70"></div>

    <div class="bg-white w-96 h-full fixed right-0 top-0 z-20 p-8">
      <DrawerHead />

      <div v-if="!totalPrice || orderId" class="flex h-full justify-center">
        <InfoBlock
          v-if="!totalPrice && !orderId"
          class="flex items-center justify-center"
          title="Ваша Корзина Пустая"
          description="Добавьте хотя бы одну пару кроссовок, чтобы сделать заказ"
          image-url="/package-icon.png"
        />
        <InfoBlock
          v-if="orderId"
          class="flex items-center justify-center"
          title="Заказ Оформлен"
          :description="`Ваш заказ ${orderId} скоро будет передан курьерской доставке`"
          image-url="/order-success-icon.png"
        />
      </div>

      <div v-else>
        <CardListItem v-if="totalPrice" />

        <div v-if="totalPrice" class="flex flex-col gap-4 mt-7">
          <div class="flex gap-2">
            <span>Итого:</span>
            <div class="flex-1 border-b border-dashed"></div>
            <b>{{ totalPrice }}$</b>
          </div>

          <div class="flex gap-2">
            <span>Налог%</span>
            <div class="flex-1 border-b border-dashed"></div>
            <b>{{ vatPrice }}$</b>
          </div>
        </div>
        <button
          :disabled="buttonDisabled"
          @click="createOrder"
          class="mt-7 transition bg-lime-500 w-full rounded-xl py-3 disabled:bg-slate-400 text-white hover:bg-lime-700"
        >
          Оформить Заказ
        </button>
      </div>
    </div>
  </div>
</template>
