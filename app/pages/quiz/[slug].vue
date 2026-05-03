<template>
  <div>
    <Questions />
  </div>
</template>

<script setup>
import Questions from '@/components/questions.vue'
import { useRoute } from 'vue-router'

const quiz = ref(null)
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || "http://localhost:6500";
const route = useRoute()

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

console.log('Quiz slug:', route.params.slug)
</script>