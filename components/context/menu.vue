<script setup lang="ts">
import { useContextMenuStore } from '~/stores/context-menu'

defineProps<{
  isVisible?: boolean
  items?: {
    label: string
    action: string
  }[]
}>()

defineEmits<{
  onAction: [action: string]
  onClose: []
}>()

const contextMenuEl = ref()
// const { isVisible, position, close, items } = useContextMenu()

const contextMenuStore = useContextMenuStore()
onClickOutside(contextMenuEl, contextMenuStore.close)
</script>

<template>
  <ul v-if="contextMenuStore.isVisible" ref="contextMenuEl" class="fixed top-[--y] left-[--x] z-100 z-10 py-6px bg-dark-700 rounded border-y border-t-white/5 border-b-black text-xs p-0 overflow-hidden w-200px" :style="{ '--x': `${contextMenuStore.position.x}px`, '--y': `${contextMenuStore.position.y}px` }">
    <li v-for="(item, index) in contextMenuStore.items" :key="index" class="block p-0">
      <button type="button" class="block w-full px-4 h-24px hover:bg-green-500 text-white transition">
        <span class="flex justify-between">
          <span>{{ label }}</span>
          <span v-if="hotkey" class="">{{ hotkey }}</span>
        </span>
      </button>
    </li>
  </ul>
</template>

<style scoped>

</style>
