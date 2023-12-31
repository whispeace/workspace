<script setup lang="ts">
import { useContextMenuStore } from '~/stores/context-menu'

interface Position {
  x: number
  y: number
}

interface Size {
  width: number
  height: number
}

const props = withDefaults(defineProps<Partial<Props>>(), {
  x: 0,
  y: 0,
  width: 0,
  height: 0,
  focused: false,
})
const emits = defineEmits<{
  'onFocus': []
  'onChange': [props: Position & Size]
}>()
const frameElement = ref()
const unfocusedFrameElement = ref()

interface Props extends Position, Size {
  focused?: boolean
  draggable?: boolean
  resizable?: boolean
}

const showSizeManipulators = ref(true)

/**
 * Регистрирует новое событие и его слушатель на HTML-элементе.
 * @param {HTMLElement} element - HTML-элемент, к которому привязывается слушатель события.
 * @param {string} eventName - Имя события, на которое нужно слушать.
 * @param {EventListener} listener - Слушатель события для регистрации.
 * @param {AddEventListenerOptions} options - Дополнительные опции слушателя события (необязательно).
 * @returns {Function} Функция, которая, при вызове, удалит зарегистрированный слушатель события.
 */
function registerEvent(element: HTMLElement, eventName: string, listener: EventListener, options?: AddEventListenerOptions): Function {
  element.addEventListener(eventName, listener, options)
  return () => element.removeEventListener(eventName, listener, options)
}

const isFocused = toRef(props.focused)

function onFocus() {
  // emits('update:focused', true)
}

function onBlur() {
  // emits('update:focused', false)
}

const elementProps = toRef<Props>(props)

const position = ref<Position>({ x: 0, y: 0 })
const size = ref<Size>({ width: 0, height: 0 })
const direction = ref<string>()
const startPointPosition = ref<Position>()

function onMove(deltaPosition: Position = { x: 0, y: 0 }, deltaSize: Size = { width: 0, height: 0 }) {
  const width = size.value.width + deltaSize.width
  const height = size.value.height + deltaSize.height
  elementProps.value = {
    width: Math.round(Math.abs(width)),
    height: Math.round(Math.abs(height)),
    x: Math.round((position.value.x + deltaPosition.x) + Math.min(width, 0)),
    y: Math.round((position.value.y + deltaPosition.y) + Math.min(height, 0)),
  }
}

watch(elementProps, (newValue) => {
  emits('onChange', { x: newValue.x, y: newValue.y, width: newValue.width, height: newValue.height })
})

function start(e: PointerEvent) {
  const target = e.target as HTMLElement
  direction.value = target.dataset.direction
  position.value = {
    x: elementProps.value.x,
    y: elementProps.value.y,
  }
  size.value = {
    width: elementProps.value.width,
    height: elementProps.value.height,
  }
  startPointPosition.value = {
    x: e.clientX,
    y: e.clientY,
  }
  emits('onFocus')
}

function move(e: PointerEvent) {
  if (!startPointPosition.value)
    return
  if (!direction.value)
    return

  const offsetX = e.clientX - startPointPosition.value.x
  const offsetY = e.clientY - startPointPosition.value.y
  let x = 0
  let y = 0
  let width = 0
  let height = 0
  if (direction.value.includes('top')) {
    height = offsetY * -1
    y = offsetY
  }
  if (direction.value.includes(('left'))) {
    width = offsetX * -1
    x = offsetX
  }
  if (direction.value.includes(('right')))
    width = offsetX

  if (direction.value.includes('bottom'))
    height = offsetY

  if (direction.value.includes('all')) {
    x = offsetX
    y = offsetY
  }
  if (direction.value.includes('bottom'))
    height = offsetY

  if (direction.value === 'all')
    showSizeManipulators.value = false

  onMove({ x, y }, { width, height })
}

function end() {
  startPointPosition.value = undefined
  direction.value = undefined
  showSizeManipulators.value = true
}

