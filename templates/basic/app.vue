<script setup lang="ts">
if (process.client) {
  window.addEventListener('message', () => {
    console.log('got message', e)
  })
}
</script>

<template>
  <div>
    hello
  </div>
</template>

<style scoped>

</style>
