<script setup lang="ts">
import { ref, computed, watch } from "vue";
import { useMediaControls, onKeyStroke } from "@vueuse/core";
import { gsap } from "gsap";
import sampleTimestamps from "./sampleTimestamps";

const player = ref(null)
const { currentTime, playing } = useMediaControls(player)

interface Hit {
  hitTimeStamp: number
  label: "perfect" | "good" | "wtf" | "miss" | null
  hasIndicator: boolean
}
const hits = ref<Hit[]>([])

for (const hitTimeStamp of sampleTimestamps) {
  hits.value.push({
    hitTimeStamp: hitTimeStamp,
    label: null,
    hasIndicator: false,
  })
}

const nextHit = computed((): Hit | null => {
  return hits.value.find(hit => hit.hitTimeStamp > currentTime.value) || null
})

const perfectMarginBefore = 0.4
const goodMarginBefore = 0.6

onKeyStroke(' ', (e) => {
  if (nextHit.value === null)
    return
  if (nextHit.value.label !== null)
    return
  e.preventDefault()

  const perfectTimings = [nextHit.value.hitTimeStamp - perfectMarginBefore, nextHit.value.hitTimeStamp]
  const goodTimings = [nextHit.value.hitTimeStamp - goodMarginBefore, nextHit.value.hitTimeStamp]

  if (currentTime.value > perfectTimings[0] && currentTime.value < perfectTimings[1]) {
    nextHit.value.label = 'perfect'
  } else if (currentTime.value > goodTimings[0] && currentTime.value < goodTimings[1]) {
    nextHit.value.label = 'good'
  } else {
    nextHit.value.label = 'wtf'
  }
})

const perfectHitsCount = computed(() => hits.value.filter(hit => hit.label === 'perfect').length)
const goodHitsCount = computed(() => hits.value.filter(hit => hit.label === 'good').length)
const wtfHitsCount = computed(() => hits.value.filter(hit => hit.label === 'wtf').length)
const missHitsCount = computed(() => hits.value.filter(hit => hit.label === 'miss').length)

watch(currentTime, () => {
  if (nextHit.value === null)
    return

  for (const hit of hits.value) {
    if (hit.label === null && hit.hitTimeStamp < currentTime.value) {
      hit.label = 'miss'
    }

    const hitSpawnIndicatorTimeStamp = hit.hitTimeStamp - 1.25;
    if (currentTime.value > hitSpawnIndicatorTimeStamp && !hit.hasIndicator) {
      hit.hasIndicator = true
      spawnIndicator(hit.hitTimeStamp - currentTime.value)
    }
  }
})

const indicatorContainer = ref()

const spawnIndicator = (duration: number) => {
  const indicator = document.createElement('div')
  indicator.style.width = '100px'
  indicator.style.height = '100px'
  indicator.style.borderRadius = '50%'
  indicator.style.position = 'absolute'
  indicator.style.background = 'rgb(34,193,195)';
  indicator.style.background = 'linear-gradient(0deg, rgba(34,193,195,1) 0%, rgba(253,187,45,1) 100%)'
  gsap.to(indicator, {
    duration: duration,
    x: 450,
    onComplete: () => {
      indicator.remove()
    }
  })
  indicatorContainer.value.appendChild(indicator)
}

const play = () => {
  playing.value = true;
}

const stop = () => {
  playing.value = false;
  currentTime.value = 0;
}
</script>

<template>
  <div>
    <div>
      <h1 v-if="!playing">Hit SPACE on the beat</h1>
      <h1 v-else-if="currentTime < sampleTimestamps[0] - 1.25">Wait for the drop...</h1>
      <h1 v-else-if="currentTime < sampleTimestamps[0]">Go!</h1>
      <h1 v-else>🎧</h1>
    </div>

    <div class="indicator-container" ref="indicatorContainer">
      <div class="indicator-hit-circle"></div>
    </div>
  
    <div class="d-flex justify-content-center">
      <div class="score-container score-container-perfect">
        <div class="score-title">
          Perfect 🤩
        </div>
        <div class="score-counter">
          {{ perfectHitsCount }}
        </div>
      </div>
      <div class="score-container score-container-good">
        <div class="score-title">
          Good 😌
        </div>
        <div class="score-counter">
          {{ goodHitsCount }}
        </div>
      </div>
      <div class="score-container score-container-wtf">
        <div class="score-title">
          Wtf 🫣
        </div>
        <div class="score-counter">
          {{ wtfHitsCount }}
        </div>
      </div>
      <div class="score-container score-container-miss">
        <div class="score-title">
          Missed 😴
        </div>
        <div class="score-counter">
          {{ missHitsCount }}
        </div>
      </div>
    </div>
  
    <div class="d-flex justify-content-center" style="margin-top: 2rem">
      <button v-if="!playing" @click="play" style="width: 400px; padding: 1rem; font-weight: 600;">
        Play
      </button>
      <button v-else @click="stop" style="width: 400px; padding: 1rem; font-weight: 600;">
        Stop
      </button>
    </div>
  
    <audio ref="player" id="audio" controls style="display: none;">
      <source src="./assets/sample.mp3" type="audio/mpeg" />
      Your browser does not support the audio element.
    </audio>
  </div>
</template>

<style scoped>
.indicator-container {
  height: 200px;
  width: 1000px;
  margin-top: 4rem;
  position: relative;
}

.indicator-hit-circle {
  width: 98px;
  height: 98px;
  border-radius: 50%;
  border: 2px solid #36383C;
  position: absolute;
  right: 50%;
  transform: translate(50%, 0%)
}

.d-flex {
  display: flex;
  gap: 1rem;
}

.flex-wrap {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.justify-content-center {
  justify-content: center;
}

.score-container {
  width: 100px;
  padding: 1rem;
  color: white;
  border-radius: 1rem;
  margin-top: 1rem;
}

.score-title {
  font-size: 1rem;
  font-weight: 600;
}

.score-counter {
  font-size: 2.5rem;
}

.score-container-perfect {
  background-color: #D27C6A;
}

.score-container-good {
  background-color: #915665;
}

.score-container-wtf {
  background-color: #4C3949;
}

.score-container-miss {
  background-color: #36383C;
}
</style>
