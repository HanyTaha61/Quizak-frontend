<template>
  <v-container fluid class="pa-0">

    <!-- QUIZ GRID -->
    <v-container id="quizzes" class="mt-10">

      <h2 class="text-center mb-6 text-h5 font-weight-bold">
        اختر اختبارك
      </h2>

      <!-- Skeleton -->
      <v-row v-if="pending">
        <v-col v-for="i in 6" :key="i" cols="12" sm="6" md="4">
          <v-skeleton-loader type="card"></v-skeleton-loader>
        </v-col>
      </v-row>

      <!-- Cards -->
      <v-row v-else>
        <v-col
          v-for="quiz in quizzes"
          :key="quiz.slug"
          cols="12"
          sm="6"
          md="4"
        >
          <v-hover v-slot="{ isHovering, props }">
            <v-card
              v-bind="props"
              class="quiz-card"
              :class="{ hovered: isHovering }"
              @click="goToQuiz(quiz)"
            >

              <!-- Trending badge -->
              <v-chip
                v-if="quiz.trending"
                class="trending-badge"
                color="deep-orange"
                size="small"
              >
                🔥 ترند
              </v-chip>

              <v-img :src="quiz.image" height="180" cover />

              <v-card-title>{{ quiz.title }}</v-card-title>

              <v-card-subtitle>
                {{ quiz.description }}
              </v-card-subtitle>

              <v-card-actions class="d-flex justify-space-between">
                <v-btn color="primary" variant="text">
                  ابدأ
                </v-btn>
              </v-card-actions>

            </v-card>
          </v-hover>
        </v-col>
      </v-row>

    </v-container>

  </v-container>
</template>

<script setup>
import { useRouter } from 'vue-router'
import { ref } from 'vue'

const router = useRouter()
const quiz = ref(null)
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || "http://localhost:6500";
const userName = ref('')

/* ---------------- API FETCH ---------------- */
const { data, pending } = await useFetch(`${API_BASE_URL}/api/quizzes`, {
  default: () => ({
    quizzes: [],
    stats: { users: 0 }
  })
})

quiz.value = data.value?.quizzes?.length ? data.value.quizzes[0] : null

/* fallback لو API مش جاهز */
const fallback = {
  stats: { users: 12450 },
  quizzes: [
    {
      id: 'animal',
      title: 'أي حيوان يشبهك؟',
      description: 'اكتشف شخصيتك الحيوانية 🐯',
      image: '/images/quiz1.jpg',
      plays: 5230,
      trending: true
    },
    {
      id: 'leader',
      title: 'أي قائد أنت؟',
      description: 'هل أنت قائد بالفطرة؟',
      image: '/images/quiz2.jpg',
      plays: 3120,
      trending: false
    },
    {
      id: 'iq',
      title: 'اختبار الذكاء السريع',
      description: 'اختبر سرعة تفكيرك ⚡',
      image: '/images/quiz3.jpg',
      plays: 8420,
      trending: true
    }
  ]
}

/* merge API مع fallback */
const quizzes = ref(data.value.quizzes)

/* ---------------- ACTIONS ---------------- */
const goToQuiz = (quiz) => {
  router.push(`/quiz/${quiz.slug}`)
}

const scrollToQuizzes = () => {
  document.getElementById('quizzes')?.scrollIntoView({ behavior: 'smooth' })
}

/* ---------------- SEO ---------------- */
useHead({
  title: 'اختبارات ممتعة - اكتشف نفسك',
  meta: [
    { name: 'description', content: 'اختبارات ممتعة وسريعة قابلة للمشاركة' }
  ]
})
</script>

<style scoped>
.hero {
  height: 65vh;
  background: url('/images/hero.jpg') center/cover no-repeat;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.overlay {
  position: absolute;
  inset: 0;
  background: rgba(0,0,0,0.55);
}

.hero-content {
  position: relative;
  color: white;
  z-index: 1;
}

.hero-title {
  font-size: 38px;
  font-weight: bold;
}

.hero-subtitle {
  font-size: 18px;
  opacity: 0.9;
}

.social-proof {
  font-size: 14px;
  opacity: 0.85;
}

.quiz-card {
  position: relative;
  cursor: pointer;
  transition: all 0.3s ease;
  border-radius: 16px;
}

.quiz-card.hovered {
  transform: translateY(-6px) scale(1.02);
  box-shadow: 0 12px 30px rgba(0,0,0,0.25);
}

.trending-badge {
  position: absolute;
  top: 10px;
  left: 10px;
  z-index: 2;
}

.plays {
  font-size: 12px;
  opacity: 0.7;
}
</style>