<script setup>
import { ref, onMounted } from 'vue'

const sheet = ref(null)
const floatingButton = ref(null)

const currentTop = ref(0)
let startY = 0

let collapsedTop = 0
let middleTop = 0
let expandedTop = 80

const BUTTON_OFFSET = 16

const onResize = () => {
  const h = window.innerHeight
  collapsedTop = h - 136
  middleTop = h * 0.5
  expandedTop = 80
  if (!currentTop.value) currentTop.value = middleTop
  updateLayout(currentTop.value)
}

onMounted(() => {
  onResize()
  window.addEventListener('resize', onResize)
})

const clamp = (val, min, max) => Math.max(min, Math.min(max, val))

const handleTouchStart = (e) => {
  startY = e.touches[0].clientY
}

const handleTouchMove = (e) => {
  const y = e.touches[0].clientY
  const diff = y - startY
  startY = y

  currentTop.value = clamp(currentTop.value + diff, expandedTop, collapsedTop)
  updateLayout(currentTop.value)
}

const nearestState = (current, velocity) => {
  const threshold = 1200
  const positions = [expandedTop, middleTop, collapsedTop]

  const currentIndex = positions
    .map((v, i) => ({ i, d: Math.abs(v - current) }))
    .sort((a, b) => a.d - b.d)[0].i

  if (velocity < -threshold) return positions[Math.max(currentIndex - 1, 0)]
  if (velocity > threshold) return positions[Math.min(currentIndex + 1, positions.length - 1)]
  return positions[currentIndex]
}

const handleTouchEnd = (e) => {
  const velocity = e.changedTouches[0].clientY - startY
  const target = nearestState(currentTop.value, velocity)
  animateTo(target)
}

const animateTo = (target) => {
  currentTop.value = target
  sheet.value.style.transition = 'all 0.35s cubic-bezier(0.25, 1.0, 0.3, 1)'
  floatingButton.value.style.transition = 'bottom 0.35s cubic-bezier(0.25, 1.0, 0.3, 1), opacity 0.35s ease'
  updateLayout(target)
  setTimeout(() => {
    sheet.value.style.transition = ''
    floatingButton.value.style.transition = ''
  }, 350)
}

const updateLayout = (top) => {
  sheet.value.style.top = `${top}px`

  const fraction = (top - expandedTop) / (collapsedTop - expandedTop)
  const inset = 16 * fraction
  sheet.value.style.left = `${inset}px`
  sheet.value.style.right = `${inset}px`

  const buttonBottom = window.innerHeight - top + BUTTON_OFFSET
  floatingButton.value.style.bottom = `${buttonBottom}px`

  floatingButton.value.style.opacity = Math.min(fraction * 2, 1)
}
</script>

<template>
  <img src="/map.png" alt="map" style="position: fixed; inset: 0; width: 100%; height: 100%; object-fit: cover; display: block; z-index: 0;">

  <div class="sheet"
       ref="sheet"
       @touchstart="handleTouchStart"
       @touchmove="handleTouchMove"
       @touchend="handleTouchEnd">
    <div class="grab"></div>
  </div>

  <button class="floating" ref="floatingButton">🛰️</button>
</template>

<style>
.sheet {
  position: fixed;
  bottom: 0;
  border-radius: 60px;
  background: white;
  overflow: hidden;
  z-index: 1;
}

.grab {
  width: 40px;
  height: 4px;
  background: gray;
  border-radius: 2px;
  margin: 4px auto;
}

.floating {
  position: fixed;
  right: 20px;
  width: 50px;
  height: 50px;
  border-radius: 25px;
  background: white;
  border: none;
  font-size: 22px;
  z-index: 2;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>