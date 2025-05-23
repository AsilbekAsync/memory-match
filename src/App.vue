<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import { Heart, Star, Sun, Moon, Cloud, Flower2, Music, Zap, Droplets, Flame } from 'lucide-vue-next'
import confetti from 'canvas-confetti'

const icons = {
  Heart,
  Star,
  Sun,
  Moon,
  Cloud,
  Flower2,
  Music,
  Zap,
  Droplets,
  Flame
}

const iconConfigs = [
  { icon: 'Heart', color: "text-rose-500" },
  { icon: 'Star', color: "text-amber-400" },
  { icon: 'Sun', color: "text-yellow-500" },
  { icon: 'Moon', color: "text-purple-400" },
  { icon: 'Cloud', color: "text-sky-400" },
  { icon: 'Flower2', color: "text-emerald-500" },
  { icon: 'Music', color: "text-blue-500" },
  { icon: 'Zap', color: "text-orange-500" },
  { icon: 'Droplets', color: "text-cyan-500" },
  { icon: 'Flame', color: "text-red-500" }
]

const createCards = () => {
  const selectedIcons = [...iconConfigs].sort(() => Math.random() - 0.5).slice(0, 6)

  const cards = []

  selectedIcons.forEach(({ icon, color }, index) => {
    cards.push({ id: index * 2, icon, color, isMatched: false })
    cards.push({ id: index * 2 + 1, icon, color, isMatched: false })
  })

  return cards.sort(() => Math.random() - 0.5)
}

const cards = ref(createCards())
const flippedIndexes = ref([])
const matches = ref(0)
const isChecking = ref(false)
const moves = ref(0)
const gameComplete = ref(false)
const gameStarted = ref(false)
const toast = ref(null)

const totalPairs = computed(() => cards.value.length / 2)

watch(gameComplete, (newValue) => {
  if (newValue) {
    const duration = 3 * 1000
    const animationEnd = Date.now() + duration
    const defaults = { startVelocity: 30, spread: 360, ticks: 60, zIndex: 0 }

    const randomInRange = (min, max) => {
      return Math.random() * (max - min) + min
    }

    const interval = setInterval(() => {
      const timeLeft = animationEnd - Date.now()

      if (timeLeft <= 0) {
        return clearInterval(interval)
      }

      const particleCount = 50 * (timeLeft / duration)

      confetti({
        ...defaults,
        particleCount,
        origin: { x: randomInRange(0.1, 0.3), y: Math.random() - 0.2 },
      })
      confetti({
        ...defaults,
        particleCount,
        origin: { x: randomInRange(0.7, 0.9), y: Math.random() - 0.2 },
      })
    }, 250)
  }
})

const showToast = (message, description = '', type = 'default') => {
  toast.value = {
    message,
    description,
    type,
    id: Date.now()
  }

  setTimeout(() => {
    toast.value = null
  }, 3000)
}

const handleCardClick = (clickedIndex) => {
  if (!gameStarted.value) gameStarted.value = true

  if (isChecking.value || cards.value[clickedIndex].isMatched) return
  if (flippedIndexes.value.includes(clickedIndex)) return
  if (flippedIndexes.value.length === 2) return

  flippedIndexes.value.push(clickedIndex)

  if (flippedIndexes.value.length === 2) {
    moves.value++
    isChecking.value = true
    const [firstIndex, secondIndex] = flippedIndexes.value
    const firstCard = cards.value[firstIndex]
    const secondCard = cards.value[secondIndex]

    if (firstCard.icon === secondCard.icon) {
      setTimeout(() => {
        cards.value = cards.value.map((card, index) =>
          index === firstIndex || index === secondIndex ? { ...card, isMatched: true } : card
        )
        flippedIndexes.value = []
        matches.value++
        isChecking.value = false

        if (matches.value === totalPairs.value) {
          gameComplete.value = true
          showToast('🎉 Congratulations! You\'ve found all the matches!', `You completed the game in ${moves.value} moves.`, 'success')
        } else {
          showToast('Match found!', '', 'match')
        }
      }, 500)
    } else {
      setTimeout(() => {
        flippedIndexes.value = []
        isChecking.value = false
      }, 500)
    }
  }
}

const resetGame = () => {
  cards.value = createCards()
  flippedIndexes.value = []
  matches.value = 0
  moves.value = 0
  isChecking.value = false
  gameComplete.value = false
  gameStarted.value = false
}
</script>

