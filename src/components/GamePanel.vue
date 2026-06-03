<template>
  <section class="game-panel">
    <div class="arcade-bg"></div>

    <div v-if="showConfetti" class="confetti-layer">
      <span v-for="n in 24" :key="n" class="confetti"></span>
    </div>

    <div v-if="showChampion" class="champion-pop">
      NEW CHAMPION!
    </div>

    <div class="top-bar">
      <div class="player-card">
        <div class="title">PLAYER</div>

        <input v-model="playerName" placeholder="NAME" />

        <div class="character-select">
          <button
            v-for="character in characters"
            :key="character.id"
            class="character-btn"
            :class="{ selected: selectedCharacter === character.id }"
            type="button"
            @click="selectedCharacter = character.id"
            @mouseenter="hoveredCharacter = character.id"
            @mouseleave="hoveredCharacter = null"
          >
            <img
              :src="hoveredCharacter === character.id ? character.frame2 : character.frame1"
              :alt="character.name"
            />
          </button>
        </div>
      </div>

      <div class="leaderboard">
        <div class="title">
          <span>RANK</span>

          <div class="rank-tools">
            <button class="icon-btn" title="下載排行榜" @click="downloadRanking">
              <font-awesome-icon :icon="['fas', 'floppy-disk']" />
            </button>

            <button class="icon-btn" title="匯入排行榜" @click="fileInput.click()">
              <font-awesome-icon :icon="['fas', 'crown']" />
            </button>

            <button class="icon-btn" title="清除所有資料" @click="clearRanking">
              <font-awesome-icon :icon="['fas', 'burst']" />
            </button>
          </div>
        </div>

        <div class="leaderboard-content">
          <div
            v-for="(player, index) in topThree"
            :key="index"
            class="rank-item"
          >
            <span class="rank-number">{{ index + 1 }}.</span>

            <img
              class="rank-character"
              :src="getCharacterFrame(player.character)"
              :alt="player.character"
            />

            <span class="rank-text">
              {{ player.name }} - {{ player.time.toFixed(2) }}s
            </span>
          </div>

          <div v-if="topThree.length === 0" class="rank-item">
            NO DATA
          </div>
        </div>
      </div>
    </div>

    <div class="machine">
      <div class="press-start">
        <label class="press-start-label" v-show="!isRunning && elapsed === 0">
          INSERT 0 COINS TO START
        </label>
      </div>

      <div class="timer-stage">
        <div class="character-stage">
          <img
            v-if="selectedCharacterData"
            class="timer-character"
            :src="isRunning ? selectedCharacterData.frame2 : selectedCharacterData.frame1"
            :alt="selectedCharacterData.name"
          />
        </div>

        <div class="timer" :class="{ running: isRunning }">
          {{ displayTime }}
          <div class="unit">s</div>
        </div>
      </div>

      <div class="buttons-row">
  <div></div>

        <div class="buttons">
          <button class="button" @click="startTimer">START</button>
          <button class="button" @click="stopTimer">STOP</button>
          <button class="button" @click="resetTimer">RESET</button>
          <button class="button" @click="addResult">SAVE</button>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const BASE_URL = import.meta.env.BASE_URL

const playerName = ref('')
const fileInput = ref(null)
const elapsed = ref(0)
const isRunning = ref(false)
const results = ref([])

const hoveredCharacter = ref(null)
const selectedCharacter = ref('green')

const characters = [
  {
    id: 'blue',
    name: 'blue',
    frame1: `${BASE_URL}characters/blue1.png`,
    frame2: `${BASE_URL}characters/blue2.png`
  },
  {
    id: 'clear',
    name: 'clear',
    frame1: `${BASE_URL}characters/clear1.png`,
    frame2: `${BASE_URL}characters/clear2.png`
  },
  {
    id: 'green',
    name: 'green',
    frame1: `${BASE_URL}characters/green1.png`,
    frame2: `${BASE_URL}characters/green2.png`
  },
  {
    id: 'pink',
    name: 'pink',
    frame1: `${BASE_URL}characters/pink1.png`,
    frame2: `${BASE_URL}characters/pink2.png`
  },
  {
    id: 'purple',
    name: 'purple',
    frame1: `${BASE_URL}characters/purple1.png`,
    frame2: `${BASE_URL}characters/purple2.png`
  },
  {
    id: 'white',
    name: 'white',
    frame1: `${BASE_URL}characters/white1.png`,
    frame2: `${BASE_URL}characters/white2.png`
  }
]

