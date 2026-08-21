<template>
  <div :class="['app-shell', themeClass]">
    <header class="topbar">
      <div class="brand">
        <div class="brand-mark" aria-hidden="true">🍌</div>
        <div>
          <h1>A di da lat</h1>
          <p>{{ headerPunchline }}</p>
        </div>
      </div>

      <div class="topbar-actions">
        <button class="icon-button" type="button" :title="soundTitle" @click="muted = !muted">
          {{ muted ? '🔇' : '🔔' }}
        </button>
        <button class="theme-toggle" type="button" @click="toggleTheme">
          {{ theme === 'dark' ? 'Sáng lên' : 'Tối lại' }}
        </button>
      </div>
    </header>

    <main class="workspace">
      <section class="timer-panel" aria-label="Pomodoro timer">
        <div class="mode-tabs" role="tablist" aria-label="Chọn chế độ">
          <button
            v-for="mode in modes"
            :key="mode.key"
            type="button"
            :class="{ active: phase === mode.key }"
            @click="switchPhase(mode.key)"
          >
            {{ mode.label }}
          </button>
        </div>

        <div class="timer-layout">
          <div class="timer-ring-wrap">
            <svg class="timer-ring" viewBox="0 0 280 280" aria-hidden="true">
              <circle class="timer-track" cx="140" cy="140" r="118" />
              <circle
                class="timer-progress"
                cx="140"
                cy="140"
                r="118"
                :style="{ strokeDasharray: circumference, strokeDashoffset: dashOffset }"
              />
            </svg>
            <div class="time-readout">
              <span class="phase-label">{{ phaseLabel }}</span>
              <strong>{{ displayTime }}</strong>
              <small>{{ statusLine }}</small>
            </div>
          </div>

          <div class="timer-side">
            <p class="session-kicker">{{ activeTask ? 'Nhiệm vụ đang xử' : 'Không task cũng được' }}</p>
            <h2>{{ activeTask?.name || fallbackTaskTitle }}</h2>
            <div class="task-meter" v-if="activeTask">
              <span>{{ activeTask.pomodoroDone }}/{{ activeTask.pomodoroRequired }} pomodoro</span>
              <div>
                <i :style="{ width: `${taskProgress(activeTask)}%` }"></i>
              </div>
            </div>
            <div class="boss-card" v-if="activeTask">
              <div class="boss-topline">
                <span>Boss deadline</span>
                <strong>{{ bossHp }} HP</strong>
              </div>
              <div class="boss-stage" aria-hidden="true">
                <div class="boss-face">{{ bossFace }}</div>
                <div class="boss-health">
                  <i :style="{ width: `${bossHp}%` }"></i>
                </div>
              </div>
              <p>{{ bossLine }}</p>
            </div>
            <p class="coach-line">{{ coachLine }}</p>

            <div class="primary-controls">
              <button class="start-button" type="button" @click="toggleTimer">
                {{ running ? 'Tạm dừng' : 'Bắt đầu' }}
              </button>
              <button class="secondary-button" type="button" @click="skipPhase">Bỏ qua</button>
              <button class="secondary-button" type="button" @click="resetTimer">Reset</button>
            </div>

            <label class="switch-row">
              <input v-model="autoStartNext" type="checkbox" />
              <span>Tự chạy phiên kế tiếp</span>
            </label>
          </div>
        </div>
      </section>

      <aside class="stats-panel" aria-label="Thống kê phiên làm việc">
        <div>
          <span class="stat-label">Hôm nay</span>
          <strong>{{ completedPomodoros }}</strong>
          <small>pomodoro xong gọn</small>
        </div>
        <div>
          <span class="stat-label">Focus</span>
          <strong>{{ focusMinutes }}m</strong>
          <small>mỗi hiệp chiến đấu</small>
        </div>
        <div>
          <span class="stat-label">Nghỉ</span>
          <strong>{{ breakMinutes }}m</strong>
          <small>não đi uống nước</small>
        </div>
      </aside>

      <section class="tools-grid">
        <div class="settings-panel">
          <div class="section-heading">
            <p>Cài đặt</p>
            <h2>Chỉnh bếp trước khi nấu deadline</h2>
          </div>

          <label class="range-row">
            <span>Focus</span>
            <input v-model.number="focusMinutes" min="5" max="60" step="5" type="range" @change="syncCurrentPhase" />
            <strong>{{ focusMinutes }}m</strong>
          </label>
          <label class="range-row">
            <span>Nghỉ</span>
            <input v-model.number="breakMinutes" min="1" max="30" step="1" type="range" @change="syncCurrentPhase" />
            <strong>{{ breakMinutes }}m</strong>
          </label>
          <label class="range-row">
            <span>Nghỉ dài</span>
            <input v-model.number="longBreakMinutes" min="5" max="45" step="5" type="range" @change="syncCurrentPhase" />
            <strong>{{ longBreakMinutes }}m</strong>
          </label>
          <label class="range-row">
            <span>Vòng dài</span>
            <input v-model.number="longBreakEvery" min="2" max="6" step="1" type="range" />
            <strong>{{ longBreakEvery }}</strong>
          </label>

          <button class="ghost-button" type="button" @click="resetSettings">Khôi phục chuẩn 25/5</button>
        </div>

        <div class="tasks-panel">
          <div class="section-heading">
            <p>Task</p>
            <h2>Việc nào chưa xong thì cho vào nồi</h2>
          </div>

          <form class="task-form" @submit.prevent="addTask">
            <input v-model="newTaskName" maxlength="80" placeholder="Ví dụ: Refactor timer không khóc" />
            <input v-model.number="newTaskPomodoros" aria-label="Số pomodoro" min="1" max="12" type="number" />
            <button type="submit">Thêm</button>
          </form>

          <ul class="task-list" v-if="tasks.length">
            <li
              v-for="task in tasks"
              :key="task.id"
              :class="{ active: activeTask?.id === task.id, done: task.done }"
            >
              <button class="task-main" type="button" @click="selectTask(task.id)">
                <span>{{ task.name }}</span>
                <small>{{ task.pomodoroDone }}/{{ task.pomodoroRequired }} pomodoro · {{ taskEstimate(task) }}</small>
              </button>
              <div class="task-actions">
                <button type="button" title="Hoàn thành" @click="completeTask(task.id)">✓</button>
                <button type="button" title="Xóa" @click="removeTask(task.id)">×</button>
              </div>
            </li>
          </ul>
          <div class="empty-state" v-else>
            Danh sách trống. Deadline đang giả vờ ngoan.
          </div>
        </div>
      </section>

      <section class="achievements-panel" aria-label="Thành tựu Pomodoro">
        <div class="section-heading">
          <p>Thành tựu</p>
          <h2>Bảng vàng chống trì hoãn</h2>
        </div>

        <div class="achievement-grid">
          <article
            v-for="achievement in achievements"
            :key="achievement.id"
            :class="['achievement-card', { unlocked: isAchievementUnlocked(achievement.id) }]"
          >
            <span>{{ achievement.icon }}</span>
            <div>
              <h3>{{ achievement.name }}</h3>
              <p>{{ achievement.description }}</p>
            </div>
          </article>
        </div>
      </section>

      <BreakMusicPlayer :is-break="phase !== 'focus' && running" />
    </main>
  </div>
