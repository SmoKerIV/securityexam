<script setup lang="ts">
import { ref, computed } from 'vue'

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

const props = defineProps<{
  quizData: QuizData | null
}>()

const selectedAnswers = ref<Record<number, number>>({})
const showFinalResults = ref(false)
const uploadedQuizData = ref<QuizData | null>(null)

// Use uploaded quiz data if available, otherwise use props
const currentQuizData = computed(() => uploadedQuizData.value || props.quizData)

const selectAnswer = (questionId: number, answerIndex: number) => {
  selectedAnswers.value[questionId] = answerIndex
  
  // Show final results if all questions are answered
  if (currentQuizData.value && Object.keys(selectedAnswers.value).length === currentQuizData.value.questions.length) {
    setTimeout(() => {
      showFinalResults.value = true
    }, 500) // Small delay to show the last answer feedback
  }
}

const score = computed(() => {
  if (!currentQuizData.value) return 0
  let correct = 0
  currentQuizData.value.questions.forEach(question => {
    if (selectedAnswers.value[question.id] === question.correctAnswer) {
      correct++
    }
  })
  return correct
})

const resetQuiz = () => {
  selectedAnswers.value = {}
  showFinalResults.value = false
}

const resetToOriginal = () => {
  uploadedQuizData.value = null
  selectedAnswers.value = {}
  showFinalResults.value = false
}

const handleFileUpload = (event: Event) => {
  const target = event.target as HTMLInputElement
  const file = target.files?.[0]
  
  if (!file) return
  
  if (file.type !== 'application/json') {
    alert('Please upload a JSON file')
    return
  }
  
  const reader = new FileReader()
  reader.onload = (e) => {
    try {
      const result = e.target?.result as string
      const jsonData = JSON.parse(result)
      
      // Validate the JSON structure
      if (!jsonData.title || !jsonData.questions || !Array.isArray(jsonData.questions)) {
        alert('Invalid quiz format. Please ensure your JSON has "title" and "questions" array.')
        return
      }
      
      // Validate each question has required fields
      const isValid = jsonData.questions.every((q: any) => 
        q.id && q.question && Array.isArray(q.options) && typeof q.correctAnswer === 'number'
      )
      
      if (!isValid) {
        alert('Invalid question format. Each question must have id, question, options array, and correctAnswer number.')
        return
      }
      
      uploadedQuizData.value = jsonData
      selectedAnswers.value = {}
      showFinalResults.value = false
      
      // Clear the file input
      target.value = ''
      
    } catch (error) {
      alert('Error parsing JSON file. Please check the file format.')
    }
  }
  
  reader.readAsText(file)
}

const isAnswered = (questionId: number) => {
  return selectedAnswers.value[questionId] !== undefined
}

const isCorrect = (questionId: number, optionIndex: number) => {
  if (!currentQuizData.value) return false
  const question = currentQuizData.value.questions.find(q => q.id === questionId)
  return question && question.correctAnswer === optionIndex
}

const isSelected = (questionId: number, optionIndex: number) => {
  return selectedAnswers.value[questionId] === optionIndex
}
</script>

<template>
  <div class="quiz-container" v-if="currentQuizData">
    <div class="quiz-header">
      <h1>{{ currentQuizData.title }}</h1>
      <p>{{ currentQuizData.description }}</p>
      <div class="progress">
        <span>Progress: {{ Object.keys(selectedAnswers).length }} / {{ currentQuizData.questions.length }}</span>
      </div>
    </div>

    <div class="file-upload" v-if="!showFinalResults">
      <div class="upload-section">
        <label for="quiz-upload" class="upload-btn">
          📁 Upload Custom Quiz (JSON)
        </label>
        <input 
          id="quiz-upload" 
          type="file" 
          @change="handleFileUpload" 
          accept=".json" 
          style="display: none;" 
        />
        <button 
          v-if="uploadedQuizData" 
          @click="resetToOriginal" 
          class="back-btn"
        >
          ↩️ Back to Original Quiz
        </button>
      </div>
      <p class="upload-note">Upload a JSON file with your own quiz questions</p>
    </div>

    <div class="questions" v-if="!showFinalResults">
      <div 
        v-for="question in currentQuizData.questions" 
        :key="question.id" 
        class="question"
        :class="{ 'answered': isAnswered(question.id) }"
      >
        <h3>{{ question.id }}. {{ question.question }}</h3>
        <div class="options">
          <label 
            v-for="(option, index) in question.options" 
            :key="index"
            class="option"
            :class="{
              'correct': isAnswered(question.id) && isCorrect(question.id, index),
              'incorrect': isAnswered(question.id) && isSelected(question.id, index) && !isCorrect(question.id, index),
              'disabled': isAnswered(question.id)
            }"
          >
            <input 
              type="radio" 
              :name="`question-${question.id}`"
              :value="index"
              :disabled="isAnswered(question.id)"
              @change="selectAnswer(question.id, index)"
            />
            <span>{{ option }}</span>
            <span v-if="isAnswered(question.id) && isCorrect(question.id, index)" class="feedback correct-mark">✓</span>
            <span v-if="isAnswered(question.id) && isSelected(question.id, index) && !isCorrect(question.id, index)" class="feedback incorrect-mark">✗</span>
          </label>
        </div>
        
        <div v-if="isAnswered(question.id)" class="instant-feedback">
          <p v-if="selectedAnswers[question.id] === question.correctAnswer" class="correct-feedback">
            ✓ Correct!
          </p>
          <p v-else class="incorrect-feedback">
            ✗ Incorrect. The correct answer is: {{ question.options[question.correctAnswer] }}
          </p>
        </div>
      </div>
    </div>

    <div class="final-results" v-if="showFinalResults">
      <h2>Quiz Complete!</h2>
      <div class="score">
        <p>Final Score: {{ score }} out of {{ currentQuizData.questions.length }}</p>
        <p class="percentage">{{ Math.round((score / currentQuizData.questions.length) * 100) }}%</p>
      </div>
      
      <button class="reset-btn" @click="resetQuiz">Try Again</button>
      <button class="back-btn" v-if="uploadedQuizData" @click="resetToOriginal">Back to Original Quiz</button>
    </div>
  </div>
  
  <div class="loading" v-else>
    <p>Loading quiz...</p>
  </div>