let timerInterval = null
let startTime = 0

const displayTime = computed(() => {
  return (elapsed.value / 1000).toFixed(2)
})

const topThree = computed(() => {
  return results.value.slice(0, 10)
})

const selectedCharacterData = computed(() => {
  return characters.find(character => character.id === selectedCharacter.value)
})

onMounted(() => {
  const saved = localStorage.getItem('cupRanking')

  if (saved) {
    results.value = JSON.parse(saved)
  }
})

const showConfetti = ref(false)
const showChampion = ref(false)

function getCharacterFrame(characterId) {
  const character = characters.find(item => item.id === characterId)

  return character
    ? character.frame1
    : `${BASE_URL}characters/green1.png`
}

function triggerChampionEffect() {
  showConfetti.value = true
  showChampion.value = true

  setTimeout(() => {
    showConfetti.value = false
    showChampion.value = false
  }, 2200)
}

function startTimer() {
  if (isRunning.value) return

  isRunning.value = true
  startTime = Date.now() - elapsed.value

  timerInterval = setInterval(() => {
    elapsed.value = Date.now() - startTime
  }, 10)
}

function stopTimer() {
  clearInterval(timerInterval)
  isRunning.value = false
}

function resetTimer() {
  clearInterval(timerInterval)
  isRunning.value = false
  elapsed.value = 0
}

function addResult() {
  if (!playerName.value.trim()) {
    alert('請輸入玩家名稱')
    return
  }

  stopTimer()

  const newResult = {
    name: playerName.value.trim(),
    character: selectedCharacter.value,
    time: elapsed.value / 1000
  }

  const oldBestTime = results.value.length ? results.value[0].time : Infinity

  results.value.push(newResult)
  results.value.sort((a, b) => a.time - b.time)

  localStorage.setItem('cupRanking', JSON.stringify(results.value))

  if (newResult.time < oldBestTime) {
    triggerChampionEffect()
  }

  playerName.value = ''
  elapsed.value = 0
}

function downloadRanking() {
  if (!results.value.length) {
    alert('尚無排行榜資料')
    return
  }

  const csvRows = [['排名', '玩家', '角色', '秒數']]

  results.value.forEach((item, index) => {
    csvRows.push([
      index + 1,
      item.name,
      item.character || '',
      item.time.toFixed(2)
    ])
  })

  const csvContent = csvRows.map(row => row.join(',')).join('\n')

  const blob = new Blob(['\ufeff' + csvContent], {
    type: 'text/csv;charset=utf-8;'
  })

  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')

  a.href = url
  a.download = 'cup-ranking.csv'
  a.click()

  URL.revokeObjectURL(url)
}

function importRanking(event) {
  const file = event.target.files[0]

  if (!file) return

  const reader = new FileReader()

  reader.onload = e => {
    try {
      const imported = JSON.parse(e.target.result)

      if (!Array.isArray(imported)) {
        throw new Error()
      }

      results.value = imported
      localStorage.setItem('cupRanking', JSON.stringify(imported))

      alert('排行榜匯入成功')
    } catch {
      alert('檔案格式錯誤')
    }
  }

  reader.readAsText(file)
  event.target.value = ''
}

function clearRanking() {
  if (!confirm('確定要清除所有排行榜資料？')) {
    return
  }

  results.value = []
  localStorage.removeItem('cupRanking')

  alert('排行榜已清除')
}
</script>

<style scoped>
.game-panel,
.game-panel * {
  box-sizing: border-box;
}

