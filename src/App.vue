<template>
    <div :class="['app min-h-screen p-6 md:p-8 bg-gradient-to-b transition-colors duration-500', theme === 'dark' ? 'from-gray-900 to-gray-800' : 'from-indigo-50 to-blue-50']">

        <!-- HEADER -->
        <header class="max-w-3xl mx-auto flex justify-between items-center mb-8">
            <div class="brand flex items-center gap-4">
                <div class="logo w-14 h-14 flex items-center justify-center bg-gradient-to-br from-indigo-500 to-cyan-400 text-white rounded-2xl shadow-2xl text-2xl">🍅</div>
                <div class="title">
                    <h1 class="text-2xl md:text-3xl font-extrabold text-gray-900 dark:text-white">Pomodoro • TruongDev</h1>
                    <p class="text-sm text-gray-500 dark:text-gray-400">Focus work. Ship more.</p>
                </div>
            </div>

            <div class="flex items-center gap-4">
                <!-- THEME TOGGLE -->
                <button class="p-2 rounded-full hover:bg-gray-100 dark:hover:bg-gray-700 transition-colors shadow-md" @click="toggleTheme">
                    <svg v-if="theme==='dark'" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">
                        <path d="M21 12.79A9 9 0 1111.21 3 7 7 0 0021 12.79z"></path>
                    </svg>
                    <svg v-else xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">
                        <circle cx="12" cy="12" r="5"></circle>
                        <path d="M12 1v2"></path>
                        <path d="M12 21v2"></path>
                        <path d="M4.2 4.2l1.4 1.4"></path>
                        <path d="M18.4 18.4l1.4 1.4"></path>
                        <path d="M1 12h2"></path>
                        <path d="M21 12h2"></path>
                        <path d="M4.2 19.8l1.4-1.4"></path>
                        <path d="M18.4 5.6l1.4-1.4"></path>
                    </svg>
                </button>
            </div>
        </header>

        <!-- MAIN TIMER -->
        <main class="flex flex-col items-center gap-8">
            <div class="flex flex-col md:flex-row items-center justify-center gap-8 md:gap-12">
                <div class="flex-1 md:flex justify-center">
                    <BreakMusicPlayer :isBreak="!isFocus" />
                </div>

                <!-- TIMER CARD -->
                <section class="timer-card bg-white/50 dark:bg-gray-800/50 backdrop-blur-xl rounded-3xl shadow-2xl p-6 flex flex-col items-center gap-6 w-[320px] md:w-[400px] flex-grow-0 md:flex-grow">
                    <!-- Ring Timer + Time Display -->
                    <div class="relative w-[260px] h-[260px]">
                        <svg :width="ringSize" :height="ringSize" viewBox="0 0 260 260" class="ring" aria-hidden>
                            <defs>
                                <linearGradient id="gradFocus" x1="0%" y1="0%" x2="100%" y2="0%">
                                    <stop offset="0%" stop-color="#6366F1" />
                                    <stop offset="100%" stop-color="#06B6D4" />
                                </linearGradient>
                                <linearGradient id="gradBreak" x1="0%" y1="0%" x2="100%" y2="0%">
                                    <stop offset="0%" stop-color="#F59E0B" />
                                    <stop offset="100%" stop-color="#EF4444" />
                                </linearGradient>
                            </defs>
                            <circle class="bg" cx="130" cy="130" r="100" stroke-width="20" fill="none" stroke="rgba(0,0,0,0.05)" />
                            <circle
                                :class="['progress', isFocus ? 'focus' : 'break']"
                                cx="130"
                                cy="130"
                                r="100"
                                stroke-width="20"
                                fill="none"
                                :stroke-dasharray="circumference"
                                :stroke-dashoffset="dashOffset"
                                stroke-linecap="round"
                            />
                        </svg>
                        <div class="absolute inset-0 flex flex-col items-center justify-center text-center">
                            <div class="text-5xl md:text-6xl font-extrabold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-indigo-500 to-cyan-400">
                                {{ displayTime }}
                            </div>
                            <div class="text-sm text-gray-500 dark:text-gray-400 mt-1">{{ statusLine }}</div>
                        </div>
                    </div>

                    <!-- Controls -->
                    <div class="flex gap-3 mt-4">
                        <button @click="togglePause" class="px-4 py-2 bg-gradient-to-r from-indigo-500 to-cyan-400 text-white rounded-xl shadow-lg hover:scale-105 transition transform">
                            {{ running ? '⏸ Pause' : '▶ Start' }}
                        </button>
                        <button @click="resetAll" class="px-4 py-2 bg-gray-100 dark:bg-gray-700 text-gray-900 dark:text-white rounded-xl hover:bg-gray-200 dark:hover:bg-gray-600 transition-colors">Reset</button>
                    </div>
                </section>

                <div class="flex-1 hidden md:flex justify-center">
                    <img src="https://media.tenor.com/JyxmV_MKbnIAAAAi/zhuigou.gif" alt="Monk chanting" class="w-48 h-48 rounded-xl shadow-lg" />
                </div>
            </div>

            <!-- SETTINGS + TASKS BELOW TIMER -->
            <section class="flex flex-col md:flex-row gap-6 w-full max-w-4xl">
                <!-- SETTINGS -->
                <div class="settings-card flex-1 bg-white/50 dark:bg-gray-800/50 backdrop-blur-xl rounded-3xl shadow-2xl p-6">
                    <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-4">Settings</h3>
                    <div class="space-y-4">
                        <div>
                            <label class="text-sm font-medium text-gray-700 dark:text-gray-300">Focus (min)</label>
                            <input type="range" min="5" max="60" v-model.number="focusInput" class="w-full mt-1 h-2 bg-gray-200 rounded-lg dark:bg-gray-700" />
                            <span class="text-sm font-semibold text-gray-900 dark:text-white">{{ focusInput }}m</span>
                        </div>
                        <div>
                            <label class="text-sm font-medium text-gray-700 dark:text-gray-300">Break (min)</label>
                            <input type="range" min="1" max="30" v-model.number="breakInput" class="w-full mt-1 h-2 bg-gray-200 rounded-lg dark:bg-gray-700" />
                            <span class="text-sm font-semibold text-gray-900 dark:text-white">{{ breakInput }}m</span>
                        </div>
                        <div class="flex gap-3">
                            <button @click="applySettings" class="flex-1 px-4 py-2 bg-gradient-to-r from-indigo-500 to-cyan-400 text-white rounded-xl shadow-lg">Apply</button>
                            <button @click="resetSettings" class="flex-1 px-4 py-2 bg-gray-100 dark:bg-gray-700 text-gray-900 dark:text-white rounded-xl">Reset</button>
                        </div>
                    </div>
                </div>

                <!-- TASKS -->
                <div
                    class="tasks-card flex-1 w-full md:w-auto bg-white/50 dark:bg-gray-800/50 backdrop-blur-xl rounded-3xl shadow-2xl p-4 sm:p-6"
                >
                    <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-3 sm:mb-4">Tasks</h3>

                    <!-- Input + Button: stacked trên mobile -->
                    <div class="flex flex-col sm:flex-row gap-2 mb-3 sm:mb-4">
                        <input
                            v-model="newTaskName"
                            placeholder="Task name..."
                            class="flex-1 px-2 py-1.5 rounded-xl bg-gray-100 dark:bg-gray-700 text-gray-900 dark:text-white"
                            @keyup.enter="addTask"
                        />
                        <input
                            v-model.number="newTaskPomodoro"
                            type="number"
                            min="1"
                            placeholder="Pomos"
                            class="w-20 px-2 py-1.5 rounded-xl bg-gray-100 dark:bg-gray-700 text-gray-900 dark:text-white"
                        />
                        <button
                            @click="addTask"
                            class="px-3 py-1.5 bg-gradient-to-r from-indigo-500 to-cyan-400 text-white rounded-xl"
                        >
                            Add
                        </button>
                    </div>

                    <!-- Task List -->
                    <ul class="space-y-2 sm:space-y-3 max-h-80 overflow-auto scrollbar-thin scrollbar-thumb-gray-300 dark:scrollbar-thumb-gray-700">
                        <li
                            v-for="(t, idx) in tasks"
                            :key="t.id"
                            class="group relative flex flex-col sm:flex-row items-start sm:items-center justify-between p-3 sm:p-4 rounded-2xl shadow-sm transition-all border"
                            :class="[
        t.done
          ? 'bg-green-50/70 dark:bg-green-900/30 border-green-200 dark:border-green-800 animate-checkmark'
          : idx === currentTaskIndex
            ? 'bg-indigo-50/70 dark:bg-indigo-900/30 border-indigo-300 dark:border-indigo-700'
            : 'bg-white/70 dark:bg-gray-800/50 border-gray-200/30 dark:border-gray-700/30'
      ]"
                        >
                            <!-- LEFT: Task info -->
                            <div class="flex flex-col flex-1 w-full">
                                <div class="flex items-center gap-2 flex-wrap">
          <span class="font-medium text-gray-900 dark:text-white" :class="{ 'line-through opacity-60': t.done }">
            {{ t.name }}
          </span>
                                    <span class="text-xs text-gray-500">({{ t.pomodoroDone }}/{{ t.pomodoroRequired }})</span>
                                </div>

                                <!-- Estimated vs Actual -->
                                <div
                                    v-if="t.done"
                                    class="text-xs mt-1"
                                    :class="t.actualMinutes <= t.estimatedMinutes
            ? 'text-green-600 dark:text-green-400'
            : 'text-red-600 dark:text-red-400'"
                                >
                                    {{ t.actualMinutes <= t.estimatedMinutes
                                    ? `On time 🎉 (${t.actualMinutes}m / ${t.estimatedMinutes}m)`
                                    : `Overtime ⏳ (${t.actualMinutes}m / ${t.estimatedMinutes}m)` }}
                                </div>
                                <div v-else class="text-xs text-gray-500 mt-1">
                                    Est: ~{{ t.estimatedMinutes }}m
                                </div>

                                <!-- Progress bar -->
                                <div class="h-1.5 w-full bg-gray-200 dark:bg-gray-700 rounded-full mt-2">
                                    <div
                                        class="h-1.5 bg-gradient-to-r from-indigo-500 to-cyan-400 rounded-full transition-all"
                                        :style="{ width: `${(t.pomodoroDone / t.pomodoroRequired) * 100}%` }"
                                    ></div>
                                </div>
                            </div>

                            <!-- RIGHT: Actions -->
                            <div class="flex gap-2 mt-2 sm:mt-0">
                                <button @click="selectTask(idx)" class="p-2 rounded-lg hover:bg-indigo-100 dark:hover:bg-indigo-800" title="Play">
                                    ▶
                                </button>
                                <button
                                    @click="markTaskDone(idx)"
                                    class="p-2 rounded-lg hover:bg-green-100 dark:hover:bg-green-800"
                                    title="Done"
                                    :disabled="t.done"
                                >
                                    ✅
                                </button>
                                <button
                                    @click="removeTask(idx)"
                                    class="p-2 rounded-lg hover:bg-red-100 dark:hover:bg-red-800"
                                    title="Delete"
                                >
                                    ✖
                                </button>
                            </div>
                        </li>
                    </ul>
                </div>
            </section>
        </main>
    </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import confetti from 'canvas-confetti'
