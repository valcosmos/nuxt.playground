<script setup lang="ts">
const { x, y } = useMouse()
const color = useColorMode()

function toggleDark() {
  color.value = color.value === 'dark' ? 'light' : 'dark'
}
</script>

<template>
  <div>
    hello world {{ x }} {{ y }}
    <button @click="toggleDark">
      {{ color.value }}
    </button>
  </div>
</template>

<style scoped>

</style>
