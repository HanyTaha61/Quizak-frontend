<template>
  <div>
    <Questions />
  </div>
</template>

<script setup>
import Questions from '@/components/questions.vue'
const quiz = ref(null)

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

const route = useRoute()

onMounted(async () => {
  quiz.value = await $fetch(`http://localhost:6500/api/quizzes/${route.params.id}`)
})

console.log('Quiz ID:', route.params.id)
</script>