<template>
  <section class="music-player" aria-label="Nhạc nghỉ giải lao">
    <div class="music-copy">
      <p>Nhạc nghỉ</p>
      <h2>Não cần soundtrack để hồi máu</h2>
    </div>

    <div class="music-controls">
      <select v-model="selectedVideoId" @change="loadSelectedVideo">
        <option v-for="video in videoOptions" :key="video.id" :value="video.id">
          {{ video.name }}
        </option>
        <option value="custom">Dán YouTube khác...</option>
      </select>

      <input
        v-if="selectedVideoId === 'custom'"
        v-model="customVideoUrl"
        placeholder="https://www.youtube.com/watch?v=..."
        @change="setCustomVideo"
      />

      <div class="transport-row">
        <button type="button" @click="togglePlay">
          {{ playing ? 'Tạm dừng' : 'Phát thử' }}
        </button>
        <label>
          Âm lượng
          <input v-model.number="volume" max="1" min="0" step="0.01" type="range" @input="changeVolume" />
        </label>
      </div>
    </div>

    <div class="player-frame">
      <div ref="playerContainer"></div>
    </div>

    <p class="video-title">{{ videoTitle || 'Chọn bài nghỉ, đừng chọn bài suy đời quá.' }}</p>
  </section>
</template>

<script setup>
import { onBeforeUnmount, onMounted, ref, watch } from 'vue'

const props = defineProps({
  isBreak: Boolean
})

const STORAGE_KEY = 'truongdev-break-video'
const playerContainer = ref(null)
const playing = ref(false)
const volume = ref(0.45)
const videoTitle = ref('')
const customVideoUrl = ref('')
const selectedVideoId = ref(localStorage.getItem(STORAGE_KEY) || '35AdtzquJYg')

let player = null

const videoOptions = [
  { id: '35AdtzquJYg', name: 'Mưa nhẹ cho não dịu' },
  { id: 'juLHy0gp_xY', name: 'VietNam Today' },
  { id: 'XtJnyl-3tNo', name: 'Việt Nam tớ đấy' }
]

onMounted(initPlayer)
onBeforeUnmount(() => {
  if (player?.destroy) player.destroy()
})

watch(
  () => props.isBreak,
  (isBreak) => {
    if (!player) return
    isBreak ? player.playVideo() : player.pauseVideo()
  }
)

function loadYouTubeAPI() {
  return new Promise((resolve) => {
    if (window.YT?.Player) {
      resolve()
      return
    }

    const existingScript = document.querySelector('script[src="https://www.youtube.com/iframe_api"]')
    if (!existingScript) {
      const tag = document.createElement('script')
      tag.src = 'https://www.youtube.com/iframe_api'
      document.body.appendChild(tag)
    }

    const previousReady = window.onYouTubeIframeAPIReady
    window.onYouTubeIframeAPIReady = () => {
      previousReady?.()
      resolve()
    }
  })
}

async function initPlayer() {
  await loadYouTubeAPI()
  if (!playerContainer.value) return

  player = new window.YT.Player(playerContainer.value, {
    videoId: currentVideoId(),
    playerVars: { autoplay: 0, controls: 1, modestbranding: 1, rel: 0 },
    events: {
      onReady: handleReady,
      onStateChange: handleStateChange
    }
  })
}

function handleReady(event) {
  event.target.setVolume(volume.value * 100)
  updateTitle()
  if (props.isBreak) event.target.playVideo()
}

function handleStateChange(event) {
  playing.value = event.data === window.YT.PlayerState.PLAYING
  updateTitle()
}

function currentVideoId() {
  return selectedVideoId.value === 'custom' ? parseYouTubeId(customVideoUrl.value) || videoOptions[0].id : selectedVideoId.value
}

function loadSelectedVideo() {
  const id = currentVideoId()
  localStorage.setItem(STORAGE_KEY, selectedVideoId.value)
  if (player && id) player.loadVideoById(id)
}

function setCustomVideo() {
  const id = parseYouTubeId(customVideoUrl.value)
  if (!id) {
    videoTitle.value = 'URL này hơi lạc đường. Thử link YouTube khác nhé.'
    return
  }

  localStorage.setItem(STORAGE_KEY, 'custom')
  if (player) player.loadVideoById(id)
}

function togglePlay() {
  if (!player) return
  playing.value ? player.pauseVideo() : player.playVideo()
}

function changeVolume() {
  if (player) player.setVolume(volume.value * 100)
}

function updateTitle() {
  if (player?.getVideoData) videoTitle.value = player.getVideoData().title
}

function parseYouTubeId(url) {
  const match = String(url).match(/(?:youtu\.be\/|v=|embed\/|shorts\/)([0-9A-Za-z_-]{11})/)
  return match?.[1] || null
}
</script>

<style scoped>
.music-player {
  display: grid;
  grid-template-columns: minmax(220px, 0.8fr) minmax(280px, 1fr);
  gap: 18px;
  align-items: center;
  padding: 20px;
  border: 1px solid rgba(148, 163, 184, 0.2);
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.7);
  box-shadow: 0 24px 80px rgba(15, 23, 42, 0.12);
  backdrop-filter: blur(18px);
}

:global(.theme-dark) .music-player {
  border-color: rgba(148, 163, 184, 0.14);
  background: rgba(15, 23, 42, 0.68);
}

.music-copy p {
  margin: 0 0 6px;
  color: rgba(23, 32, 51, 0.66);
  font-size: 0.78rem;
  font-weight: 900;
  text-transform: uppercase;
}

:global(.theme-dark) .music-copy p,
:global(.theme-dark) .video-title {
  color: rgba(238, 242, 255, 0.68);
}

.music-copy h2 {
  margin: 0;
  font-size: clamp(1.25rem, 3vw, 2rem);
  font-weight: 900;
  line-height: 1.05;
}

.music-controls {
  display: grid;
  gap: 10px;
}

.music-controls select,
.music-controls input:not([type='range']) {
  min-height: 42px;
  border: 1px solid rgba(148, 163, 184, 0.32);
  border-radius: 8px;
  padding: 0 12px;
  color: inherit;
  background: rgba(255, 255, 255, 0.72);
}

:global(.theme-dark) .music-controls select,
:global(.theme-dark) .music-controls input:not([type='range']) {
  background: rgba(15, 23, 42, 0.68);
}

.transport-row {
  display: flex;
  gap: 12px;
  align-items: center;
  flex-wrap: wrap;
}

.transport-row button {
  min-height: 42px;
  border: 0;
  border-radius: 8px;
  padding: 0 14px;
  color: #fff;
  background: #14b8a6;
  font: inherit;
  font-weight: 900;
  cursor: pointer;
}

.transport-row label {
  display: flex;
  gap: 10px;
  align-items: center;
  font-weight: 800;
}

.transport-row input {
  accent-color: #14b8a6;
}

.player-frame {
  position: relative;
  grid-column: 1 / -1;
  overflow: hidden;
  border-radius: 8px;
  background: #0f172a;
  aspect-ratio: 16 / 9;
}

.player-frame > div,
.player-frame iframe {
  position: absolute;
  inset: 0;
  width: 100% !important;
  height: 100% !important;
}

.video-title {
  grid-column: 1 / -1;
  margin: 0;
  color: rgba(23, 32, 51, 0.66);
  font-weight: 750;
}

@media (max-width: 820px) {
  .music-player {
    grid-template-columns: 1fr;
  }
}
</style>
