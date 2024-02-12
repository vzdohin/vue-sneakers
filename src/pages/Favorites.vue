<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

import CardList from '../components/CardList.vue'

const favorites = ref([])

onMounted(async () => {
  try {
    const { data } = await axios.get(
      'https://8eb0adbfe66e34e4.mokky.dev/favorites?_relations=items'
    )

    favorites.value = data.map((obj) => obj.item)
    console.log(data)
  } catch (err) {
    console.log(err)
  }
})
</script>

<template>
  <h2 class="text-3xl font-bold mb-8">Мои избранные кроссовки</h2>
  <CardList :items="favorites" is-favorites />
</template>