.game-panel {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
  padding: 18px;
  color: #fff4c7;
  font-family: 'Press Start 2P', 'Unifont', monospace;
  background: transparent;
  isolation: isolate;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.arcade-bg {
  position: absolute;
  inset: 0;
  z-index: -1;
  background: transparent;
}

.top-bar {
  height: 36%;
  min-height: 190px;
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 14px;
  flex-shrink: 0;
}

.player-card,
.leaderboard {
  min-height: 0;
  height: 100%;
  padding: 12px;
  background: #181020;
  border: 4px solid #ffd54f;
  box-shadow: 6px 6px 0 #000;
}

.player-card {
  overflow: hidden;
}

.title {
  margin-bottom: 10px;
  height: 26px;
  color: #7df9ff;
  font-size: clamp(10px, 1vw, 14px);
  display: flex;
  align-items: center;
  justify-content: space-between;
}

input {
  width: 100%;
  padding: 0.65rem;
  color: white;
  background: #2a193f;
  border: 3px solid #7df9ff;
  outline: none;
  font-family: 'Press Start 2P', 'Unifont', monospace;
  font-size: clamp(18px, 1.5vw, 25px);
}

.character-select {
  margin-top: 10px;
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 8px;
}

.character-btn {
  height: 58px;
  padding: 3px;
  color: #fff4c7;
  background: #090510;
  border: 2px solid #3a2a52;
  border-bottom: 4px solid #000;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition:
    transform 0.12s ease,
    border-color 0.12s ease,
    box-shadow 0.12s ease,
    background 0.12s ease;
}

.character-btn img {
  width: 52px;
  height: 52px;
  object-fit: contain;
  image-rendering: pixelated;
}

.character-btn:hover {
  background: #12091f;
  border-color: #7df9ff;
  box-shadow: 0 0 10px #7df9ff;
  transform: translateY(-2px);
}

.character-btn.selected {
  border-color: #ffdd00;
  box-shadow:
    0 0 8px #ffdd00,
    0 0 14px #7df9ff;
}

.leaderboard-content {
  height: calc(100% - 24px);
  overflow-y: auto;
  scrollbar-width: thin;
  scrollbar-color: #7df9ff #10081d;
}

.rank-item {
  color: #fff4c7;
  font-size: clamp(10px, 1.15vw, 16px);
  line-height: 1.45;
  font-family: 'Press Start 2P', 'Unifont', monospace;
  display: flex;
  align-items: center;
  gap: 8px;
  white-space: nowrap;
}

.rank-number {
  min-width: 36px;
}

.rank-character {
  width: 30px;
  height: 30px;
  object-fit: contain;
  image-rendering: pixelated;
  filter: drop-shadow(0 0 4px #7df9ff);
}

.rank-text {
  overflow: hidden;
  text-overflow: ellipsis;
}

.machine {
  flex: 1;
  min-height: 0;
  padding: 10px 22px 20px;
  display: grid;
  grid-template-rows: 38px 1fr auto;
  align-items: center;
  border: 4px solid #7df9ff;
  background: rgba(0, 0, 0, .55);
  box-shadow:
    0 0 18px rgba(125, 249, 255, .7),
    inset 0 0 24px rgba(125, 249, 255, .25);
}

.press-start {
  height: 34px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.press-start-label {
  color: #ffdd00;
  font-size: clamp(11px, 1.3vw, 18px);
  animation: blink 1s steps(2) infinite;
  text-shadow:
    2px 2px #000,
    0 0 12px #ffdd00;
}

.timer-stage {
  width: 100%;
  min-height: 0;
  display: grid;
  grid-template-columns: 30% 70%;
  align-items: center;
}

.character-stage {
  width: 100%;
  height: 100%;
  min-height: 150px;
  display: flex;
  align-items: flex-end;
  justify-content: center;
  overflow: visible;

  padding-top: 50px;
  padding-bottom: 10px;
}

.timer-character {
  width: clamp(180px, 22vw,280px);
  height: clamp(180px, 22vw, 280px);
  object-fit: contain;
  image-rendering: pixelated;
  filter:
    drop-shadow(0 0 8px #7df9ff)
    drop-shadow(0 0 18px #ffffff);
}

.timer {
  min-width: 0;
  display: flex;
  align-items: flex-end;
  justify-content: center;
  gap: 12px;
  font-size: clamp(68px, 10vw, 128px);
  line-height: 1;
  color: #00ff88;
  text-shadow:
    5px 5px #003b20,
    0 0 22px #00ff88;
}

.timer.running {
  animation: timerPulse .35s steps(2) infinite;
}

.unit {
  margin-bottom: 10px;
  font-family: "Unifont", monospace;
  font-size: clamp(28px, 3vw, 52px);
  color: #ffdd00;
}

.buttons-row {
  width: 100%;
  display: grid;
  grid-template-columns: 30% 70%;
  align-items: center;
}

.buttons {
  margin-top: 8px;

  display: flex;
  gap: 20px;
  justify-content: center;
  flex-wrap: wrap;
}

button {
  padding: 10px 14px;
  color: #1b1028;
  background: #ffdd00;
  border: 0;
  border-bottom: 5px solid #9b6d00;
  font-family: 'Press Start 2P', 'Unifont', monospace;
  font-size: clamp(9px, .8vw, 12px);
  cursor: pointer;
}

button:active {
  transform: translateY(4px);
  border-bottom-width: 1px;
}

.rank-tools {
  display: flex;
  align-items: center;
  gap: 8px;
}

.icon-btn {
  width: 26px;
  height: 26px;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #1b1028;
  background: #ffdd00;
  border: 0;
  border-bottom: 4px solid #9b6d00;
  font-size: 13px;
  cursor: pointer;
}

.icon-btn:hover,
.button:hover {
  background: #fff176;
  filter: drop-shadow(0 0 6px #ffdd00);
}

.confetti-layer {
  position: absolute;
  inset: 0;
  z-index: 20;
  pointer-events: none;
  overflow: hidden;
}

.confetti {
  position: absolute;
  top: -20px;
  width: 10px;
  height: 18px;
  background: #ffdd00;
  box-shadow:
    0 0 8px #ffdd00,
    0 0 14px #7df9ff;
  animation: confettiFall 2.2s linear forwards;
}

.confetti:nth-child(3n) {
  background: #7df9ff;
}

.confetti:nth-child(4n) {
  background: #ff4fd8;
}

.confetti:nth-child(5n) {
  background: #00ff88;
}

.confetti:nth-child(1) { left: 4%; animation-delay: 0s; }
.confetti:nth-child(2) { left: 8%; animation-delay: .12s; }
.confetti:nth-child(3) { left: 12%; animation-delay: .05s; }
.confetti:nth-child(4) { left: 18%; animation-delay: .22s; }
.confetti:nth-child(5) { left: 23%; animation-delay: .1s; }
.confetti:nth-child(6) { left: 29%; animation-delay: .28s; }
.confetti:nth-child(7) { left: 34%; animation-delay: .04s; }
.confetti:nth-child(8) { left: 39%; animation-delay: .2s; }
.confetti:nth-child(9) { left: 44%; animation-delay: .08s; }
.confetti:nth-child(10) { left: 49%; animation-delay: .26s; }
.confetti:nth-child(11) { left: 54%; animation-delay: .14s; }
.confetti:nth-child(12) { left: 59%; animation-delay: .02s; }
.confetti:nth-child(13) { left: 64%; animation-delay: .18s; }
.confetti:nth-child(14) { left: 69%; animation-delay: .06s; }
.confetti:nth-child(15) { left: 74%; animation-delay: .24s; }
.confetti:nth-child(16) { left: 79%; animation-delay: .11s; }
.confetti:nth-child(17) { left: 84%; animation-delay: .3s; }
.confetti:nth-child(18) { left: 89%; animation-delay: .03s; }
.confetti:nth-child(19) { left: 94%; animation-delay: .16s; }
.confetti:nth-child(20) { left: 98%; animation-delay: .25s; }
.confetti:nth-child(21) { left: 15%; animation-delay: .34s; }
.confetti:nth-child(22) { left: 37%; animation-delay: .36s; }
.confetti:nth-child(23) { left: 62%; animation-delay: .32s; }
.confetti:nth-child(24) { left: 86%; animation-delay: .38s; }

.champion-pop {
  position: absolute;
  left: 50%;
  top: 48%;
  z-index: 25;
  transform: translate(-50%, -50%);
  padding: 18px 24px;
  color: #ffdd00;
  background: rgba(0, 0, 0, .85);
  border: 4px solid #7df9ff;
  font-size: clamp(18px, 2vw, 34px);
  text-align: center;
  text-shadow:
    3px 3px #000,
    0 0 14px #ffdd00;
  box-shadow:
    0 0 20px #7df9ff,
    0 0 40px #ffdd00;
  animation: championPop 2.2s ease forwards;
}

@keyframes blink {
  0%, 45% {
    opacity: 1;
  }

  46%, 100% {
    opacity: .25;
  }
}

@keyframes timerPulse {
  0% {
    transform: scale(1);
  }

  100% {
    transform: scale(1.015);
  }
}

@keyframes confettiFall {
  0% {
    transform: translateY(-20px) rotate(0deg);
    opacity: 1;
  }

  100% {
    transform: translateY(110vh) rotate(720deg);
    opacity: 0;
  }
}

@keyframes championPop {
  0% {
    transform: translate(-50%, -50%) scale(.4);
    opacity: 0;
  }

  15% {
    transform: translate(-50%, -50%) scale(1.15);
    opacity: 1;
  }

  35% {
    transform: translate(-50%, -50%) scale(1);
  }

  80% {
    opacity: 1;
  }

  100% {
    transform: translate(-50%, -50%) scale(.8);
    opacity: 0;
  }
}
</style>