const cssVariables = computed(() => ({
  '--width': `${elementProps.value.width}`,
  '--height': `${elementProps.value.height}`,
  '--x': `${elementProps.value.x}px`,
  '--y': `${elementProps.value.y}px`,
}))

useEventListener(frameElement, 'pointerdown', start, true)
useEventListener(window, 'pointermove', move, true)
useEventListener(window, 'pointerup', end, true)
// useEventListener(unfocusedFrameElement, 'pointerdown', start)

const contextMEnuStore = useContextMenuStore()
console.log(contextMEnuStore)

function openContext(e: MouseEvent) {
  console.log(e)

  contextMEnuStore.open({
    positionMenu: { x: e.clientX, y: e.clientY },
    itemsMenu: [
      // {
      //   label: 'Copy',
      //   hotkey: '⌘C',
      //   action: () => {}
      // },
      {
        label: 'Remove',
        action: () => {},
      },
    ],
  })
}
</script>

<template>
  <div ref="frameElement" class="frame absolute translate-x-[--x] translate-y-[--y]" :style="cssVariables" @contextmenu.prevent="openContext">
    <div data-direction="all" class="hover:border border-primary ">
      <div class="pointer-events-none">
        <slot />
      </div>
    </div>

    <div v-if="focused && showSizeManipulators" class="absolute top-0 left-0 grid grid-cols-[12px_1px_12px] grid-rows-[12px_1px_12px] place-content-center translate--12px">
      <div data-direction="top-left" class="after:border-primary cursor-nwse-resize after:translate-[--translate]" />
      <div data-direction="top" class="cursor-ns-resize border-b border-primary w-1px scale-x-[--width] origin-top-left" />
      <div data-direction="top-right" class="after:border-primary cursor-nesw-resize translate-x-[calc(var(--width,1)*1px-1px)] after:translate-y-[--translate] after:translate-x-[calc(var(--translate)*-1)]" />
      <div data-direction="left" class="cursor-ew-resize border-r border-primary  h-1px scale-y-[--height] origin-top-left" />
      <div data-direction="all" class="w-1px h-1px scale-x-[--width] scale-y-[--height] origin-top-left" />
      <div data-direction="right" class="cursor-ew-resize border-l border-primary h-1px scale-y-[--height] origin-top-left  translate-x-[calc(var(--width)*1px-1px)]" />
      <div data-direction="bottom-left" class="after:border-primary cursor-nesw-resize translate-y-[calc(var(--height)*1px-1px)]  after:translate-y-[calc(var(--translate)*-1)] after:translate-x-[--translate]" />
      <div data-direction="bottom" class="cursor-ns-resize border-t border-primary w-1px scale-x-[--width] origin-top-left translate-y-[calc(var(--height)*1px-1px)]" />
      <div data-direction="bottom-right" class="after:border-primary cursor-nwse-resize translate-x-[calc(var(--width)*1px-1px)] translate-y-[calc(var(--height)*1px-1px)] after:translate-[calc(var(--translate)*-1)]" />
    </div>

    <div v-if="focused && showSizeManipulators" class="absolute top-0 -left-0 translate-y-[calc(var(--height)*1px+8px)] translate-x-[calc((var(--width)*1px-1px)/2)]">
      <div class="h-16px rounded-2px bg-primary px-4px text-10px flex items-center justify-center translate-x--50%">
        {{ elementProps.width }}x{{ elementProps.height }}
      </div>
    </div>
  </div>
</template>

<style scoped>
[data-direction="top-left"],
[data-direction="top-right"],
[data-direction="bottom-left"],
[data-direction="bottom-right"] {
  position: relative;
  z-index: 2;
  display: grid;
  place-content: center;
}

[data-direction="top-left"]:after,
[data-direction="top-right"]:after,
[data-direction="bottom-left"]:after,
[data-direction="bottom-right"]:after {
  content: '';
  display: block;
  width: 8px;
  aspect-ratio: 1;
  border: 1px solid;
  background-color: #fff;
  --translate: 6px;
}
</style>
