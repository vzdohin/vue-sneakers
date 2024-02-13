<script setup>
import axios from 'axios'
import { ref, inject } from 'vue'

import CartListItem from './CartListItem.vue'
import InfoBlock from './InfoBlock.vue'

const props = defineProps({
  totalPrice: Number,
  vatPrice: Number
})

const { cart, closeDrawer } = inject('cart')

const isCreatingOrder = ref(false)
const orderId = ref(null)

const createOrder = async () => {
  try {
    isCreatingOrder.value = true
    const { data } = await axios.post('https://8eb0adbfe66e34e4.mokky.dev/orders', {
      items: cart.value,
      totalPrice: props.totalPrice.value
    })

    cart.value = []
    orderId.value = data.id
    // closeDrawer()

    return data
  } catch (err) {
    console.log(err)
    alert('Что-то пошло не так, попробуйте снова.')
  } finally {
    isCreatingOrder.value = false
  }
}
</script>

<template>
  <div class="fixed top-0 left-0 h-full w-full bg-black z-10 opacity-70" @click="closeDrawer"></div>
  <div class="bg-white w-96 h-full fixed right-0 top-0 z-20 p-8">
    <div class="flex items-center gap-5 mb-8">
      <svg
        @click="closeDrawer"
        class="opacity-30 cursor-pointer rotate-180 hover:opacity-100 transition hover:-translate-x-1"
        width="16"
        height="14"
        viewBox="0 0 16 14"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path
          d="M1 7H14.7143"
          stroke="black"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
        />
        <path
          d="M8.71436 1L14.7144 7L8.71436 13"
          stroke="black"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
        />
      </svg>

      <h2 class="text-2xl font-bold">Корзина</h2>
    </div>

    <div class="flex h-full items-center" v-if="!totalPrice || orderId">
      <InfoBlock
        v-if="!totalPrice && !orderId"
        title="Корзина пустая"
        description="Добавьте хоть одну пару кроссовок, чтобы сделать заказ."
        image-url="../../package-icon.png"
      />
      <InfoBlock
        v-if="orderId"
        title="Заказ оформлен"
        :description="`Ваш заказ №${orderId} скоро будет передан курьерской доставкой.`"
        image-url="../../order-success-icon.png"
      />
    </div>

    <CartListItem v-if="totalPrice" />

    <div v-if="totalPrice" class="flex flex-col gap-4 mt-7">
      <div class="flex gap-2">
        <span>Итого:</span>
        <div class="flex-1 border-b border-dashed"></div>
        <b>{{ totalPrice }} руб.</b>
      </div>
      <div class="flex gap-2">
        <span>Налог 20%(НДС):</span>
        <div class="flex-1 border-b border-dashed"></div>
        <b>{{ vatPrice }} руб.</b>
      </div>
      <button
        :disabled="totalPrice ? false : true"
        @click="createOrder"
        class="mt-4 disabled:bg-slate-300 bg-lime-500 w-full rounded-xl py-3 text-white transition hover:bg-lime-600 active:bg-lime-700 cursor-pointer"
      >
        Оформить заказ
      </button>
    </div>
  </div>
</template>