</template>

<script setup>
import { computed, onBeforeUnmount, onMounted, ref, watch } from 'vue'
import confetti from 'canvas-confetti'
import BreakMusicPlayer from './components/BreakMusicPlayer.vue'

const STORAGE_KEY = 'truongdev-pomodoro-v2'
const radius = 118
const circumference = 2 * Math.PI * radius
const modes = [
  { key: 'focus', label: 'Focus' },
  { key: 'shortBreak', label: 'Nghỉ ngắn' },
  { key: 'longBreak', label: 'Nghỉ dài' }
]

const savedState = loadState()
const theme = ref(savedState.theme || 'light')
const focusMinutes = ref(savedState.focusMinutes || 25)
const breakMinutes = ref(savedState.breakMinutes || 5)
const longBreakMinutes = ref(savedState.longBreakMinutes || 15)
const longBreakEvery = ref(savedState.longBreakEvery || 4)
const autoStartNext = ref(savedState.autoStartNext ?? false)
const muted = ref(savedState.muted ?? false)
const phase = ref(savedState.phase || 'focus')
const secondsLeft = ref(savedState.secondsLeft || focusMinutes.value * 60)
const running = ref(false)
const completedPomodoros = ref(savedState.completedPomodoros || 0)
const tasks = ref(savedState.tasks || [])
const activeTaskId = ref(savedState.activeTaskId || tasks.value.find((task) => !task.done)?.id || null)
const unlockedAchievements = ref(savedState.unlockedAchievements || [])
const newTaskName = ref('')
const newTaskPomodoros = ref(1)

