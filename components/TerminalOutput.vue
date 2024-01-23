<script setup lang="ts">
import 'xterm/css/xterm.css'
import { Terminal } from 'xterm'

const props = defineProps<{
  stream?: ReadableStream
}>()

const root = ref<HTMLDivElement>()

const terminal = new Terminal()

const stream = new WritableStream({
  write(chunk) {
    terminal.write(chunk)
  },
})

watch(() => props.stream, (s) => {
  s?.pipeTo(stream)
}, { immediate: true })
onMounted(() => {
  terminal.open(root.value!)

  // props.stream.pipeTo(stream)
})
</script>

<template>
  <div ref="root">
    <slot />
  </div>
</template>

<style scoped>

</style>
