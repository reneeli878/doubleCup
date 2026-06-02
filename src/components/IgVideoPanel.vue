<template>
  <section class="video-panel">
    <video
      ref="videoPlayer"
      :src="currentVideo"
      class="video-player"
      autoplay
      :muted="isMuted"
      playsinline
      @ended="nextVideo"
    ></video>

    <div class="controls">
      <button
        class="control-btn"
        @click="toggleMute"
      >
        <font-awesome-icon
          :icon="isMuted ? ['fas', 'volume-xmark'] : ['fas', 'volume-high']"
        />
      </button>

      <input
        class="volume-slider"
        type="range"
        min="0"
        max="100"
        v-model="volume"
        @input="changeVolume"
      />

      <button
        class="control-btn"
        @click="nextVideo"
      >
        <font-awesome-icon
          :icon="['fas', 'forward-fast']"
        />
      </button>
    </div>
  </section>
</template>

<script setup>
import { ref, nextTick, onMounted } from 'vue'

const videoPlayer = ref(null)

const videos = [
  '/videos/reel1.mp4',
  '/videos/reel2.mp4',
  '/videos/reel3.mp4'
]

const currentIndex = ref(0)
const currentVideo = ref(videos[0])

const volume = ref(50)
const isMuted = ref(true)

function applyVolume() {
  if (!videoPlayer.value) return

  videoPlayer.value.volume = volume.value / 100
  videoPlayer.value.muted = isMuted.value
}

async function nextVideo() {
  currentIndex.value =
    (currentIndex.value + 1) % videos.length

  currentVideo.value =
    videos[currentIndex.value]

  await nextTick()

  applyVolume()

  try {
    videoPlayer.value.play()
  } catch (e) {
    console.log(e)
  }
}

function toggleMute() {
  isMuted.value = !isMuted.value
  applyVolume()
}

function changeVolume() {
  isMuted.value = Number(volume.value) === 0
  applyVolume()
}

onMounted(() => { 
  applyVolume();
})
</script>

<style scoped>
.video-panel {
  position: relative;

  width: 100%;
  height: 100%;

  background: #000;
  overflow: hidden;

  display: flex;
  justify-content: center;
  align-items: center;
}

.video-player {
  width: 100%;
  height: 100%;
  object-fit: contain;
  background: #000;
}

.controls {
  position: absolute;

  left: 50%;
  bottom: 14px;
  transform: translateX(-50%);

  display: flex;
  align-items: center;
  gap: 8px;

  width: 190px;
  padding: 8px 10px;

  background: rgba(0,0,0,.75);
  border: 2px solid white;
  border-radius: 10px;

  backdrop-filter: blur(4px);
}

.control-btn {
  display: flex;
  align-items: center;
  justify-content: center;

  width: 26px;
  height: 26px;

  border: none;
  border-radius: 7px;

  background: white;
  color: #111;

  cursor: pointer;
  font-size: 12px;
}

.volume-slider {
  width: 105px;
}

.control-btn:hover {
  background: #ddd;
}
</style>