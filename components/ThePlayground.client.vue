<script setup lang="ts">
const iframe = ref<HTMLIFrameElement>()
const wcUrl = ref<string>()
type Status = 'init' | 'mount' | 'install' | 'start' | 'ready' | 'error'
const status = ref<Status>('init')
const error = shallowRef<{ message: string }>()
const stream = ref<ReadableStream>()
async function startDevServer() {
  const wc = await useWebContainer()
  status.value = 'mount'

  wc.on('server-ready', (port, url) => {
    console.log('server-ready', port, url)
    status.value = 'ready'
    wcUrl.value = url
  })

  wc.on('error', (err) => {
    status.value = 'error'
    error.value = err
  })

  wc.mount({
    'package.json': {
      file: {
        contents: JSON.stringify({
          private: true,
          scripts: {
            dev: 'nuxt dev',
          },
          dependencies: {
            nuxt: 'latest',
          },
        }, null, 2),
      },
    },
  })

  status.value = 'install'

  const installProcess = await wc.spawn('pnpm', ['install'])
  installProcess.output.pipeTo(stream)
  stream.value = installProcess.output

  const installExitCode = await installProcess.exit

  if (installExitCode !== 0) {
    status.value = 'error'
    error.value = {
      message: `Unable to run npm install, exit as ${installExitCode}`,
    }
    throw new Error('Unable to run npm install')
  }
  status.value = 'start'

  // `npm run dev`
  const devProcess = await wc.spawn('pnpm', ['run', 'dev'])
  // devProcess.output.pipeTo(stream)
  stream.value = devProcess.output
  wc.on('server-ready', (port, url) => (iframe.value!.src = url))
}

onMounted(startDevServer)

watchEffect(() => {
  if (iframe.value && wcUrl.value)
    iframe.value.src = wcUrl.value
})
</script>

<template>
  <div h-full w-full grid="~ rows-[2fr_1fr]">
    <iframe v-show="status === 'ready'" ref="iframe" w-full h-full />
    <div v-show="status !== 'ready'" w-full h-full flex="~ col items-center justify-center" capitalize text-lg>
      <div class="i-svg-spinners-90-ring-with-bg" />
      {{ status }}ing...
    </div>
    <TerminalOutput :stream="stream" />
  </div>
</template>

<style scoped>

</style>
