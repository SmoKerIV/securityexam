<script setup lang="ts">
import { ref } from 'vue'
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

// Remove the original quiz loading - only use uploaded files
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
