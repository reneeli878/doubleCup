<template>
  <div
    class="fund-box"
    :class="{ refreshing: isRefreshing }"
  >
    <div class="fund-title">MISSION STATUS</div>

    <div class="amount">
      NT$ {{ Math.floor(displayAmount).toLocaleString() }}
    </div>

    <div class="progress-row">
      <span>達成率</span>
      <strong>{{ progressPercent }}%</strong>
    </div>

    <div class="progress-bar">
      <div
        class="progress-fill"
        :style="{ width: progressWidth }"
      ></div>
    </div>

    <div class="people">
        <font-awesome-icon
          :icon="['fas', 'person-dots-from-line']"
        /> 
        {{ people }} 人支持
    </div>

    <div class="updated">
      UPDATE {{ lastUpdated }}
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const TARGET_AMOUNT = 30000;

const rawAmount = ref(0)
const displayAmount = ref(0)
const rawPeople = ref(0)
const lastUpdated = ref('--:--:--')
const isRefreshing = ref(false)

let timer = null
let animationFrame = null

const people = computed(() => {
  if (!rawPeople.value) return '...'
  return rawPeople.value.toLocaleString()
})

const progressPercent = computed(() => {
  if (!rawAmount.value) return 0
  return Math.floor((rawAmount.value / TARGET_AMOUNT) * 100)
})

const progressWidth = computed(() => {
  return `${Math.min(progressPercent.value, 100)}%`
})

function refreshAnimation() {
  isRefreshing.value = false

  requestAnimationFrame(() => {
    isRefreshing.value = true

    setTimeout(() => {
      isRefreshing.value = false
    }, 900)
  })
}

function animateAmount(target) {
  if (animationFrame) cancelAnimationFrame(animationFrame)

  const start = displayAmount.value
  const diff = target - start
  const duration = 1200
  const startTime = performance.now()

  const update = (time) => {
    const progress = Math.min((time - startTime) / duration, 1)
    displayAmount.value = start + diff * progress

    if (progress < 1) {
      animationFrame = requestAnimationFrame(update)
    }
  }

  animationFrame = requestAnimationFrame(update)
}

async function fetchFundData() {
  try {
    refreshAnimation()

    const url = 'https://www.zeczec.com/projects/LESEMEUR-OSUCUP'
    const proxyUrl = `https://corsproxy.io/?${encodeURIComponent(
      `${url}?t=${Date.now()}`
    )}`

    const response = await fetch(proxyUrl)
    const html = await response.text()

    const parser = new DOMParser()
    const doc = parser.parseFromString(html, 'text/html')

    const amountEl = doc.querySelector('.js-sum-raised')

    if (amountEl) {
      const amount =
        parseInt(amountEl.textContent.replace(/[^0-9]/g, ''), 10) || 0

      rawAmount.value = amount
      animateAmount(amount)
    }

    const bodyText = doc.body.textContent || ''

    const peopleMatch =
      bodyText.match(/(\d[\d,]*)\s*人贊助/) ||
      bodyText.match(/贊助人數[：:\s]*(\d[\d,]*)/)

    if (peopleMatch) {
      rawPeople.value =
        parseInt(peopleMatch[1].replace(/,/g, ''), 10) || 0
    }

    lastUpdated.value = new Date().toLocaleTimeString('zh-TW')
  } catch (error) {
    console.error(error)
  }
}

onMounted(() => {
  fetchFundData()
  timer = setInterval(fetchFundData, 30000)
})

onUnmounted(() => {
  if (timer) clearInterval(timer)
  if (animationFrame) cancelAnimationFrame(animationFrame)
})
</script>

<style scoped>
.fund-box {
  width: 100%;
  height: 100%;
  padding: 20px 24px;
  box-sizing: border-box;

  display: flex;
  flex-direction: column;
  justify-content: center;

  
  color: white;
  overflow: hidden;

  font-family: "Unifont", monospace;
}

.fund-box,
.fund-box * {
  font-family: "Unifont", monospace !important;
}

.fund-title {
  font-size: 12px;
  color: #ffdd00;
  margin-bottom: 14px;
  text-shadow: 2px 2px #000;
}

.amount {
  font-size: clamp(32px, 4vw, 58px);
  line-height: 1.1;
  color: #00ff88;
  word-break: break-word;
  text-shadow:
    3px 3px #003b20,
    0 0 12px #00ff88;
}

.progress-row {
  margin-top: 18px;
  display: flex;
  justify-content: space-between;
  font-size: 16px;
  color: white;
}

.progress-row strong {
  color: #ffdd00;
  font-size: 18px;
}

.progress-bar {
  width: 100%;
  height: 16px;
  margin-top: 8px;
  border: 2px solid white;
  background: #222;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background:
    repeating-linear-gradient(
      90deg,
      #ffdd00 0px,
      #ffdd00 10px,
      #ff9900 10px,
      #ff9900 20px
    );
  transition: width .8s steps(8);
}

.people {
  margin-top: 18px;
  font-size: clamp(14px, 2vw, 22px);
  color: #4da3ff;
  text-shadow:
    2px 2px #001b3a,
    0 0 10px #4da3ff;
}

.updated {
  margin-top: 14px;
  font-size: 15px;
  color: #aaa;
  letter-spacing: 1px;
}

.refreshing {
  animation: refreshBlink 0.9s steps(2);
}

.refreshing .amount {
  animation: amountBounce 0.9s steps(3);
}

.refreshing .progress-fill {
  filter: brightness(1.7);
}

@keyframes refreshBlink {
  0% {
    box-shadow: inset 0 0 0 0 #00ff88;
  }

  40% {
    box-shadow: inset 0 0 0 4px #00ff88;
  }

  100% {
    box-shadow: inset 0 0 0 0 #00ff88;
  }
}

@keyframes amountBounce {
  0% {
    transform: scale(1);
  }

  40% {
    transform: scale(1.04);
  }

  100% {
    transform: scale(1);
  }
}
</style>