<template>
  <div class="container">
    <h1 class="title">Список продуктів</h1>

    <input
        v-model="searchQuery"
        type="text"
        placeholder="Пошук за назвою, брендом, категорією, описом, ціною або оцінкою"
        class="search-input"
    />

    <div class="table-wrapper">
      <table class="product-table">
        <thead>
        <tr>
          <th @click="sortBy('title')">Назва</th>
          <th @click="sortBy('description')">Опис</th>
          <th @click="sortBy('price')">Ціна</th>
          <th @click="sortBy('rating')">Оцінка</th>
          <th @click="sortBy('brand')">Бренд</th>
          <th @click="sortBy('category')">Категорія</th>
          <th>Фото</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="product in paginatedProducts" :key="product.id">
          <td>{{ product.title }}</td>
          <td>{{ product.description }}</td>
          <td>{{ product.price }}$</td>
          <td :class="product.rating < 4.5 ? 'low-rating' : 'high-rating'">
            {{ product.rating }}
          </td>
          <td>{{ product.brand }}</td>
          <td>{{ product.category }}</td>
          <td>
            <img :src="product.thumbnail" alt="product" class="product-image" />
          </td>
        </tr>
        </tbody>
      </table>
    </div>

    <div class="pagination">
      <button @click="prevPage" :disabled="page === 1">⬅ Назад</button>
      <span>Сторінка {{ page }} з {{ totalPages }}</span>
      <button @click="nextPage" :disabled="page === totalPages">➡ Далі</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'

const products = ref([])
const searchQuery = ref('')
const sortKey = ref('')
const sortAsc = ref(true)
const page = ref(1)
const perPage = 5

onMounted(async () => {
  const res = await fetch('https://dummyjson.com/products')
  const json = await res.json()
  products.value = json.products
})

const normalize = (str) => {
  return String(str)
      .toLowerCase()
      .normalize('NFD')
      .replace(/[\u0300-\u036f]/g, '')
      .replace(/[^\w\s.]/gi, '')
      .trim()
}

const filteredProducts = computed(() => {
  const query = normalize(searchQuery.value)
  if (!query) return products.value

  const terms = query.split(/\s+/)

  return products.value.filter(product => {
    const fields = [
      product.title,
      product.description,
      product.brand,
      product.category,
      String(product.price),
      String(product.rating)
    ].map(normalize)

    return terms.every(term =>
        fields.some(field => field.includes(term))
    )
  })
})

const sortedProducts = computed(() => {
  const items = [...filteredProducts.value]
  if (!sortKey.value) return items

  return items.sort((a, b) => {
    const aVal = a[sortKey.value]
    const bVal = b[sortKey.value]
    if (typeof aVal === 'number' && typeof bVal === 'number') {
      return sortAsc.value ? aVal - bVal : bVal - aVal
    } else {
      return sortAsc.value
          ? String(aVal).localeCompare(String(bVal))
          : String(bVal).localeCompare(String(aVal))
    }
  })
})

watch([searchQuery, sortKey, sortAsc], () => {
  page.value = 1
})

const totalPages = computed(() =>
    Math.ceil(sortedProducts.value.length / perPage)
)

const paginatedProducts = computed(() => {
  const start = (page.value - 1) * perPage
  return sortedProducts.value.slice(start, start + perPage)
})

const nextPage = () => {
  if (page.value < totalPages.value) page.value++
}

const prevPage = () => {
  if (page.value > 1) page.value--
}

const sortBy = (key) => {
  if (sortKey.value === key) {
    sortAsc.value = !sortAsc.value
  } else {
    sortKey.value = key
    sortAsc.value = true
  }
}
</script>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
  font-family: 'Segoe UI', sans-serif;
}

.title {
  text-align: center;
  font-size: 2.2rem;
  margin-bottom: 2rem;
  color: #222;
}

.search-input {
  width: 100%;
  padding: 12px 16px;
  font-size: 1rem;
  margin-bottom: 24px;
  border: 1px solid #ccc;
  border-radius: 8px;
  outline: none;
  box-shadow: 0 1px 3px rgba(0,0,0,0.05);
}

.search-input:focus {
  border-color: #0070f3;
  box-shadow: 0 0 0 3px rgba(0, 112, 243, 0.2);
}

.table-wrapper {
  overflow-x: auto;
  border-radius: 10px;
  background-color: white;
  box-shadow: 0 2px 6px rgba(0,0,0,0.05);
}

.product-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.95rem;
}

.product-table th,
.product-table td {
  padding: 14px 12px;
  border-bottom: 1px solid #eee;
  text-align: center;
}

.product-table th {
  background-color: #f4f6f8;
  cursor: pointer;
  font-weight: 600;
  color: #333;
}

.product-table tbody tr:hover {
  background-color: #f9f9f9;
  transition: background-color 0.2s ease-in-out;
}

.low-rating {
  color: #e53e3e;
  font-weight: 600;
}

.high-rating {
  color: #38a169;
  font-weight: 600;
}

.product-image {
  width: 80px;
  height: 80px;
  object-fit: cover;
  border-radius: 6px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.1);
}

.pagination {
  margin-top: 30px;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
}

.pagination button {
  padding: 10px 20px;
  border: none;
  background-color: #0070f3;
  color: white;
  border-radius: 6px;
  font-size: 0.95rem;
  cursor: pointer;
  transition: 0.2s;
}

.pagination button:hover:not(:disabled) {
  background-color: #0059c1;
}

.pagination button:disabled {
  background-color: #bbb;
  cursor: not-allowed;
}
</style>
