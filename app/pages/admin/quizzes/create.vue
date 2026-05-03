<template>
  <v-container class="max-width-900 py-10">
    
    <!-- 🚀 HEADER SECTION -->
    <div class="d-flex align-center justify-space-between mb-8">
      <div>
        <h1 class="text-h4 font-weight-bold mb-1">Quiz Architect</h1>
        <p class="text-subtitle-1 text-medium-emphasis">Design your logic-based personality quiz</p>
      </div>
      <v-btn
        color="primary"
        size="x-large"
        rounded="xl"
        elevation="4"
        prepend-icon="mdi-rocket-launch"
        @click="submitQuiz"
      >
        Publish Quiz
      </v-btn>
    </div>

    <!-- 🔹 BASIC INFO -->
    <v-card variant="outlined" class="rounded-lg border-opacity-100 mb-8 pa-6 bg-surface">
      <div class="d-flex align-center mb-4">
        <v-icon color="primary" class="mr-2">mdi-information-outline</v-icon>
        <h3 class="text-h6 font-weight-bold">Identity</h3>
      </div>

      <v-row>
        <v-col cols="12" md="6">
          <v-text-field 
            v-model="quiz.title" 
            label="Quiz Title" 
            variant="filled" 
            placeholder="e.g. Which Star Wars character are you?" 
          />
        </v-col>
        <v-col cols="12" md="6">
          <v-text-field 
            v-model="quiz.description" 
            label="Quiz Description" 
            variant="filled" 
            placeholder="briefly describe your quiz" 
          />
        </v-col>
        <v-col cols="12" md="6">
          <v-text-field 
            v-model="quiz.slug" 
            label="URL Slug" 
            variant="filled" 
            prefix="quizapp.com/" 
          />
        </v-col>
        <v-col cols="12">
          <v-text-field 
            v-model="quiz.image" 
            label="Cover Image URL" 
            variant="filled" 
            prepend-inner-icon="mdi-image-outline"
          />
        </v-col>
      </v-row>
    </v-card>

    <!-- 🔹 RESULTS BUILDER -->
    <section class="mb-10">
      <div class="d-flex align-center justify-space-between mb-4">
        <div class="d-flex align-center">
          <v-icon color="success" class="mr-2">mdi-trophy-outline</v-icon>
          <h3 class="text-h6 font-weight-bold">Outcomes & Results</h3>
        </div>
        <v-btn variant="text" color="primary" prepend-icon="mdi-plus" @click="addResult">Add Outcome</v-btn>
      </div>

      <v-row>
        <v-col v-for="(r, ri) in quiz.results" :key="ri" cols="12" md="6">
          <v-card variant="outlined" class="pa-4 position-relative border-dashed">
            <v-btn 
              icon="mdi-close-circle" 
              variant="text" 
              color="error" 
              size="small" 
              class="position-absolute top-0 right-0 mt-2 mr-2"
              @click="quiz.results.splice(ri, 1)"
            />
            <v-text-field v-model="r.key" label="Result ID (e.g. lion)" variant="underlined" density="compact" />
            <v-text-field v-model="r.title" label="Display Title" variant="underlined" density="compact" />
            <v-textarea v-model="r.description" label="Result Description" variant="underlined" rows="2" />
          </v-card>
        </v-col>
      </v-row>
      
      <v-empty-state
        v-if="!quiz.results.length"
        text="Define what users can get as a result first."
        icon="mdi-flask-empty-outline"
      />
    </section>

    <!-- 🔹 QUESTIONS BUILDER -->
    <section class="mb-10">
      <div class="d-flex align-center justify-space-between mb-4">
        <div class="d-flex align-center">
          <v-icon color="secondary" class="mr-2">mdi-comment-question-outline</v-icon>
          <h3 class="text-h6 font-weight-bold">Questions</h3>
        </div>
        <v-btn color="secondary" rounded="pill" prepend-icon="mdi-plus" @click="addQuestion">New Question</v-btn>
      </div>

      <v-hover v-for="(q, qi) in quiz.questions" :key="qi" v-slot="{ isHovering, props }">
        <v-card
          v-bind="props"
          class="mb-6 rounded-xl transition-swing"
          :elevation="isHovering ? 8 : 1"
          variant="flat"
          border
        >
          <div class="bg-secondary-lighten-5 pa-4 d-flex align-center">
            <v-chip size="small" color="secondary" class="mr-3">Q{{ qi + 1 }}</v-chip>
            <v-text-field 
              v-model="q.text" 
              placeholder="Type your question here..." 
              hide-details 
              variant="plain"
              class="text-h6 font-weight-medium"
            />
          </div>

          <v-divider />

          <div class="pa-6">
            <div v-for="(opt, oi) in q.options" :key="oi" class="d-flex align-center gap-3 mb-3">
              <v-text-field 
                v-model="opt.text" 
                label="Option text" 
                variant="outlined" 
                density="comfortable" 
                hide-details 
              />
              <v-select
                :items="resultKeys"
                v-model="opt.value"
                label="Points to..."
                variant="outlined"
                density="comfortable"
                hide-details
                style="max-width: 200px"
                :rules="[v => !!v || 'Result mapping is required']"
              />
              <v-btn 
                icon="mdi-delete-outline" 
                variant="text" 
                color="grey" 
                @click="q.options.splice(oi, 1)" 
              />
            </div>
            
            <v-btn 
              variant="tonal" 
              size="small" 
              prepend-icon="mdi-plus" 
              class="mt-2"
              @click="addOption(qi)"
            >
              Add Choice
            </v-btn>
          </div>
        </v-card>
      </v-hover>
    </section>

  </v-container>
</template>

<script setup>
import { ref, computed } from 'vue'

const quiz = ref({
  title: '',
  description: '',
  slug: '',
  image: '',
  questions: [],
  results: []
})

definePageMeta({
  ssr: false
})

/* 🔥 dynamic result keys (important UX improvement) */
const resultKeys = computed(() =>
  quiz.value.results.map(r => r.key)
)

/* ---------------- BUILDERS ---------------- */
const addQuestion = () => {
  quiz.value.questions.push({
    text: '',
    options: [
      { text: '', value: '' },
      { text: '', value: '' }
    ]
  })
}

const addOption = (index) => {
  quiz.value.questions[index].options.push({
    text: '',
    value: ''
  })
}

const addResult = () => {
  quiz.value.results.push({
    key: '',
    title: '',
    description: '',
    image: ''
  })
}

/* ---------------- SUBMIT ---------------- */
const submitQuiz = async () => {
  try {
    await $fetch('http://localhost:6500/api/quizzes', {
      method: 'POST',
      body: quiz.value
    })

    alert('✅ Quiz Created')

  } catch (err) {
    console.error(err)
  }
}

// validation check before quiz is published (used to disable publish button if quiz is incomplete/invalid) 
const isQuizValid = computed(() => {
  if (!quiz.value.title || !quiz.value.slug) return false
  if (!quiz.value.results.length) return false
  if (!quiz.value.questions.length) return false

  return quiz.value.questions.every(q =>
    q.text &&
    q.options.length >= 2 &&
    q.options.every(opt => opt.text && opt.value)
  )
})
</script>

<style scoped>
.max-width-900 {
  max-width: 900px;
}
.border-dashed {
  border-style: dashed !important;
  border-width: 2px !important;
}
.gap-3 {
  gap: 12px;
}
.transition-swing {
  transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
}
</style>