import BreakMusicPlayer from "./components/BreakMusicPlayer.vue";

const defaultFocus = 25
const defaultBreak = 5

const focusInput = ref(defaultFocus)
const breakInput = ref(defaultBreak)
const focusSeconds = ref(defaultFocus * 60)
const breakSeconds = ref(defaultBreak * 60)

const taskMode = ref(true)
const muted = ref(false)
const running = ref(false)
const isFocus = ref(true)
const secondsLeft = ref(focusSeconds.value)

let idCounter = 1
const tasks = ref([])
const newTaskName = ref('')
const newTaskPomodoro = ref(1)
const currentTaskIndex = ref(-1)
const currentPomodoroIndex = ref(0)
let timer = null

const audio = new Audio('https://actions.google.com/sounds/v1/alarms/alarm_clock.ogg')
audio.volume = 0.6

const circumference = 2 * Math.PI * 100
const ringSize = 260

const totalSecondsForPhase = computed(() => (isFocus.value ? focusSeconds.value : breakSeconds.value))
const dashOffset = computed(() => {
    const ratio = Math.max(0, Math.min(1, secondsLeft.value / (totalSecondsForPhase.value || 1)))
    return circumference * (1 - ratio)
})

const displayTime = computed(() => {
    const m = Math.floor(secondsLeft.value / 60).toString().padStart(2, '0')
    const s = Math.floor(secondsLeft.value % 60).toString().padStart(2, '0')
    return `${m}:${s}`
})

