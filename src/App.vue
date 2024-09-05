<script setup lang="ts">
// https://github.com/vueuse/head
// you can use this to manipulate the document head in any components,
// they will be rendered correctly in the html results with vite-ssg
useHead({
  title: 'App Store',
  meta: [
    {
      name: 'description',
      content: 'App Store with Vite + Vue 3 + Unocss',
    },
  ],
})
</script>

<template>
  <RouterView />
</template>
