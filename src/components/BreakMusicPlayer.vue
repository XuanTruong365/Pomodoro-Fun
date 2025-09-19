<template>
    <div class="music-player flex flex-col items-center gap-2 p-4 bg-white dark:bg-gray-800/80 rounded-xl shadow-md w-full max-w-md">

        <!-- Select Video / Custom URL -->
        <div class="flex gap-2 w-full items-center">
            <label class="text-sm font-medium text-gray-700 dark:text-gray-300">Break Music:</label>
            <select v-model="selectedVideoId" @change="onVideoSelect" class="flex-1 rounded-md p-1">
                <option v-for="v in videoOptions" :key="v.id" :value="v.id">{{ v.name }}</option>
                <option value="">Custom...</option>
            </select>
        </div>
        <input
            v-if="selectedVideoId === ''"
            type="text"
            v-model="customVideoUrl"
            @change="setCustomVideo"
            placeholder="Paste YouTube URL"
            class="w-full rounded-md p-1 mt-1"
        />

        <!-- Play/Pause + Volume -->
        <div class="flex items-center gap-3 mt-2 w-full">
            <button @click="togglePlay" class="px-3 py-1 bg-indigo-500 text-white rounded-md hover:bg-indigo-600">
                {{ playing ? '⏸ Pause' : '▶ Play' }}
            </button>
            <input type="range" min="0" max="1" step="0.01" v-model.number="volume" @input="changeVolume" class="flex-1"/>
        </div>

        <!-- Player -->
        <div class="w-full mt-2 relative overflow-hidden rounded-lg shadow">
            <!-- giữ tỷ lệ 16:9 -->
            <div class="relative pb-[56.25%] h-0">
                <div ref="playerContainer" class="absolute top-0 left-0 w-full h-full"></div>
            </div>
        </div>

        <!-- Video Title -->
        <div class="text-sm text-gray-700 dark:text-gray-300 mt-1">{{ videoTitle }}</div>
    </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue'

const props = defineProps({
    isBreak: Boolean
})

const playerContainer = ref(null)
let player = null
const playing = ref(false)
const volume = ref(0.5)
const videoTitle = ref('')

// ---- LocalStorage support ----
const savedVideoId = localStorage.getItem('breakVideoId') || '3JZ_D3ELwOQ'
const selectedVideoId = ref(savedVideoId)
const customVideoUrl = ref('')

const videoOptions = [
    { id: 'juLHy0gp_xY', name: 'VietNam Today' },
    { id: '35AdtzquJYg', name: 'Relaxing Rain Sounds' },
    { id: 'XtJnyl-3tNo', name: 'Việt Nam Tớ Đấy' }
]

// ---- Load YouTube API ----
function loadYouTubeAPI() {
    return new Promise(resolve => {
        if (window.YT && window.YT.Player) return resolve()
        const tag = document.createElement('script')
        tag.src = "https://www.youtube.com/iframe_api"
        document.body.appendChild(tag)
        window.onYouTubeIframeAPIReady = () => resolve()
    })
}

// ---- Initialize or Update Player ----
async function initPlayer() {
    await loadYouTubeAPI()
    if (!playerContainer.value) return
    if (!player) {
        player = new YT.Player(playerContainer.value, {
            videoId: selectedVideoId.value,
            playerVars: { autoplay: 0, controls: 0, modestbranding: 1, rel: 0 },
            events: { onReady: onPlayerReady, onStateChange: onStateChange }
        })
    } else {
        // chỉ load video mới, không recreate
        player.loadVideoById(selectedVideoId.value)
    }
}

function onPlayerReady(e) {
    e.target.setVolume(volume.value * 100)
    fetchTitle()
    if (props.isBreak) e.target.playVideo()
}

function onStateChange(e) {
    playing.value = e.data === YT.PlayerState.PLAYING
    fetchTitle()
}

function fetchTitle() {
    if (player && player.getVideoData) videoTitle.value = player.getVideoData().title
}

// ---- Play/Pause & Volume ----
function togglePlay() {
    if (!player) return
    if (playing.value) player.pauseVideo()
    else player.playVideo()
}

function changeVolume() {
    if (player) player.setVolume(volume.value * 100)
}

// ---- Handle Video Selection ----
function onVideoSelect() {
    if (selectedVideoId.value !== '') {
        saveSelectedVideo(selectedVideoId.value)
        if (player) player.loadVideoById(selectedVideoId.value)
    }
}

function setCustomVideo() {
    const id = parseYouTubeId(customVideoUrl.value)
    if (!id) return alert('Invalid URL')
    selectedVideoId.value = id
    saveSelectedVideo(id)
    if (player) player.loadVideoById(id)
}

function saveSelectedVideo(id) {
    localStorage.setItem('breakVideoId', id)
}

// ---- Watch break status ----
watch(() => props.isBreak, val => {
    if (!player) return
    if (val) player.playVideo()
    else player.pauseVideo()
})

// ---- Parse YouTube ID from URL ----
function parseYouTubeId(url) {
    if (!url) return null
    const regex = /(?:v=|\/)([0-9A-Za-z_-]{11}).*/
    const match = url.match(regex)
    return match ? match[1] : null
}

// ---- Mounted ----
onMounted(() => {
    initPlayer()
})
</script>

<style scoped>
.music-player input[type="range"] { width: 100%; }
.music-player iframe {
    width: 100% !important;
    height: 100% !important;
    border-radius: 0.75rem; /* bo góc đồng bộ UI */
}
</style>