const statusLine = computed(() => {
    if (taskMode.value && tasks.value.length && currentTaskIndex.value >= 0) {
        const t = tasks.value[currentTaskIndex.value]
        return `${isFocus.value ? 'Focus' : 'Break'} • ${t.name}`
    }
    return isFocus.value ? 'Focus' : 'Break'
})

watch(currentPomodoroIndex, v => { if (v < 0) currentPomodoroIndex.value = 0 })

function addTask() {
    const name = newTaskName.value.trim()
    const p = Math.max(1, Math.floor(Number(newTaskPomodoro.value) || 1))
    if (!name) return

    const estMinutes = p * focusInput.value

    tasks.value.push({
        id: idCounter++,
        name,
        pomodoroRequired: p,
        pomodoroDone: 0,
        done: false,
        createdAt: new Date(),
        estimatedMinutes: estMinutes,
        completedAt: null,
        actualMinutes: null
    })

    newTaskName.value = ''
    newTaskPomodoro.value = 1
    if (currentTaskIndex.value === -1) currentTaskIndex.value = tasks.value.length - 1
}

function removeTask(index) {
    const removingCurrent = index === currentTaskIndex.value
    tasks.value.splice(index, 1)
    if (tasks.value.length === 0) currentTaskIndex.value = -1
    else if (removingCurrent) currentTaskIndex.value = Math.min(index, tasks.value.length - 1)
    else if (index < currentTaskIndex.value) currentTaskIndex.value = currentTaskIndex.value - 1
}

