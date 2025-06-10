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
const uploadedQuizData = ref<QuizData | null>(null)

// Handle shared quiz loading from URL
onMounted(() => {
  const urlParams = new URLSearchParams(window.location.search)
  const encodedData = urlParams.get('data')

  if (encodedData) {
    try {
      // Decode the quiz data from URL
      const decodedData = decodeURIComponent(atob(encodedData))
      const sharedQuizData = JSON.parse(decodedData)

      // Validate the quiz data
      if (sharedQuizData.title && sharedQuizData.questions && Array.isArray(sharedQuizData.questions)) {
        uploadedQuizData.value = sharedQuizData
        // Clean up URL
        window.history.replaceState({}, document.title, window.location.pathname)
      }
    } catch (error) {
      console.error('Error loading shared quiz:', error)
      alert('Invalid shared quiz link')
    }
  }
})
</script>

<template>
  <div class="app-container">
    <SecurityHeader />
    <main class="main-content">
      <div class="content-wrapper">
        <div class="quiz-wrapper">
          <QuizComponent :quiz-data="uploadedQuizData" @quiz-uploaded="uploadedQuizData = $event" />
        </div>
      </div>
    </main>
  </div>
</template>

<style scoped>
.app-container {
  width: 100%;
  height: 100vh;
  background: linear-gradient(135deg, #0f172a 0%, #1e293b 50%, #334155 100%);
  background-attachment: fixed;
  position: relative;
  overflow-x: hidden;
  display: flex;
  flex-direction: column;
}

.app-container::before {
  content: '';
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background:
    radial-gradient(circle at 20% 30%, rgba(59, 130, 246, 0.1) 0%, transparent 40%),
    radial-gradient(circle at 80% 70%, rgba(99, 102, 241, 0.08) 0%, transparent 50%),
    radial-gradient(circle at 50% 50%, rgba(147, 197, 253, 0.05) 0%, transparent 60%);
  pointer-events: none;
  z-index: 0;
}

.main-content {
  flex: 1;
  position: relative;
  z-index: 1;
  display: flex;
  align-items: flex-start;
  justify-content: center;
  padding: 0;
  overflow-y: auto;
}

.content-wrapper {
  width: 100%;
  max-width: 1400px;
  padding: 20px;
  display: flex;
  align-items: flex-start;
  justify-content: center;
  min-height: 100%;
}

.quiz-wrapper {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

@media (max-width: 768px) {
  .content-wrapper {
    padding: 10px;
  }
}
</style>
