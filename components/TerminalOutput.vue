<script setup lang="ts">
import 'xterm/css/xterm.css'
import { Terminal } from 'xterm'

const props = defineProps<{
  stream?: ReadableStream
}>()

const root = ref<HTMLDivElement>()

const terminal = new Terminal()

// const stream = new WritableStream({
//   write(chunk) {
//     terminal.write(chunk)
//   },
// })

watch(
  () => props.stream,
  (s) => {
    if (!s)
      return
    // s?.pipeTo(stream)
    const reader = s.getReader()
    function read() {
      reader.read().then(({ done, value }) => {
        terminal.write(value)
        if (!done)
          read()
      })
    }
    read()
  },
  {
    flush: 'sync',
    immediate: true,
  },
)
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
