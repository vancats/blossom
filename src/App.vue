<script setup lang="ts">
const WIDTH = 500
const HEIGHT = 500
const el = $ref<HTMLCanvasElement>()
const ctx = $computed(() => el!.getContext('2d')!)

interface Point {
  x: number
  y: number
}

interface Branch {
  start: Point
  length: number
  theta: number
}

function lineTo(p1: Point, p2: Point) {
  ctx.beginPath()
  ctx.moveTo(p1.x, p1.y)
  ctx.lineTo(p2.x, p2.y)
  ctx.stroke()
}

function getEndPoint(b: Branch): Point {
  const { start, length, theta } = b
  return {
    x: start.x + length * Math.cos(theta),
    y: start.y + length * Math.sin(theta),
  }
}

function drawBranch(b: Branch) {
  lineTo(b.start, getEndPoint(b))
}

function init() {
  ctx.strokeStyle = 'rgba(21, 223,122,0.5)'

  step({
    start: { x: WIDTH / 2, y: HEIGHT },
    length: 20,
    theta: -Math.PI / 2,
  })
}

const pendingTasks: Function[] = []
function step(b: Branch, depth = 0) {
  const end = getEndPoint(b)
  drawBranch(b)
  if (depth < 4 || Math.random() < 0.51) {
    pendingTasks.push(() => step({
      start: end,
      length: b.length + (Math.random() * 10 - 5),
      theta: b.theta - 0.3 * Math.random(),
    }, depth + 1))
  }

  if (depth < 4 || Math.random() < 0.51) {
    pendingTasks.push(() => step({
      start: end,
      length: b.length + (Math.random() * 8 - 4),
      theta: b.theta + 0.5 * Math.random(),
    }, depth + 1))
  }
}

function frame() {
  const tasks = [...pendingTasks]
  pendingTasks.length = 0
  tasks.forEach(fn => fn())
}

let frameCount = 0
function startFrame() {
  requestAnimationFrame(() => {
    frameCount += 1
    if (frameCount % 8 === 0)
      frame()

    startFrame()
  })
}

startFrame()

onMounted(() => {
  // 需要在 onMounted 里面才能获取到 canvas 节点
  init()
})
</script>

<template>
  <main font-sans p="x-4 y-10" text="gray-700 dark:gray-200">
    <canvas ref="el" :width="WIDTH" :height="HEIGHT" border />
    <Footer />
  </main>
</template>
