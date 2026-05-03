<template>
  <v-container class="d-flex align-center justify-center bg-gradient">

    <v-card class="pa-6 quiz-card position-relative" :class="{ 'result-glow': result }" elevation="10" width="320">

      <!-- Overlay Countdown -->
      <transition name="reveal">
        <div v-if="isCounting" class="overlay">
          <div class="circle-wrapper">

            <svg class="progress-ring" width="140" height="140">
              <circle class="ring-bg" cx="70" cy="70" r="45" />

              <circle
                class="ring-progress"
                cx="70"
                cy="70"
                r="45"
                :style="{ strokeDashoffset: progressOffset }"
              />
            </svg>

            <div class="counter-text">
              {{ countdown }}
            </div>

          </div>
        </div>
      </transition>

      <!-- Quiz Content -->
      <div :class="{ blurred: isCounting }">

        <!-- Header -->
        <div class="d-flex justify-space-between align-center mb-4">
          <h2 class="text-h6 font-weight-bold">
            اكتشف الحيوان الذي يشبهك
          </h2>

          <span class="text-caption">
            {{ current + 1 }} / {{ questions.length }}
          </span>
        </div>

        <!-- Progress -->
        <v-progress-linear
          :model-value="progress"
          height="8"
          rounded
          class="mb-4"
        />

        <!-- Questions -->
        <div v-if="!result">
          <h3 class="text-subtitle-1 mb-4">
            {{ questions[current].text }}
          </h3>

          <v-row>
            <v-col
              cols="12"
              v-for="opt in questions[current].options"
              :key="opt.value"
            >
              <v-hover v-slot="{ isHovering, props }">
                <v-card
                  v-bind="props"
                  :class="[
                    'option-card',
                    {
                      hovered: isHovering,
                      selected: answers[current] === opt.value
                    }
                  ]"
                  @click="selectAnswer(opt.value)"
                >
                  <v-card-text class="d-flex justify-space-between align-center">
                    <span>{{ opt.text }}</span>
                    <v-icon>mdi-arrow-right</v-icon>
                  </v-card-text>
                </v-card>
              </v-hover>
            </v-col>
          </v-row>

          <!-- Navigation -->
          <div class="d-flex justify-space-between mt-4">
            <v-btn
              variant="text"
              :disabled="current === 0"
              @click="goBack"
            >
              رجوع
            </v-btn>

            <v-btn
              color="primary"
              :disabled="answers[current] === undefined"
              @click="goNext"
            >
              التالي
            </v-btn>
          </div>
        </div>

        <!-- Result -->
        <div v-else class="text-center mt-6">

        <v-img :src="getAnimalImage(result.key)" height="140" class="result-image mb-4" />

          <h2 class="text-h5"> {{ username }}، شخصيتك تشبه {{ translatedAnimal }} </h2>

          <p class="mt-2">{{ result.description }}</p>
         <v-btn color="green" class="mt-2" @click="shareResult"> واتساب </v-btn>
        <v-btn color="blue-darken-3" class="mt-2" @click="shareOnFacebook"> فيسبوك </v-btn>
          <v-btn variant="text" class="mt-2" @click="resetQuiz">
            إعادة الاختبار
          </v-btn>

        </div>

      </div>
    </v-card>

  </v-container>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import confetti from 'canvas-confetti'
import { useRoute } from 'vue-router'

const route = useRoute()
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || "http://localhost:6500";
const quiz = ref(null) // لو هتجيبها من API لاحقاً

const currentQuizId = computed(() => route.params.slug)

const calculatedScore = computed(() => {
  // simple scoring engine (MVP logic)
  const map = {}

  answers.value.forEach((a) => {
    if (!a) return
    map[a] = (map[a] || 0) + 1
  })

  // أعلى اختيار متكرر
  let max = 0
  let result = null

  Object.entries(map).forEach(([key, value]) => {
    if (value > max) {
      max = value
      result = key
    }
  })

  return max
})

/* ---------------- STATE ---------------- */
const current = ref(0)
const answers = ref([])
const result = ref(null)
const username = ref('أحمد')

/* Countdown */
const countdown = ref(5)
const total = 5
const isCounting = ref(true)

/* ---------------- QUESTIONS ---------------- */
const questions = [
  {
    text: 'كيف تقضي وقت فراغك؟',
    options: [
      { text: 'المغامرة', value: 'lion' },
      { text: 'الأصدقاء', value: 'dolphin' },
      { text: 'القراءة', value: 'owl' },
      { text: 'الوحدة', value: 'wolf' }
    ]
  },
  {
    text: 'كيف يصفك الآخرون؟',
    options: [
      { text: 'قائد', value: 'lion' },
      { text: 'مرح', value: 'dolphin' },
      { text: 'ذكي', value: 'owl' },
      { text: 'غامض', value: 'wolf' }
    ]
  },
  {
    text: 'كيف يصفك الآخرون؟',
    options: [
      { text: 'قائد', value: 'lion' },
      { text: 'مرح', value: 'dolphin' },
      { text: 'ذكي', value: 'owl' },
      { text: 'غامض', value: 'wolf' }
    ]
  },
  {
    text: 'كيف يصفك الآخرون؟',
    options: [
      { text: 'قائد', value: 'lion' },
      { text: 'مرح', value: 'dolphin' },
      { text: 'ذكي', value: 'owl' },
      { text: 'غامض', value: 'wolf' }
    ]
  },
  {
    text: 'كيف يصفك الآخرون؟',
    options: [
      { text: 'قائد', value: 'lion' },
      { text: 'مرح', value: 'dolphin' },
      { text: 'ذكي', value: 'owl' },
      { text: 'غامض', value: 'wolf' }
    ]
  }
]

