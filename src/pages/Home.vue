<script setup>
import { reactive, watch, ref, onMounted } from 'vue'
import debounce from 'lodash.debounce'
import axios from 'axios'
import { inject } from 'vue'
import CardList from '../components/CardList.vue'

const { cart, addToCart, removeFromCart } = inject('cart')

const filters = reactive({
  sortBy: 'title',
  searchQuery: ''
})

const items = ref([])

const onClickAddPlus = (item) => {
  if (!item.isAdded) {
    addToCart(item)
  } else {
    removeFromCart(item)
  }
}
// сортировка и поиск
const onChangeQuery = (e) => {
  filters.sortBy = e.target.value
}
const onChangeSearchInput = debounce((e) => {
  filters.searchQuery = e.target.value
}, 150)
const addToFavorite = async (item) => {
  try {
    if (!item.isFavorite) {
      const obj = {
        item_id: item.id
      }
      const { data } = await axios.post('https://8eb0adbfe66e34e4.mokky.dev/favorites', obj)
      item.isFavorite = true
      item.favoriteId = data.id
    } else {
      await axios.delete(`https://8eb0adbfe66e34e4.mokky.dev/favorites/${item.favoriteId}`)
      item.isFavorite = false
      item.favoriteId = null
    }
  } catch (err) {
    console.log(err)
  }
}
const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get('https://8eb0adbfe66e34e4.mokky.dev/favorites')
    items.value = items.value.map((item) => {
      const favorite = favorites.find((favorite) => favorite.item_id === item.id)
      if (!favorite) {
        return item
      }
      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id
      }
    })
  } catch (e) {
    console.log(e)
  }
}

const fetchItems = async () => {
  try {
    const params = {
      sortBy: filters.sortBy
    }

    if (filters.searchQuery) {
      params.title = `*${filters.searchQuery}*`
    }
    const { data } = await axios.get('https://8eb0adbfe66e34e4.mokky.dev/items', {
      params
    })
    items.value = data.map((obj) => ({
      ...obj,
      isFavorite: false,
      favoriteId: null,
      isAdded: false
    }))
  } catch (e) {
    console.log(e)
  }
}
onMounted(async () => {
  // список корзины в локалсторадже
  const localCart = localStorage.getItem('cart')
  cart.value = localCart ? JSON.parse(localCart) : []

  await fetchItems()
  await fetchFavorites()

  // список значков добавлено
  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cart.value.some((cartItem) => cartItem.id === item.id)
  }))
})
// очистка добавленых в корзину после отправки заказа
watch(cart, () => {
  items.value = items.value.map((item) => ({
    ...item,
    isAdded: false
  }))
})

watch(filters, fetchItems)
</script>

<template>
  <div class="flex justify-between items-center">
    <h2 class="text-3xl font-bold mb-8">Все кроссовки</h2>

    <div class="flex gap-4">
      <select @change="onChangeQuery" class="py-2 px-3 border rounded-md outline-none">
        <option value="name">По названию</option>
        <option value="price">По цене (по возрастанию)</option>
        <option value="-price">По цене (по убыванию)</option>
      </select>

      <div class="relative">
        <img class="absolute left-4 top-3" src="/public/search.svg" />
        <input
          @input="onChangeSearchInput"
          class="border rounded-md py-2 pl-11 pr-4 outline-none focus:border-gray-400"
          placeholder="Поиск"
        />
      </div>
    </div>
  </div>
  <section class="mt-10">
    <CardList :items="items" @add-to-favorite="addToFavorite" @add-to-cart="onClickAddPlus" />
  </section>
</template>
