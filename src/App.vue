<script setup lang="ts">
import { ref, onMounted } from 'vue'
import SecurityHeader from './components/SecurityHeader.vue'
import QuizComponent from './components/QuizComponent.vue'

interface Question {
  id: number
  question: string
  options: string[]
  correctAnswer: number
}

interface QuizData {
  title: string
  description: string
  questions: Question[]
}

const quizData = ref<QuizData | null>(null)

const loadQuizData = async () => {
  try {
    const response = await fetch('/quiz.json')
    if (response.ok) {
      quizData.value = await response.json()
    } else {
      console.error('Could not load quiz.json file')
    }
  } catch (error) {
    console.error('Failed to fetch quiz data:', error)
  }
}

onMounted(() => {
  loadQuizData()
})
</script>

<template>
  <div class="app-container">
    <SecurityHeader />
    <QuizComponent :quiz-data="quizData" />
  </div>
</template>

<style scoped>
.app-container {
  width: 100%;
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
}
</style>