</template>

<style scoped>
.quiz-container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.quiz-header {
    text-align: center;
    margin-bottom: 30px;
}

.quiz-header h1 {
    color: #333;
    margin-bottom: 10px;
}

.quiz-header p {
    color: #666;
    font-size: 1.1rem;
}

.progress {
  margin-top: 10px;
  padding: 10px;
  background: rgba(102, 126, 234, 0.1);
  border-radius: 5px;
  color: #667eea;
  font-weight: bold;
}

.question {
    background: white;
    border-radius: 10px;
    padding: 20px;
    margin-bottom: 20px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.question.answered {
  border-left: 4px solid #667eea;
}

.question h3 {
    color: #333;
    margin-bottom: 15px;
    font-size: 1.2rem;
}

.options {
    display: flex;
    flex-direction: column;
    gap: 10px;
}

.option {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 10px;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.2s;
}

.option:hover {
    background-color: #f8f9fa;
}

.option input[type="radio"] {
    margin: 0;
}

.option span {
    color: #333;
    font-size: 1rem;
}

.option.correct {
  background-color: #d4edda;
  border: 2px solid #28a745;
}

.option.incorrect {
  background-color: #f8d7da;
  border: 2px solid #dc3545;
}

.option.disabled {
  cursor: not-allowed;
  opacity: 0.8;
}

.option.disabled input {
  cursor: not-allowed;
}

.submit-btn,
.reset-btn {
    background: linear-gradient(45deg, #667eea, #764ba2);
    color: white;
    border: none;
    padding: 15px 30px;
    border-radius: 10px;
    font-size: 1.1rem;
    cursor: pointer;
    margin-top: 20px;
    transition: transform 0.2s;
}

.submit-btn:hover,
.reset-btn:hover {
    transform: translateY(-2px);
}

.submit-btn:disabled {
    background: #ccc;
    cursor: not-allowed;
    transform: none;
}

.results {
    text-align: center;
}

.score {
    background: white;
    border-radius: 10px;
    padding: 30px;
    margin-bottom: 30px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.score p {
    font-size: 1.5rem;
    color: #333;
    margin-bottom: 10px;
}

.percentage {
    font-size: 2rem !important;
    font-weight: bold;
    color: #667eea;
}

.answer-review {
    text-align: left;
    margin-bottom: 20px;
}

.answer-review h3 {
    text-align: center;
    margin-bottom: 20px;
    color: #333;
}

.question-review {
    background: white;
    border-radius: 10px;
    padding: 15px;
    margin-bottom: 15px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.question-review h4 {
    color: #333;
    margin-bottom: 10px;
}

.correct-answer {
    color: #28a745;
    font-weight: bold;
    margin-bottom: 5px;
}

.user-answer {
    margin-bottom: 0;
}

.user-answer.correct {
    color: #28a745;
    font-weight: bold;
}

.user-answer.incorrect {
    color: #dc3545;
    font-weight: bold;
}

.loading {
    text-align: center;
    padding: 50px;
    font-size: 1.2rem;
    color: #666;
}

.feedback {
  margin-left: auto;
  font-weight: bold;
  font-size: 1.2rem;
}

.correct-mark {
  color: #28a745;
}

.incorrect-mark {
  color: #dc3545;
}

.instant-feedback {
  margin-top: 15px;
  padding: 10px;
  border-radius: 5px;
  font-weight: bold;
}

.correct-feedback {
  color: #28a745;
  background-color: #d4edda;
  border: 1px solid #c3e6cb;
}

.incorrect-feedback {
  color: #721c24;
  background-color: #f8d7da;
  border: 1px solid #f5c6cb;
}

.final-results {
  text-align: center;
  padding: 30px;
}

.final-results h2 {
  color: #333;
  margin-bottom: 20px;
}

.file-upload {
  text-align: center;
  margin-top: 20px;
}

.file-upload input {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 1rem;
}

.file-upload .note {
  margin-top: 10px;
  color: #666;
  font-size: 0.9rem;
}

.upload-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.upload-btn {
  background: linear-gradient(45deg, #667eea, #764ba2);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  font-size: 1rem;
  cursor: pointer;
  transition: transform 0.2s;
}

.upload-btn:hover {
  transform: translateY(-2px);
}

.back-btn {
  background: #f8f9fa;
  color: #333;
  border: 1px solid #ccc;
  padding: 10px 20px;
  border-radius: 5px;
  font-size: 1rem;
  cursor: pointer;
  transition: background-color 0.2s;
}

.back-btn:hover {
  background-color: #e2e6ea;
}

.upload-note {
  color: #666;
  font-size: 0.9rem;
  text-align: center;
  margin-top: 10px;
}
</style>