function selectTask(idx) {
    if (idx >= 0 && idx < tasks.value.length) currentTaskIndex.value = idx
}
function markTaskDone(idx) {
    const t = tasks.value[idx]
    if (!t || t.done) return

    t.done = true
    t.pomodoroDone = t.pomodoroRequired
    t.completedAt = new Date()
    t.actualMinutes = Math.round((t.completedAt - t.createdAt) / 60000)
}

function applySettings() {
    focusSeconds.value = Math.max(1, Math.floor(focusInput.value)) * 60
    breakSeconds.value = Math.max(1, Math.floor(breakInput.value)) * 60
    resetPhase()
}
function resetSettings() { focusInput.value = defaultFocus; breakInput.value = defaultBreak; applySettings() }

function start() {
    if (running.value) return
    if (taskMode.value && tasks.value.length && (currentTaskIndex.value === -1 || tasks.value[currentTaskIndex.value]?.done)) {
        const idx = tasks.value.findIndex(t => !t.done)
        currentTaskIndex.value = idx === -1 ? -1 : idx
    }
    running.value = true
    timer = setInterval(tick, 1000)
}

function pause() { running.value = false; clearInterval(timer); timer = null }
function togglePause() { if (running.value) pause(); else start() }
function resetAll() {
    pause()
    for (const t of tasks.value) { t.pomodoroDone = 0; t.done = false }
    currentTaskIndex.value = tasks.value.length ? 0 : -1
    currentPomodoroIndex.value = 0
    isFocus.value = true
    secondsLeft.value = focusSeconds.value
}
function resetPhase() { secondsLeft.value = isFocus.value ? focusSeconds.value : breakSeconds.value }
function skipPomodoro() { secondsLeft.value = 1 }
function toggleMute() { muted.value = !muted.value }

