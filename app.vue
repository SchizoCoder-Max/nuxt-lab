<template>
  <div>
    <NuxtLayout />
    <NuxtPage />
  </div>
</template>

<script setup>
const route = useRoute()

useHead({
  title: route.meta.title || 'Мій сайт'
})
</script>