let timer = null
let endsAt = 0

const activeTask = computed(() => tasks.value.find((task) => task.id === activeTaskId.value) || null)
const totalSeconds = computed(() => secondsForPhase(phase.value))
const progressRatio = computed(() => {
  const total = totalSeconds.value || 1
  return Math.max(0, Math.min(1, secondsLeft.value / total))
})
const dashOffset = computed(() => circumference * (1 - progressRatio.value))
const displayTime = computed(() => {
  const minutes = Math.floor(secondsLeft.value / 60).toString().padStart(2, '0')
  const seconds = Math.floor(secondsLeft.value % 60).toString().padStart(2, '0')
  return `${minutes}:${seconds}`
})
const phaseLabel = computed(() => modes.find((mode) => mode.key === phase.value)?.label || 'Focus')
const themeClass = computed(() => `theme-${theme.value} phase-${phase.value}`)
const soundTitle = computed(() => (muted.value ? 'Bật chuông' : 'Tắt chuông'))
const fallbackTaskTitle = computed(() => (phase.value === 'focus' ? 'Chọn một việc để cà chua bớt cô đơn' : 'Nghỉ đi, bạn xứng đáng được thở'))
const headerPunchline = computed(() => (running.value ? 'Đang chạy. Đừng alt-tab vô định.' : 'Bấm start, việc tự nhiên bớt đáng sợ hơn 3%.'))
const bossHp = computed(() => {
  if (!activeTask.value) return 100
  return Math.max(0, 100 - taskProgress(activeTask.value))
})
const bossFace = computed(() => {
  if (bossHp.value <= 0) return '💥'
  if (bossHp.value <= 35) return '😰'
  if (bossHp.value <= 65) return '😤'
  return '😎'
})
const bossLine = computed(() => {
  if (!activeTask.value) return ''
  if (bossHp.value <= 0) return 'Boss đã bay màu. Jira tự nhiên yên tĩnh hẳn.'
  if (bossHp.value <= 35) return 'Boss bắt đầu run. Thêm một hiệp nữa là nó viết retrospective.'
  if (bossHp.value <= 65) return 'Boss mất tự tin rồi. Bạn cứ gõ tiếp, đừng thương lượng.'
  return 'Boss còn gáy. Mỗi pomodoro là một cú đấm vào deadline.'
})
const achievements = [
  {
    id: 'first-pomodoro',
    icon: '🍅',
    name: 'Cà chua đầu đời',
    description: 'Hoàn thành pomodoro đầu tiên. Nhỏ thôi, nhưng não đã nghe lời.'
  },
  {
    id: 'boss-slayer',
    icon: '🏆',
    name: 'Hạ boss deadline',
    description: 'Hoàn thành một task bằng đủ số pomodoro đã hứa.'
  },
  {
    id: 'deep-work',
    icon: '🔥',
    name: 'Không bị tab mới dụ',
    description: 'Cán mốc 4 pomodoro. Sự tập trung bắt đầu có cơ bắp.'
  },
  {
    id: 'double-digit',
    icon: '⚡',
    name: 'Deadline nhìn bạn khác đi',
    description: 'Đạt 10 pomodoro. Lịch làm việc vừa phải ngồi thẳng lưng.'
  }
]
const statusLine = computed(() => {
  if (phase.value === 'focus') return running.value ? 'Đang tập trung, mạng xã hội tạm thời là truyền thuyết.' : 'Sẵn sàng gom não lại một chỗ.'
  if (phase.value === 'longBreak') return 'Nghỉ dài. Não được bảo hành chính hãng.'
  return 'Nghỉ ngắn. Đứng dậy uống nước, đừng thương lượng.'
})
const coachLine = computed(() => {
  if (!running.value) return 'Timer không phán xét. Nó chỉ ngồi đó, rất tròn, chờ bạn.'
  if (secondsLeft.value <= 60 && phase.value === 'focus') return 'Một phút cuối. Deadline đang mất bình tĩnh.'
  if (phase.value === 'focus') return 'Làm một việc thôi. Đa nhiệm là cosplay năng suất.'
  return 'Nghỉ thật nhé. Scroll 69 video không tính là thiền.'
})

