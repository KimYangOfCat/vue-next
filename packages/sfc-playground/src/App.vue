<script setup lang="ts">
import Header from './Header.vue'
import { Repl, ReplStore, SFCOptions } from '@vue/repl'
import Monaco from '@vue/repl/monaco-editor'
import { ref, watchEffect } from 'vue'

const setVH = () => {
  document.documentElement.style.setProperty('--vh', window.innerHeight + `px`)
}
window.addEventListener('resize', setVH)
setVH()

const useDevMode = ref(false)
const useSSRMode = ref(false)

let hash = location.hash.slice(1)
if (hash.startsWith('__DEV__')) {
  hash = hash.slice(7)
  useDevMode.value = true
}
if (hash.startsWith('__SSR__')) {
  hash = hash.slice(7)
  useSSRMode.value = true
}

const store = new ReplStore({
  serializedState: hash,
  defaultVueRuntimeURL: import.meta.env.PROD
    ? `${location.origin}/vue.runtime.esm-browser.js`
    : `${location.origin}/src/vue-dev-proxy`,
  defaultVueServerRendererURL: import.meta.env.PROD
    ? `${location.origin}/server-renderer.esm-browser.js`
    : `${location.origin}/src/vue-server-renderer-dev-proxy`
})

// enable experimental features
const sfcOptions: SFCOptions = {
  script: {
    inlineTemplate: !useDevMode.value,
    isProd: !useDevMode.value,
    reactivityTransform: true,
    defineModel: true
  },
  style: {
    isProd: !useDevMode.value
  },
  template: {
    isProd: !useDevMode.value
  }
}

// persist state
watchEffect(() => {
  const newHash = store
    .serialize()
    .replace(/^#/, useSSRMode.value ? `#__SSR__` : `#`)
    .replace(/^#/, useDevMode.value ? `#__DEV__` : `#`)
  history.replaceState({}, '', newHash)
})

function toggleDevMode() {
  const dev = (useDevMode.value = !useDevMode.value)
  sfcOptions.script!.inlineTemplate =
    sfcOptions.script!.isProd =
    sfcOptions.template!.isProd =
    sfcOptions.style!.isProd =
      !dev
  store.setFiles(store.getFiles())
}

function toggleSSR() {
  useSSRMode.value = !useSSRMode.value
  store.setFiles(store.getFiles())
}
</script>

<template>
  <Header
    :store="store"
    :dev="useDevMode"
    :ssr="useSSRMode"
    @toggle-dev="toggleDevMode"
    @toggle-ssr="toggleSSR"
  />
  <Repl
    :editor="Monaco"
    @keydown.ctrl.s.prevent
    @keydown.meta.s.prevent
    :ssr="useSSRMode"
    :store="store"
    :showCompileOutput="true"
    :autoResize="true"
    :sfcOptions="sfcOptions"
    :clearConsole="false"
  />
</template>

<style>
.dark {
  color-scheme: dark;
}

body {
  font-size: 13px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen,
    Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  margin: 0;
  --base: #444;
  --nav-height: 50px;
}

.vue-repl {
  height: calc(var(--vh) - var(--nav-height)) !important;
}

button {
  border: none;
  outline: none;
  cursor: pointer;
  margin: 0;
  background-color: transparent;
}
</style>