<template>
  <div
    class="flex flex-col items-center justify-center min-h-screen p-4 space-y-8 bg-gradient-to-br from-slate-950 via-violet-950 to-slate-950">
      <div v-if="toast" :key="toast.id" :class="[
        'fixed inline top-4 left-1/2 transform -translate-x-1/2 px-4 py-3 rounded-lg shadow-lg z-50 max-w-md text-white',
        toast.type === 'success' ? 'bg-gradient-to-r from-violet-600 to-indigo-600 border-none' :
          toast.type === 'match' ? 'bg-gradient-to-r from-emerald-600 to-teal-600 border-none' :
            'bg-gradient-to-r from-slate-700 to-slate-800 border-none'
      ]">
        <div class="font-medium">{{ toast.message }}</div>
        <div v-if="toast.description" class="text-sm opacity-90">{{ toast.description }}</div>
      </div>

    <div class="text-center space-y-3 max-w-md">
      <h1
        class="text-4xl sm:text-5xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-violet-400 via-indigo-400 to-purple-400 tracking-tight">
        Memory Match
      </h1>
      <p class="text-slate-300 text-sm sm:text-base">Find all matching pairs to win the game</p>
    </div>

    <div class="w-full max-w-md">
      <div class="flex justify-between items-center mb-4 px-2">
        <div class="flex items-center gap-2">
          <div class="bg-violet-900/40 backdrop-blur-sm px-3 py-1.5 rounded-lg border border-violet-800/50">
            <span class="text-violet-200 text-sm font-medium">
              Matches: {{ matches }}/{{ totalPairs }}
            </span>
          </div>
        </div>
        <div class="bg-violet-900/40 backdrop-blur-sm px-3 py-1.5 rounded-lg border border-violet-800/50">
          <span class="text-violet-200 text-sm font-medium">Moves: {{ moves }}</span>
        </div>
      </div>

      <div
        class="grid grid-cols-3 sm:grid-cols-4 gap-3 p-4 rounded-xl bg-gradient-to-b from-violet-900/20 to-indigo-900/10 backdrop-blur-sm border border-violet-800/30 shadow-[0_0_15px_rgba(109,40,217,0.15)]">
        <div v-for="(card, index) in cards" :key="card.id" class="perspective-1000" :style="{
          transform: 'scale(1)',
          opacity: 1,
          transition: `all 0.3s ease, transform 0.3s ${index * 0.05}s`
        }">
          <div @click="handleCardClick(index)"
            class="relative w-full aspect-square rounded-xl cursor-pointer transform-style-3d transition-all duration-300"
            :class="[
              card.isMatched ? 'shadow-[0_0_15px_rgba(139,92,246,0.5)]' : 'shadow-md hover:shadow-[0_0_10px_rgba(139,92,246,0.3)]'
            ]" :style="{
                transform: card.isMatched || flippedIndexes.includes(index) ? 'rotateY(180deg)' : 'rotateY(0deg)'
              }">
            <!-- Card Back -->
            <div
              class="absolute inset-0 backface-hidden rounded-xl flex items-center justify-center bg-gradient-to-br from-violet-700/80 via-indigo-700/80 to-purple-700/80 border-2 border-violet-500/50"
              :class="[card.isMatched || flippedIndexes.includes(index) ? 'opacity-0' : 'opacity-100']">
              <div class="w-10 h-10 rounded-full bg-violet-500/30 flex items-center justify-center">
                <span class="text-violet-200 text-xl font-bold">?</span>
              </div>
              <div class="absolute inset-0 bg-gradient-to-br from-transparent via-violet-500/5 to-white/10 rounded-xl">
              </div>
            </div>

            <div
              class="absolute inset-0 backface-hidden rounded-xl flex items-center justify-center rotateY-180 bg-gradient-to-br from-slate-800/90 via-slate-900/90 to-slate-950/90"
              :class="[card.isMatched ? 'border-2 border-emerald-500/50' : 'border-2 border-violet-600/50']">
              <component :is="icons[card.icon]" :class="[
                'w-10 h-10',
                card.color,
                card.isMatched ? 'filter drop-shadow-[0_0_8px_rgba(255,255,255,0.5)]' : ''
              ]" />
              <div class="absolute inset-0 bg-gradient-to-br from-transparent via-violet-500/5 to-white/5 rounded-xl">
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <button @click="resetGame"
      class="px-6 py-2.5 text-base font-medium rounded-lg bg-gradient-to-r from-violet-700/80 to-indigo-700/80 border border-violet-500/50 hover:border-violet-400/70 text-white shadow-md hover:shadow-lg hover:shadow-violet-700/20 transition-all duration-300">
      {{ gameStarted ? "Start New Game" : "Shuffle Cards" }}
    </button>
  </div>
</template>