watch([theme, focusMinutes, breakMinutes, longBreakMinutes, longBreakEvery, autoStartNext, muted, phase, secondsLeft, completedPomodoros, tasks, activeTaskId, unlockedAchievements], saveState, {
  deep: true
})

watch([completedPomodoros, tasks], updateAchievements, { deep: true })

onMounted(() => {
  document.documentElement.classList.toggle('dark', theme.value === 'dark')
})

onBeforeUnmount(stopTicker)

function loadState() {
  try {
    return JSON.parse(localStorage.getItem(STORAGE_KEY)) || {}
  } catch {
    return {}
  }
}

function saveState() {
  localStorage.setItem(
    STORAGE_KEY,
    JSON.stringify({
      theme: theme.value,
      focusMinutes: focusMinutes.value,
      breakMinutes: breakMinutes.value,
      longBreakMinutes: longBreakMinutes.value,
      longBreakEvery: longBreakEvery.value,
      autoStartNext: autoStartNext.value,
      muted: muted.value,
      phase: phase.value,
      secondsLeft: secondsLeft.value,
      completedPomodoros: completedPomodoros.value,
      tasks: tasks.value,
      activeTaskId: activeTaskId.value,
      unlockedAchievements: unlockedAchievements.value
    })
  )
}

function secondsForPhase(targetPhase) {
  if (targetPhase === 'focus') return focusMinutes.value * 60
  if (targetPhase === 'longBreak') return longBreakMinutes.value * 60
  return breakMinutes.value * 60
}

function toggleTheme() {
  theme.value = theme.value === 'dark' ? 'light' : 'dark'
  document.documentElement.classList.toggle('dark', theme.value === 'dark')
}

function toggleTimer() {
  running.value ? pauseTimer() : startTimer()
}

function startTimer() {
  if (running.value) return
  running.value = true
  endsAt = Date.now() + secondsLeft.value * 1000
  timer = window.setInterval(updateCountdown, 250)
}

function pauseTimer() {
  running.value = false
  stopTicker()
}

function stopTicker() {
  if (!timer) return
  window.clearInterval(timer)
  timer = null
}

function updateCountdown() {
  secondsLeft.value = Math.max(0, Math.ceil((endsAt - Date.now()) / 1000))
  if (secondsLeft.value === 0) finishPhase()
}

function finishPhase() {
  stopTicker()
  running.value = false
  chime()

  if (phase.value === 'focus') {
    completedPomodoros.value += 1
    tickActiveTask()
    updateAchievements()
    phase.value = completedPomodoros.value % longBreakEvery.value === 0 ? 'longBreak' : 'shortBreak'
    celebrate(phase.value === 'longBreak' ? 90 : 28)
  } else {
    phase.value = 'focus'
  }

  secondsLeft.value = secondsForPhase(phase.value)
  if (autoStartNext.value) startTimer()
}

function switchPhase(nextPhase) {
  phase.value = nextPhase
  resetTimer()
}

function resetTimer() {
  pauseTimer()
  secondsLeft.value = secondsForPhase(phase.value)
}

function skipPhase() {
  secondsLeft.value = 0
  finishPhase()
}

function syncCurrentPhase() {
  if (!running.value) secondsLeft.value = secondsForPhase(phase.value)
}

function resetSettings() {
  focusMinutes.value = 25
  breakMinutes.value = 5
  longBreakMinutes.value = 15
  longBreakEvery.value = 4
  syncCurrentPhase()
}

