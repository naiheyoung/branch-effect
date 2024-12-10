<template>
  <div>
    <div mb>
      <button btn mr-0.5 @click="initBranch">Refresh</button>
      <button btn ml-0.5 @click="toggleDark()">Theme</button>
    </div>
    <canvas ref="canvas" :width="w" :height="h" border mx-auto></canvas>
  </div>
</template>

<script setup lang="ts">
const canvas = $ref<HTMLCanvasElement>()!
const ctx = $computed(() => canvas.getContext('2d')!)
const w = $ref<number>(400)
const h = $ref<number>(400)
let pendingTask: Function[] = []
let frameCount = 0

const getBranchLength = (): number => {
  return Math.floor(Math.random() * (8 - 4 + 1)) + 4
}

interface Point {
  x: number
  y: number
}

interface Branch {
  start: Point
  theta: number
  length: number
}

const startPoint: Point = {
  x: w / 2,
  y: h
}

const startBranch: Branch = {
  start: startPoint,
  theta: -Math.PI / 2,
  length: getBranchLength()
}

const lineTo = (start: Point, end: Point) => {
  ctx.beginPath()
  ctx.moveTo(start.x, start.y)
  ctx.lineTo(end.x, end.y)
  ctx.stroke()
}

const getEndPoint = (b: Branch): Point => {
  return {
    x: b.start.x + b.length * Math.cos(b.theta),
    y: b.start.y + b.length * Math.sin(b.theta)
  }
}

const drawBranch = (b: Branch) => {
  lineTo(b.start, getEndPoint(b))
}

const step = (b: Branch, depth = 0, work = true) => {
  drawBranch(b)

  const end = getEndPoint(b)
  if ((depth < 3 || Math.random() < 0.5) && work) {
    pendingTask.push(() =>
      step(
        {
          start: end,
          theta: b.theta - 0.3 * Math.random(),
          length: getBranchLength()
        },
        depth + 1,
        end.y < h && end.y > 0
      )
    )
  }
  if ((depth < 3 || Math.random() < 0.5) && work) {
    pendingTask.push(() =>
      step(
        {
          start: end,
          theta: b.theta + 0.3 * Math.random(),
          length: getBranchLength()
        },
        depth + 1,
        end.y < h && end.y > 0
      )
    )
  }
}

const frame = () => {
  const tasks = [...pendingTask]
  pendingTask.length = 0
  tasks.forEach(fn => fn())
}

const startFrame = () => {
  requestAnimationFrame(() => {
    frameCount++
    if (frameCount % 5 === 0) {
      frame()
    }
    startFrame()
  })
}

const initBranch = () => {
  pendingTask.length = 0
  ctx.clearRect(0, 0, w, h)
  ctx.strokeStyle = 'rgba(136, 136, 136, 0.35)'
  Array.from({ length: 2 }).forEach(() => step(startBranch))
}

onMounted(() => {
  initBranch()
  startFrame()
})
</script>