function playSound() { if (!muted.value) { try { audio.currentTime = 0; audio.play() } catch (e) {} } }
function bigConfetti() { confetti({ particleCount: 140, spread: 160 }) }
function smallConfetti() { confetti({ particleCount: 18, spread: 60 }) }

function tick() {
    if (secondsLeft.value > 0) { secondsLeft.value--; return }
    playSound(); smallConfetti()

    if (isFocus.value) {
        if (taskMode.value && tasks.value.length && currentTaskIndex.value >= 0) {
            const t = tasks.value[currentTaskIndex.value]
            if (!t.done) {
                t.pomodoroDone = Math.min(t.pomodoroRequired, t.pomodoroDone + 1)
                currentPomodoroIndex.value++
                if (t.pomodoroDone >= t.pomodoroRequired) {
                    t.done = true
                    const nextIdx = tasks.value.findIndex((x, i) => !x.done && i > currentTaskIndex.value)
                    if (nextIdx !== -1) currentTaskIndex.value = nextIdx
                    else {
                        const anyLeft = tasks.value.findIndex(x => !x.done)
                        currentTaskIndex.value = anyLeft !== -1 ? anyLeft : -1
                    }
                }
            }
        }
        isFocus.value = false
        secondsLeft.value = breakSeconds.value
    } else {
        isFocus.value = true
        secondsLeft.value = focusSeconds.value
        if (taskMode.value && tasks.value.length && tasks.value.every(t => t.done)) {
            pause(); bigConfetti(); playSound(); return
        }
    }
}

watch([taskMode, () => tasks.value.length], () => {
    if (!taskMode.value || tasks.value.length === 0) { if (currentTaskIndex.value !== -1) currentTaskIndex.value = -1 }
    else { if (currentTaskIndex.value === -1 && tasks.value.length > 0) { currentTaskIndex.value = tasks.value.findIndex(t => !t.done); if (currentTaskIndex.value === -1) currentTaskIndex.value = 0 } }
})

resetAll(); applySettings()

const theme = ref('light')
function toggleTheme() { theme.value = theme.value === 'dark' ? 'light' : 'dark' }
</script>

<style scoped>
.ring { transform: rotate(-90deg); --tw-ring-shadow: 0 0 #0000; }
.ring .bg { stroke: rgba(0,0,0,0.05); }
.ring .progress { transition: stroke-dashoffset 0.9s linear, filter 0.2s ease; }
.ring .progress.focus { stroke: url(#gradFocus); }
.ring .progress.break { stroke: url(#gradBreak); }
.ring .progress.urgent { filter: url(#glow); }

@keyframes pulse {
    0% { transform: scale(1); }
    50% { transform: scale(1.06); }
    100% { transform: scale(1); }
}

input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 1.5rem; height: 1.5rem;
    background: white; border-radius: 50%; cursor: pointer; box-shadow: 0 2px 6px rgba(0,0,0,0.2);
    transition: background 0.3s;
}

input[type="range"]:hover::-webkit-slider-thumb { background: #6366F1; }

@keyframes checkmark {
    0% { transform: scale(0.9); opacity: 0.7; }
    50% { transform: scale(1.05); opacity: 1; }
    100% { transform: scale(1); opacity: 1; }
}

.animate-checkmark {
    animation: checkmark 0.4s ease-out;
}

</style>