function addTask() {
  const name = newTaskName.value.trim()
  if (!name) return

  const task = {
    id: crypto.randomUUID(),
    name,
    pomodoroRequired: Math.max(1, Number(newTaskPomodoros.value) || 1),
    pomodoroDone: 0,
    done: false,
    createdAt: new Date().toISOString(),
    completedAt: null
  }

  tasks.value.unshift(task)
  activeTaskId.value = task.id
  newTaskName.value = ''
  newTaskPomodoros.value = 1
}

function selectTask(id) {
  activeTaskId.value = id
}

function removeTask(id) {
  tasks.value = tasks.value.filter((task) => task.id !== id)
  if (activeTaskId.value === id) activeTaskId.value = tasks.value.find((task) => !task.done)?.id || null
}

function completeTask(id) {
  const task = tasks.value.find((item) => item.id === id)
  if (!task) return
  task.done = true
  task.pomodoroDone = task.pomodoroRequired
  task.completedAt = new Date().toISOString()
  if (activeTaskId.value === id) activeTaskId.value = tasks.value.find((item) => !item.done)?.id || null
  unlockAchievement('boss-slayer')
  celebrate(80)
}

function tickActiveTask() {
  if (!activeTask.value || activeTask.value.done) return
  activeTask.value.pomodoroDone = Math.min(activeTask.value.pomodoroRequired, activeTask.value.pomodoroDone + 1)
  if (activeTask.value.pomodoroDone >= activeTask.value.pomodoroRequired) completeTask(activeTask.value.id)
}

function taskProgress(task) {
  return Math.round((task.pomodoroDone / task.pomodoroRequired) * 100)
}

function taskEstimate(task) {
  return `ước tính ${task.pomodoroRequired * focusMinutes.value}m`
}

function isAchievementUnlocked(id) {
  return unlockedAchievements.value.includes(id)
}

function unlockAchievement(id) {
  if (isAchievementUnlocked(id)) return
  unlockedAchievements.value = [...unlockedAchievements.value, id]
  celebrate(44)
}

function updateAchievements() {
  if (completedPomodoros.value >= 1) unlockAchievement('first-pomodoro')
  if (completedPomodoros.value >= 4) unlockAchievement('deep-work')
  if (completedPomodoros.value >= 10) unlockAchievement('double-digit')
  if (tasks.value.some((task) => task.done)) unlockAchievement('boss-slayer')
}

function chime() {
  if (muted.value) return
  const audio = new Audio('https://actions.google.com/sounds/v1/alarms/beep_short.ogg')
  audio.volume = 0.45
  audio.play().catch(() => {})
}

function celebrate(particleCount) {
  confetti({ particleCount, spread: 75, origin: { y: 0.72 } })
}
</script>

