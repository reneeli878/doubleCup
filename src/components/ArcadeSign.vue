<template>
  <div class="arcade-sign">
    <div
      class="sign-rotator"
      :class="{ fading }"
    >
        <img :src="currentImage" class="sign-layer depth depth-3">
        <img :src="currentImage" class="sign-layer depth depth-2">
        <img :src="currentImage" class="sign-layer depth depth-1">

        <transition name="fade-sign" mode="out-in">
            <img
                :key="currentIndex"
                :src="currentImage"
                class="sign-layer front-face"
            />
        </transition>

        <transition name="fade-sign" mode="out-in">
            <img    
                :key="`${currentIndex}-back`"
                :src="currentImage"
                class="sign-layer back-face"
            />
        </transition>


        <img :src="currentImage" class="sign-layer back-face">
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

import blue from '../assets/sign/blue.png'
import green from '../assets/sign/green.png'
import pink from '../assets/sign/pink.png'
import purple from '../assets/sign/purple.png'
import trans from '../assets/sign/trans.png'
import white from '../assets/sign/white.png'

const images = [white, blue, green, pink, purple, trans]

const currentIndex = ref(0)
const fading = ref(false)

const currentImage = computed(() => images[currentIndex.value])

const spinDuration = 3000

let intervalId = null
let changeTimeoutId = null
let fadeTimeoutId = null

function nextColor() {
  fading.value = true

  fadeTimeoutId = setTimeout(() => {
    currentIndex.value = (currentIndex.value + 1) % images.length
    fading.value = false
  }, 250)
}

onMounted(() => {
  intervalId = setInterval(() => {
    changeTimeoutId = setTimeout(() => {
      nextColor()
    }, spinDuration * 0.6)
  }, spinDuration)
})

onUnmounted(() => {
  clearInterval(intervalId)
  clearTimeout(changeTimeoutId)
  clearTimeout(fadeTimeoutId)
})
</script>

<style scoped>
.arcade-sign {
    width: 100%;
    height: 100%;
    position: relative;

    display: flex;
    justify-content: center;
    align-items: center;

    perspective: 220px;
}

.arcade-sign::before {
  content: "";
  position: absolute;
  inset: 0;

  border-radius: 50%;

  background:
    radial-gradient(
      circle,
      rgba(255, 255, 255, .1) 0%,
      rgba(0, 255, 255, .16) 35%,
      rgba(255, 0, 255, .12) 60%,
      transparent 80%
    );

  filter: blur(28px);
  z-index: 0;
  pointer-events: none;
}

.sign-rotator {
  position: relative;

  width: 280px;
  height: 280px;

  transform-style: preserve-3d;
  animation: spin3d 3s linear infinite;
  will-change: transform;

  transition:
    opacity .25s ease,
    filter .25s ease;

  z-index: 5;
}



.sign-layer {
  position: absolute;
  inset: 0;

  width: 100%;
  height: 100%;

  object-fit: contain;
  backface-visibility: visible;
}

.front-face {
  transform:
    rotate(20deg)
    translateZ(6px);

  z-index: 6;

  filter:
    drop-shadow(0 0 8px rgba(255,255,255,.8))
    drop-shadow(0 0 20px rgba(255,0,255,.45))
    drop-shadow(0 0 40px rgba(0,255,255,.35));
}


.back-face {
  transform:
    rotateY(180deg)
    scaleX(-1)
    rotate(20deg)
    translateZ(6px);

  z-index: 5;

  filter:
    drop-shadow(0 0 4px rgba(255,255,255,.65))
    drop-shadow(0 0 9px rgba(255,0,255,.35))
    drop-shadow(0 0 15px rgba(0,255,255,.25));
}


.depth {
  filter:
    brightness(.85)
    drop-shadow(0 0 2px rgba(255,255,255,.4))
    drop-shadow(0 0 4px rgba(255,0,255,.25))
    drop-shadow(0 0 6px rgba(0,255,255,.15));
}

.depth-1 {
  transform:
    rotate(20deg)
    translateZ(3px);

  opacity: .6;
  z-index: 4;
}

.depth-2 {
  transform:
    rotate(20deg)
    translateZ(0px);

  opacity: .42;
  z-index: 3;
}

.depth-3 {
  transform:
    rotate(20deg)
    translateZ(-3px);

  opacity: .28;
  z-index: 2;
}

@keyframes spin3d {
  0% {
    transform: rotateY(0deg) scaleX(1);
  }

  25% {
    transform: rotateY(90deg) scaleX(.9);
  }

  50% {
    transform: rotateY(180deg) scaleX(1);
  }

  75% {
    transform: rotateY(270deg) scaleX(.9);
  }

  100% {
    transform: rotateY(360deg) scaleX(1);
  }
}

.fade-sign-enter-active,
.fade-sign-leave-active {
  transition:
    opacity .35s ease,
    filter .35s ease;
}

.fade-sign-enter-from,
.fade-sign-leave-to {
  opacity: 0;

  filter:
    brightness(1.5)
    blur(2px);
}

.fade-sign-enter-to,
.fade-sign-leave-from {
  opacity: 1;

  filter:
    brightness(1)
    blur(0);
}

</style>