const animalMap = {
  lion: 'الأسد',
  dolphin: 'الدلفين',
  owl: 'البومة',
  wolf: 'الذئب'
}

const translatedAnimal = computed(() => {
  return animalMap[result.value?.key] || ''
})

/* ---------------- PROGRESS LOGIC (FIXED) ---------------- */
const progress = computed(() => {
  if (result.value) return 100

  const p = ((current.value + 1) / questions.length) * 100

  // يمنع الوصول لـ 100% قبل النتيجة
  return Math.min(p, 95)
})

/* ---------------- CIRCULAR COUNTDOWN ---------------- */
const circumference = 2 * Math.PI * 45

const progressOffset = computed(() => {
  return circumference - (countdown.value / total) * circumference
})

let interval = null

const startCountdown = () => {
  // مهم: نوقف أي interval قديم
  if (interval) clearInterval(interval)

  countdown.value = total
  isCounting.value = true

  interval = setInterval(() => {
    countdown.value--

    if (countdown.value <= 0) {
      clearInterval(interval)
      isCounting.value = false
    }
  }, 1000)
}

onMounted(() => {
    startCountdown()
})

/* ---------------- QUIZ LOGIC ---------------- */
const selectAnswer = (value) => {
  answers.value[current.value] = value
}

const goNext = async () => {
  if (current.value < questions.length - 1) {
    current.value++
  } else {

    const res = await $fetch(`${API_BASE_URL}/api/results/submit`, {
      method: 'POST',
      body: {
    quizId: currentQuizId.value,
    answers: answers.value,
    timeSpent: 0
  }
})
result.value = res.data
triggerCelebration()

console.log("API RESPONSE:", res)
console.log("RESULT VALUE:", result.value)

  }
}


const goBack = () => {
  if (current.value > 0) {
    current.value--
  }
}

const resetQuiz = () => {
    current.value = 0
    answers.value = []
    result.value = null
    startCountdown()
}

/* ---------------- HELPERS ---------------- */
const getAnimalImage = (animal) => {
  return `/images/${animal}.png`
}

const shareResult = () => {
  const link = 'https://www.quizak.com'

  const text = `🔥أنا أشبه ${translatedAnimal.value} 😎
شخصيتي تشبه ${translatedAnimal.value}!

جرب بنفسك 👇
${link}`

  window.open(`https://wa.me/?text=${encodeURIComponent(text)}`)
}

const shareOnFacebook = () => {
  const link = `https://www.quizak.com/quiz/${route.params.slug}`

  const text = `🔥 أنا شخصيتي تشبه ${translatedAnimal.value} 😎`

  navigator.clipboard.writeText(`${text}\n${link}`)

  const fbUrl = `https://www.facebook.com/sharer/sharer.php?u=${encodeURIComponent(link)}`

  window.open(fbUrl, '_blank')
}

const triggerCelebration = () => {
  // burst 1
  confetti({
    particleCount: 120,
    spread: 80,
    origin: { y: 0.6 }
  })

  // burst 2 delayed
  setTimeout(() => {
    confetti({
      particleCount: 80,
      spread: 120,
      origin: { y: 0.6 }
    })
  }, 300)
}
</script>

<style scoped>
.bg-gradient {
  background: linear-gradient(135deg, #667eea, #764ba2);
}

.quiz-card {
  border-radius: 20px;
  position: relative;
}

/* OVERLAY */
.overlay {
  position: absolute;
  inset: 0;
  background: rgba(0,0,0,0.45);
  backdrop-filter: blur(8px);
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 20px;
  z-index: 10;
}

/* CIRCLE */
.circle-wrapper {
  position: relative;
  width: 140px;
  height: 140px;
}

.progress-ring {
  transform: rotate(-90deg);
}

.ring-bg {
  fill: none;
  stroke: rgba(255,255,255,0.2);
  stroke-width: 10;
}

.ring-progress {
  fill: none;
  stroke: #ffffff;
  stroke-width: 10;
  stroke-linecap: round;

  stroke-dasharray: 282.6;
  transition: stroke-dashoffset 1s linear;
}

/* COUNTER */
.counter-text {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;

  font-size: 42px;
  font-weight: bold;
  color: white;
}

/* BLUR */
.blurred {
  filter: blur(4px);
  pointer-events: none;
}

/* OPTIONS */
.option-card {
  cursor: pointer;
  transition: all 0.25s ease;
}

.option-card.hovered {
  transform: scale(1.02);
}

.option-card.selected {
  background: #e3f2fd;
  border: 2px solid #1976d2;
}

/* REVEAL */
.reveal-enter-active {
  transition: all 0.6s ease;
}

.reveal-enter-from {
  opacity: 0;
  transform: scale(0.8);
}

.reveal-enter-to {
  opacity: 1;
  transform: scale(1);
}

.result-glow {
  animation: glowPulse 1.5s ease-in-out infinite;
  box-shadow: 0 0 25px rgba(255,255,255,0.4);
}

@keyframes glowPulse {
  0% {
    transform: scale(1);
    box-shadow: 0 0 10px rgba(255,255,255,0.2);
  }
  50% {
    transform: scale(1.02);
    box-shadow: 0 0 30px rgba(255,255,255,0.5);
  }
  100% {
    transform: scale(1);
    box-shadow: 0 0 10px rgba(255,255,255,0.2);
  }
}

.result-image {
  border-radius: 16px;
  object-fit: contain; /* 🔥 أهم سطر */
}
</style>