<style scoped>
.app-shell {
  min-height: 100vh;
  padding: 24px;
  color: #172033;
  background:
    radial-gradient(circle at 12% 10%, rgba(244, 114, 182, 0.16), transparent 24rem),
    radial-gradient(circle at 88% 18%, rgba(20, 184, 166, 0.16), transparent 22rem),
    linear-gradient(135deg, #f8fafc 0%, #eef2f7 52%, #fef3c7 100%);
}

.app-shell.theme-dark {
  color: #eef2ff;
  background:
    radial-gradient(circle at 12% 10%, rgba(244, 114, 182, 0.13), transparent 24rem),
    radial-gradient(circle at 88% 18%, rgba(45, 212, 191, 0.11), transparent 22rem),
    linear-gradient(135deg, #111827 0%, #172033 54%, #2d2537 100%);
}

.topbar,
.workspace {
  width: min(1180px, 100%);
  margin: 0 auto;
}

.topbar,
.brand,
.topbar-actions,
.primary-controls,
.task-actions {
  display: flex;
  align-items: center;
}

.topbar {
  justify-content: space-between;
  gap: 18px;
  margin-bottom: 22px;
}

.brand {
  gap: 14px;
}

.brand-mark {
  display: grid;
  width: 48px;
  height: 48px;
  place-items: center;
  border-radius: 8px;
  color: white;
  font-weight: 900;
  background: linear-gradient(135deg, #ef4444, #14b8a6);
  box-shadow: 0 16px 40px rgba(239, 68, 68, 0.24);
}

h1,
h2,
p {
  margin: 0;
}

.brand h1 {
  font-size: clamp(1.35rem, 3vw, 2rem);
  font-weight: 900;
}

.brand p,
.time-readout small,
.coach-line,
.stat-label,
.stats-panel small,
.section-heading p,
.task-main small,
.empty-state {
  color: rgba(23, 32, 51, 0.66);
}

.theme-dark .brand p,
.theme-dark .time-readout small,
.theme-dark .coach-line,
.theme-dark .stat-label,
.theme-dark .stats-panel small,
.theme-dark .section-heading p,
.theme-dark .task-main small,
.theme-dark .empty-state {
  color: rgba(238, 242, 255, 0.68);
}

.topbar-actions {
  gap: 10px;
}

button {
  border: 0;
  cursor: pointer;
  font: inherit;
}

.icon-button,
.theme-toggle,
.secondary-button,
.ghost-button,
.task-actions button {
  border-radius: 8px;
  color: inherit;
  background: rgba(255, 255, 255, 0.72);
  box-shadow: inset 0 0 0 1px rgba(148, 163, 184, 0.28);
}

.theme-dark .icon-button,
.theme-dark .theme-toggle,
.theme-dark .secondary-button,
.theme-dark .ghost-button,
.theme-dark .task-actions button {
  background: rgba(15, 23, 42, 0.62);
}

.icon-button {
  width: 42px;
  height: 42px;
}

.theme-toggle,
.ghost-button {
  min-height: 42px;
  padding: 0 14px;
  font-weight: 800;
}

.workspace {
  display: grid;
  gap: 18px;
}

.timer-panel,
.stats-panel,
.settings-panel,
.tasks-panel,
.achievements-panel,
.music-player {
  border: 1px solid rgba(148, 163, 184, 0.2);
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.7);
  box-shadow: 0 24px 80px rgba(15, 23, 42, 0.12);
  backdrop-filter: blur(18px);
}

.theme-dark .timer-panel,
.theme-dark .stats-panel,
.theme-dark .settings-panel,
.theme-dark .tasks-panel,
.theme-dark .achievements-panel {
  border-color: rgba(148, 163, 184, 0.14);
  background: rgba(15, 23, 42, 0.68);
}

.timer-panel {
  padding: clamp(18px, 3vw, 34px);
}

.mode-tabs {
  display: inline-grid;
  grid-template-columns: repeat(3, minmax(86px, 1fr));
  gap: 6px;
  padding: 5px;
  border-radius: 8px;
  background: rgba(15, 23, 42, 0.08);
}

.theme-dark .mode-tabs {
  background: rgba(255, 255, 255, 0.08);
}

.mode-tabs button {
  min-height: 38px;
  border-radius: 7px;
  color: inherit;
  background: transparent;
  font-weight: 800;
}

.mode-tabs button.active {
  color: #ffffff;
  background: #ef4444;
}

.timer-layout {
  display: grid;
  grid-template-columns: minmax(260px, 0.9fr) minmax(280px, 1fr);
  align-items: center;
  gap: clamp(20px, 5vw, 54px);
  margin-top: 28px;
}

.timer-ring-wrap {
  position: relative;
  width: min(100%, 360px);
  aspect-ratio: 1;
  margin: 0 auto;
}

.timer-ring {
  width: 100%;
  height: 100%;
  transform: rotate(-90deg);
}

.timer-track,
.timer-progress {
  fill: none;
  stroke-width: 18;
}

.timer-track {
  stroke: rgba(148, 163, 184, 0.22);
}

.timer-progress {
  stroke: #ef4444;
  stroke-linecap: round;
  transition: stroke-dashoffset 0.25s linear;
}

.phase-shortBreak .timer-progress {
  stroke: #14b8a6;
}

.phase-longBreak .timer-progress {
  stroke: #8b5cf6;
}

.time-readout {
  position: absolute;
  inset: 0;
  display: grid;
  place-items: center;
  align-content: center;
  gap: 6px;
  text-align: center;
}

.phase-label {
  font-size: 0.78rem;
  font-weight: 900;
  letter-spacing: 0;
  text-transform: uppercase;
}

.time-readout strong {
  font-size: clamp(3.4rem, 10vw, 5.8rem);
  line-height: 0.95;
  font-weight: 950;
}

.time-readout small {
  width: min(240px, 70%);
  min-height: 42px;
  font-weight: 700;
}

.timer-side {
  display: grid;
  gap: 16px;
}

.session-kicker,
.section-heading p,
.stat-label {
  font-size: 0.78rem;
  font-weight: 900;
  text-transform: uppercase;
}

.timer-side h2,
.section-heading h2 {
  font-size: clamp(1.35rem, 3vw, 2.4rem);
  font-weight: 900;
  line-height: 1.05;
}

.coach-line {
  font-weight: 700;
}

.task-meter {
  display: grid;
  gap: 8px;
  font-weight: 800;
}

.task-meter div,
.task-list li .task-main::after {
  height: 8px;
  overflow: hidden;
  border-radius: 999px;
  background: rgba(148, 163, 184, 0.24);
}

.task-meter i {
  display: block;
  height: 100%;
  border-radius: inherit;
  background: linear-gradient(90deg, #ef4444, #14b8a6);
}

.boss-card {
  display: grid;
  gap: 10px;
  padding: 14px;
  border: 1px solid rgba(239, 68, 68, 0.2);
  border-radius: 8px;
  background: rgba(239, 68, 68, 0.08);
}

.theme-dark .boss-card {
  background: rgba(239, 68, 68, 0.12);
}

.boss-topline,
.boss-stage {
  display: flex;
  align-items: center;
  gap: 10px;
}

.boss-topline {
  justify-content: space-between;
  font-weight: 900;
}

.boss-topline span {
  text-transform: uppercase;
  font-size: 0.78rem;
}

.boss-face {
  display: grid;
  width: 46px;
  height: 46px;
  place-items: center;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.76);
  font-size: 1.65rem;
}

.theme-dark .boss-face {
  background: rgba(15, 23, 42, 0.72);
}

.boss-health {
  flex: 1;
  height: 14px;
  overflow: hidden;
  border-radius: 999px;
  background: rgba(15, 23, 42, 0.12);
}

.theme-dark .boss-health {
  background: rgba(255, 255, 255, 0.12);
}

.boss-health i {
  display: block;
  height: 100%;
  border-radius: inherit;
  background: linear-gradient(90deg, #22c55e, #f59e0b, #ef4444);
  transition: width 0.35s ease;
}

.boss-card p {
  color: rgba(23, 32, 51, 0.72);
  font-weight: 800;
}

.theme-dark .boss-card p {
  color: rgba(238, 242, 255, 0.76);
}

.primary-controls {
  flex-wrap: wrap;
  gap: 10px;
}

.start-button {
  min-height: 48px;
  padding: 0 22px;
  border-radius: 8px;
  color: #fff;
  background: #ef4444;
  font-weight: 950;
  box-shadow: 0 16px 34px rgba(239, 68, 68, 0.28);
}

.secondary-button {
  min-height: 48px;
  padding: 0 16px;
  font-weight: 850;
}

.switch-row {
  display: flex;
  align-items: center;
  gap: 10px;
  font-weight: 750;
}

.switch-row input {
  width: 18px;
  height: 18px;
  accent-color: #ef4444;
}

.stats-panel {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1px;
  overflow: hidden;
}

.stats-panel div {
  display: grid;
  gap: 4px;
  padding: 18px;
  background: rgba(255, 255, 255, 0.24);
}

.theme-dark .stats-panel div {
  background: rgba(255, 255, 255, 0.04);
}

.stats-panel strong {
  font-size: 2rem;
  font-weight: 950;
}

.tools-grid {
  display: grid;
  grid-template-columns: minmax(260px, 0.82fr) minmax(320px, 1.18fr);
  gap: 18px;
}

.settings-panel,
.tasks-panel,
.achievements-panel {
  padding: 20px;
}

.section-heading {
  margin-bottom: 18px;
}

.range-row {
  display: grid;
  grid-template-columns: 74px 1fr 52px;
  align-items: center;
  gap: 12px;
  min-height: 48px;
  font-weight: 800;
}

input[type='range'] {
  accent-color: #ef4444;
}

.ghost-button {
  width: 100%;
  margin-top: 16px;
}

.task-form {
  display: grid;
  grid-template-columns: 1fr 76px 88px;
  gap: 8px;
}

.task-form input,
.task-form button {
  min-height: 44px;
  border: 1px solid rgba(148, 163, 184, 0.32);
  border-radius: 8px;
  padding: 0 12px;
  color: inherit;
  background: rgba(255, 255, 255, 0.72);
}

.theme-dark .task-form input {
  background: rgba(15, 23, 42, 0.68);
}

.task-form button {
  color: #fff;
  border-color: transparent;
  background: #14b8a6;
  font-weight: 900;
}

.task-list {
  display: grid;
  gap: 10px;
  max-height: 380px;
  padding: 0;
  margin: 16px 0 0;
  overflow: auto;
  list-style: none;
}

.task-list li {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 10px;
  align-items: center;
  padding: 12px;
  border: 1px solid rgba(148, 163, 184, 0.26);
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.5);
}

.theme-dark .task-list li {
  background: rgba(15, 23, 42, 0.44);
}

.task-list li.active {
  border-color: #ef4444;
}

.task-list li.done {
  opacity: 0.62;
}

.task-main {
  display: grid;
  gap: 5px;
  min-width: 0;
  color: inherit;
  text-align: left;
  background: transparent;
}

.task-main span {
  overflow-wrap: anywhere;
  font-weight: 900;
}

.task-actions {
  gap: 6px;
}

.task-actions button {
  width: 34px;
  height: 34px;
  font-weight: 950;
}

.empty-state {
  padding: 26px 4px 8px;
  font-weight: 800;
}

.achievement-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 12px;
}

.achievement-card {
  display: flex;
  gap: 12px;
  min-height: 116px;
  padding: 14px;
  border: 1px solid rgba(148, 163, 184, 0.24);
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.42);
  opacity: 0.48;
  filter: grayscale(1);
  transition: transform 0.2s ease, opacity 0.2s ease, filter 0.2s ease;
}

