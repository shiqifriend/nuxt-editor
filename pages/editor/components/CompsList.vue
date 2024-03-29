<script setup lang='ts'>
import NGoast from './NGoast.vue'
import { BasicDescribes } from '~/components/Basic/index.describe'
import { ShapeDescribes } from '~/components/Shape/index.describe'
import { useCanvasRender } from '~/store/canvasRender'

const canvasRender = useCanvasRender()

const basicComponents = BasicDescribes
const basicShapes = ShapeDescribes

let dragFlag = false
let mousedownSnapshotName = ''
let currentDescribe: any = ''
const currentComponent = ref('')
const style = ref({ left: '0', top: '0', transform: '', cursor: 'grab' })
function mousedownHandler(e: MouseEvent, name: string) {
  e.preventDefault()
  dragFlag = true
  mousedownSnapshotName = name
  document.addEventListener('mousemove', mousemoveHandler)
  document.addEventListener('mouseup', mouseupHandler)
}
function mousemoveHandler(e: MouseEvent) {
  if (!dragFlag)
    return
  currentComponent.value = mousedownSnapshotName
  const { clientX, clientY } = e
  if (!currentDescribe)
    currentDescribe = findDiscribe(mousedownSnapshotName)
  requestAnimationFrame(() => {
    style.value = {
      ...style.value,
      transform: `scale(${canvasRender.scale})`,
      left: `${clientX - currentDescribe!.w / 2}px`,
      top: `${clientY - currentDescribe!.h / 2}px`,
    }
  })
}
function mouseupHandler(e: MouseEvent) {
  if (!dragFlag)
    return
  const { clientX, clientY } = e
  const position = calcPosition({ x: clientX, y: clientY })
  canvasRender.addComponent({
    ...currentDescribe,
    ...position,
  })

  dragFlag = false
  currentComponent.value = ''
  mousedownSnapshotName = ''
  currentDescribe = ''
}
function findDiscribe(name: string) {
  return [...BasicDescribes, ...ShapeDescribes].find((describe) => {
    return describe.name === name
  })
}
function calcPosition(pointer: { x: number; y: number }) {
  const container = document.querySelector('.canvas-target')!
  const rect = container.getBoundingClientRect()
  const { x, y } = pointer
  const { left, top } = rect
  return {
    left: (x - left) / canvasRender.scale - currentDescribe!.w / 2,
    top: (y - top) / canvasRender.scale - currentDescribe!.h / 2,
  }
}

onUnmounted(() => {
  document.removeEventListener('mousemove', mousemoveHandler)
  document.removeEventListener('mouseup', mouseupHandler)
})
</script>

<template>
  <div>
    <Teleport to="body">
      <NGoast
        v-if="currentComponent"
        :component="currentComponent"
        absolute z-100
        :style="style"
      />
    </Teleport>
    <div>
      <div text-sm text-center b-b="1px gray dashed" py-2>
        <strong> Basic Components </strong>
      </div>
      <div flex flex-wrap justify-around p-2>
        <div
          v-for="item of basicComponents" :key="item.name"
          flex-center class="comp-item"
          @mousedown="mousedownHandler($event, item.name)"
        >
          <div :class="item.icon" />
        </div>
      </div>
    </div>
    <div>
      <div text-sm text-center b-b="1px gray dashed" py-2>
        <strong> Basic Shapes </strong>
      </div>
      <div flex flex-wrap justify-around p-2>
        <div
          v-for="item of basicShapes" :key="item.name"
          flex-center class="comp-item"
          @mousedown="mousedownHandler($event, item.name)"
        >
          <div :class="item.icon" />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped lang='scss'>
  .comp-item {
    width: 45%;
    height: 25px;
    margin: 5px 0 5px 0;
    border: 1px dashed gray;
    cursor: pointer;
    transition: all .5s;
    &:hover {
      color: orange
    }
  }
</style>
