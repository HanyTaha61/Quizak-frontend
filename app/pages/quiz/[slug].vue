<template>
  <v-container class="slug">
      <div v-if="quiz">
        <v-container v-if="!quizReady" class="text-center">
          <v-row >
          <v-text-field 
            v-model="userName" 
            label="اكتب اسمك لبدء الاختبار" 
            variant="filled" 
            placeholder="مثال: أحمد علي" 
          />
          <v-btn color="primary" class="mt-3 ml-3" @click="quizReady = true"> ابدأ الاختبار </v-btn>
        </v-row>
        </v-container>
        <div v-if="quizReady && userName.trim() !== ''">
          <Questions :quiz="quiz" :user-name="userName"/>
        </div>
      </div>
      <div v-else>
      Loading...
    </div>
  </v-container>
</template>

<script setup>
import Questions from '@/components/questions.vue'
import { useRoute } from 'vue-router'

const quiz = ref(null)
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || "http://localhost:6500";
const route = useRoute()
const userName = ref("")
const quizReady = ref(false)

useHead(() => ({
  title: quiz.value?.title || 'Quizak',

  meta: [
    {
      property: 'og:title',
      content: quiz.value?.title || 'Quizak'
    },
    {
      property: 'og:description',
      content: quiz.value?.description || ''
    },
    {
      property: 'og:image',
      content: quiz.value?.image || '/images/share.jpg'
    }
  ]
}))


  onMounted(async () => {
    quiz.value = await $fetch(`${API_BASE_URL}/api/quizzes/${route.params.slug}`)
  })
</script>