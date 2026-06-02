<template>
  <div class="layout">

    <div class="left-panel">
      <IgVideoPanel />
    </div>

    <div class="right-panel">

      <div class="top-panel">
        <div class="topl-panel">
          <FundStatus />
        </div>

        <div class="topr-panel">
          <ArcadeSign />
        </div>
      </div>

      <div class="bottom-panel">
        <GamePanel />
      </div>

    </div>

  </div>
</template>

<script setup>
import IgVideoPanel from '../components/IgVideoPanel.vue'
import FundStatus from '../components/FundStatus.vue'
import GamePanel from '../components/GamePanel.vue'
import ArcadeSign from '../components/ArcadeSign.vue';
</script>

<style scoped>
.layout {
  width: 100vw;
  height: 100vh;
  display: flex;
  overflow: hidden;

  background:
    linear-gradient(rgba(132, 0, 255, 0.18) 1px, transparent 1px),
    linear-gradient(90deg, rgba(138,43,226,.18) 1px, transparent 1px),
    #050505;

  background-size: 40px 40px;
  animation: gridMove 12s linear infinite;
}

.left-panel {
  width: 40%;
  height: 100%;
  background: #000;
  overflow: hidden;
}

.right-panel {
  width: 60%;
  height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.top-panel {
  height: 40%;
  display: flex;
  overflow: hidden;
}

.bottom-panel {
  height: 60%;
  overflow: hidden;
}

.topl-panel {
  width: 65%;
  height: 100%;
  overflow: hidden;
}

.topr-panel {
  width: 35%;
  height: 100%;
  overflow: hidden;
}

@keyframes gridMove {
  from {
    background-position: 0 0, 0 0;
  }

  to {
    background-position: 0 40px, 40px 0;
  }
}
</style>