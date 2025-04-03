<script setup lang="ts">
import { ref } from 'vue'
import { useFetch } from '#app'

useHead({ title: 'Список продуктів' })

const { data: res, pending, error } = await useFetch('https://dummyjson.com/products?limit=100')

const products = ref(res.value?.products || [])
</script>

<template>
  <div class="p-4">
    <h1 class="text-2xl font-bold mb-4">Список продуктів</h1>

    <UTable
        v-if="!pending && !error"
        :rows="products"
        :columns="[
        { key: 'thumbnail', label: 'Фото' },
        { key: 'title', label: 'Назва', sortable: true },
        { key: 'description', label: 'Опис' },
        { key: 'price', label: 'Ціна', sortable: true },
        { key: 'rating', label: 'Оцінка', sortable: true },
        { key: 'brand', label: 'Бренд' },
        { key: 'category', label: 'Категорія' }
      ]"
        :paginable="true"
        :page-size="10"
        :searchable="true"
    >
      <template #thumbnail-data="{ row }">
        <img :src="row.thumbnail" width="100" height="100" />
      </template>

      <template #rating-data="{ row }">
        <span :class="row.rating < 4.5 ? 'text-red-500' : 'text-green-600'">
          {{ row.rating }}
        </span>
      </template>
    </UTable>

    <div v-if="pending">Завантаження...</div>
    <div v-if="error">Сталася помилка 😞</div>
  </div>
</template>
