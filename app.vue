<template>
  <NuxtLayout>
    <NuxtPage />
  </NuxtLayout>
</template>


<script setup>
const route = useRoute()

useHead({
  title: route.meta.title || 'Мій сайт'
})
</script>