.theme-dark .achievement-card {
  background: rgba(15, 23, 42, 0.42);
}

.achievement-card.unlocked {
  border-color: rgba(20, 184, 166, 0.38);
  opacity: 1;
  filter: grayscale(0);
  background: rgba(20, 184, 166, 0.1);
}

.achievement-card.unlocked:hover {
  transform: translateY(-2px);
}

.achievement-card > span {
  display: grid;
  flex: 0 0 42px;
  width: 42px;
  height: 42px;
  place-items: center;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.72);
  font-size: 1.45rem;
}

.theme-dark .achievement-card > span {
  background: rgba(15, 23, 42, 0.72);
}

.achievement-card h3 {
  margin: 0 0 6px;
  font-size: 1rem;
  font-weight: 950;
}

.achievement-card p {
  margin: 0;
  color: rgba(23, 32, 51, 0.66);
  font-weight: 740;
}

.theme-dark .achievement-card p {
  color: rgba(238, 242, 255, 0.68);
}

@media (max-width: 820px) {
  .app-shell {
    padding: 16px;
  }

  .topbar,
  .timer-layout,
  .tools-grid {
    grid-template-columns: 1fr;
  }

  .topbar {
    align-items: flex-start;
  }

  .stats-panel {
    grid-template-columns: 1fr;
  }

  .achievement-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  .task-form {
    grid-template-columns: 1fr 76px;
  }

  .task-form button {
    grid-column: 1 / -1;
  }
}

@media (max-width: 560px) {
  .topbar {
    display: grid;
  }

  .mode-tabs {
    width: 100%;
    grid-template-columns: 1fr;
  }

  .range-row,
  .task-list li {
    grid-template-columns: 1fr;
  }

  .achievement-grid {
    grid-template-columns: 1fr;
  }

  .task-actions {
    justify-content: flex-end;
  }
}
</style>
