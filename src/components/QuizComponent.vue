<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import QuizHeader from './quiz/QuizHeader.vue'
import QuizActions from './quiz/QuizActions.vue'
import QuizQuestion from './quiz/QuizQuestion.vue'
import QuizResults from './quiz/QuizResults.vue'
import SavedQuizzesList from './quiz/SavedQuizzesList.vue'
import ShareQuiz from './ShareQuiz.vue'
import Modal from './ui/Modal.vue'

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
const savedQuizzes = ref<Array<{ id: string, title: string, fileName: string }>>([])
const showSavedQuizzes = ref(false)
const showSharePopup = ref(false)

// Load saved quizzes from localStorage on component mount
onMounted(() => {
  loadSavedQuizzes()

  const urlParams = new URLSearchParams(window.location.search)
  const sharedQuizId = urlParams.get('quiz')

  if (sharedQuizId) {
    loadSavedQuiz(sharedQuizId)
  }
})

// Use props quiz data if available, otherwise use uploaded data
const currentQuizData = computed(() => props.quizData || uploadedQuizData.value)

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

const emit = defineEmits<{
  quizUploaded: [quizData: any]
}>()

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
      emit('quizUploaded', jsonData)
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

const loadSavedQuizzes = () => {
  const saved = localStorage.getItem('savedQuizzes')
  if (saved) {
    savedQuizzes.value = JSON.parse(saved)
  }
}

const saveQuiz = () => {
  if (!currentQuizData.value) return

  const quizId = Date.now().toString()
  const fileName = `quiz_${quizId}.json`

  // Save to localStorage
  const quizData = JSON.stringify(currentQuizData.value, null, 2)
  localStorage.setItem(`quiz_${quizId}`, quizData)

  // Add to saved quizzes list
  const quizInfo = {
    id: quizId,
    title: currentQuizData.value.title,
    fileName: fileName
  }

  savedQuizzes.value.push(quizInfo)
  localStorage.setItem('savedQuizzes', JSON.stringify(savedQuizzes.value))

  alert(`Quiz saved successfully! ID: ${quizId}`)
}

const loadSavedQuiz = (quizId: string) => {
  const savedQuiz = localStorage.getItem(`quiz_${quizId}`)
  if (savedQuiz) {
    uploadedQuizData.value = JSON.parse(savedQuiz)
    selectedAnswers.value = {}
    showFinalResults.value = false
    showSavedQuizzes.value = false
  }
}

const deleteSavedQuiz = (quizId: string) => {
  // Remove from localStorage
  localStorage.removeItem(`quiz_${quizId}`)

  // Remove from saved quizzes list
  savedQuizzes.value = savedQuizzes.value.filter(quiz => quiz.id !== quizId)
  localStorage.setItem('savedQuizzes', JSON.stringify(savedQuizzes.value))
}

const downloadQuiz = (quizId: string) => {
  const savedQuiz = localStorage.getItem(`quiz_${quizId}`)
  if (savedQuiz) {
    try {
      const blob = new Blob([savedQuiz], { type: 'application/json' })
      const url = URL.createObjectURL(blob)

      // Use a cleaner download approach
      const link = document.createElement('a')
      link.href = url
      link.download = `quiz_${quizId}.json`
      link.click()

      URL.revokeObjectURL(url)
    } catch (error) {
      console.error('Download failed:', error)
    }
  }
}

const compressQuizData = (quizData: any): string => {
  const compressed = {
    t: quizData.title,
    d: quizData.description,
    q: quizData.questions.map((question: any) => ({
      i: question.id,
      q: question.question,
      o: question.options,
      a: question.correctAnswer
    }))
  }

  return JSON.stringify(compressed)
}

const copyShareLink = (quizId: string) => {
  const savedQuiz = localStorage.getItem(`quiz_${quizId}`)
  if (savedQuiz) {
    try {
      const quizData = JSON.parse(savedQuiz)

      // Compress the quiz data
      const compressedJson = compressQuizData(quizData)
      const encodedQuiz = btoa(compressedJson)

      const baseUrl = window.location.origin + window.location.pathname
      const shareUrl = `${baseUrl}?q=${encodedQuiz}`

      navigator.clipboard.writeText(shareUrl).then(() => {
        alert('Compressed share link copied to clipboard!')
      }).catch(() => {
        prompt('Copy this link to share:', shareUrl)
      })
    } catch (error) {
      console.error('Error generating share link:', error)
      alert('Error generating share link')
    }
  }
}

const exportCurrentQuiz = () => {
  if (!currentQuizData.value) return

  try {
    const dataStr = JSON.stringify(currentQuizData.value, null, 2)
    const blob = new Blob([dataStr], { type: 'application/json' })
    const url = URL.createObjectURL(blob)

    // Use a cleaner download approach
    const link = document.createElement('a')
    link.href = url
    link.download = `${currentQuizData.value.title.replace(/[^a-z0-9]/gi, '_').toLowerCase()}.json`
    link.click()

    URL.revokeObjectURL(url)
  } catch (error) {
    console.error('Export failed:', error)
  }
}

// Event handlers for child components
const handleSelectAnswer = (questionId: number, answerIndex: number) => {
  selectAnswer(questionId, answerIndex)
}

const handleUploadFile = (event: Event) => {
  handleFileUpload(event)
}

const handleShowSavedQuizzes = () => {
  showSavedQuizzes.value = true
}

const handleSaveQuiz = () => {
  saveQuiz()
}

const handleExportQuiz = () => {
  exportCurrentQuiz()
}

const handleShareQuiz = () => {
  showSharePopup.value = true
}

const handleResetQuiz = () => {
  resetQuiz()
}

const handleLoadQuiz = (quizId: string) => {
  loadSavedQuiz(quizId)
}

const handleDownloadQuiz = (quizId: string) => {
  downloadQuiz(quizId)
}

const handleShareSavedQuiz = (quizId: string) => {
  copyShareLink(quizId)
}

const handleDeleteQuiz = (quizId: string) => {
  deleteSavedQuiz(quizId)
}

// XSS demonstration
const xssInput = ref('')
const xssOutput = ref('')

const handleXssDemo = () => {
  // Deliberately vulnerable - this demonstrates XSS
  xssOutput.value = xssInput.value
}

const clearXssDemo = () => {
  xssInput.value = ''
  xssOutput.value = ''
}
</script>

<template>
  <div class="quiz-container" v-if="currentQuizData">
    <QuizHeader :title="currentQuizData.title" :description="currentQuizData.description"
      :progress="Object.keys(selectedAnswers).length" :total="currentQuizData.questions.length" />

    <QuizActions v-if="!showFinalResults" @uploadFile="handleUploadFile" @showSavedQuizzes="handleShowSavedQuizzes"
      @saveQuiz="handleSaveQuiz" @exportQuiz="handleExportQuiz" @shareQuiz="handleShareQuiz" />

    <!-- Share Popup -->
    <Modal :show="showSharePopup" title="Share Quiz" max-width="500px" @close="showSharePopup = false">
      <ShareQuiz :quiz-data="currentQuizData" />
    </Modal>

    <div class="questions" v-if="!showFinalResults">
      <QuizQuestion v-for="question in currentQuizData.questions" :key="question.id" :question="question"
        :selected-answer="selectedAnswers[question.id]" :is-answered="isAnswered(question.id)"
        @selectAnswer="(answerIndex) => handleSelectAnswer(question.id, answerIndex)" />
    </div>

    <QuizResults v-if="showFinalResults" :score="score" :total="currentQuizData.questions.length"
      @resetQuiz="handleResetQuiz" @saveQuiz="handleSaveQuiz" @exportQuiz="handleExportQuiz"
      @shareQuiz="handleShareQuiz" @showSavedQuizzes="handleShowSavedQuizzes" />
  </div>

  <div class="no-quiz" v-else>
    <div class="upload-prompt">
      <h2>📚 Welcome to the Quiz Platform</h2>
      <p>Upload a JSON file to start your quiz or load a saved quiz</p>

      <div class="upload-section">
        <label for="quiz-upload" class="upload-btn-large">
          📁 Upload Quiz File (JSON)
        </label>
        <input id="quiz-upload" type="file" @change="handleFileUpload" accept=".json" style="display: none;" />
        <button @click="showSavedQuizzes = true" class="saved-btn-large">
          📂 Load Saved Quiz
        </button>
      </div>

      <!-- XSS Demonstration Section -->
      <div class="xss-demo-section">
        <h3>⚠️ XSS Vulnerability Demonstration</h3>
        <p class="xss-warning">
          <strong>Warning:</strong> This is an intentionally vulnerable input field for educational purposes.
          Try entering: <code>&lt;img src=x onerror=alert('XSS!')&gt;</code>
        </p>
        
        <div class="xss-input-section">
          <div class="input-group">
            <label for="xss-input">Enter potentially malicious content:</label>
            <input 
              id="xss-input"
              v-model="xssInput" 
              type="text" 
              class="xss-input"
              placeholder="Try: <script>alert('XSS!')</script> or <img src=x onerror=alert('XSS!')>"
            />
          </div>
          
          <div class="xss-buttons">
            <button @click="handleXssDemo" class="demo-btn" :disabled="!xssInput">
              🔓 Execute (Unsafe)
            </button>
            <button @click="clearXssDemo" class="clear-btn">
              🧹 Clear
            </button>
          </div>
        </div>

        <div v-if="xssOutput" class="xss-output-section">
          <h4>Vulnerable Output (Raw HTML):</h4>
          <div class="xss-output vulnerable" v-html="xssOutput"></div>
          
          <h4>Safe Output (Escaped):</h4>
          <div class="xss-output safe">{{ xssOutput }}</div>
        </div>

        <div class="xss-explanation">
          <h4>🛡️ Security Lesson:</h4>
          <ul>
            <li><strong>Vulnerability:</strong> The "Unsafe" output uses <code>v-html</code> which executes scripts</li>
            <li><strong>Protection:</strong> The "Safe" output uses text interpolation <code>{{}}</code> which escapes HTML</li>
            <li><strong>Best Practice:</strong> Always sanitize user input and avoid <code>v-html</code> with user data</li>
            <li><strong>Real Impact:</strong> XSS can steal cookies, session tokens, or redirect users to malicious sites</li>
          </ul>
        </div>
      </div>

      <div class="format-info">
        <h3>Expected JSON Format:</h3>
        <pre class="format-example">{
  "title": "Your Quiz Title",
  "description": "Quiz description",
  "questions": [
    {
      "id": 1,
      "question": "Your question?",
      "options": ["Option A", "Option B", "Option C", "Option D"],
      "correctAnswer": 2
    }
  ]
}</pre>
        <p class="format-note">
          <strong>Note:</strong> correctAnswer is the index (0-based) of the correct option
        </p>
      </div>
    </div>
  </div>

  <!-- Saved Quizzes Modal -->
  <Modal :show="showSavedQuizzes" title="Saved Quizzes" @close="showSavedQuizzes = false">
    <SavedQuizzesList :quizzes="savedQuizzes" @loadQuiz="handleLoadQuiz" @downloadQuiz="handleDownloadQuiz"
      @shareQuiz="handleShareSavedQuiz" @deleteQuiz="handleDeleteQuiz" />
  </Modal>
</template>

<style scoped>
.quiz-container {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Inter', 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  min-height: calc(100vh - 140px);
  display: flex;
  flex-direction: column;
}

.questions {
  display: flex;
  flex-direction: column;
}

.no-quiz {
  text-align: center;
  padding: 60px 20px;
  color: #cbd5e1;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: calc(100vh - 200px);
}

.upload-prompt {
  max-width: 700px;
  margin: 0 auto;
  padding: 40px;
  background: rgba(15, 23, 42, 0.95);
  backdrop-filter: blur(20px);
  border-radius: 24px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  border: 1px solid rgba(59, 130, 246, 0.2);
  position: relative;
  overflow: hidden;
}

.upload-prompt::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: radial-gradient(circle at 50% 0%, rgba(59, 130, 246, 0.1) 0%, transparent 50%);
  pointer-events: none;
}

.upload-prompt>* {
  position: relative;
  z-index: 1;
}

.upload-prompt h2 {
  color: #f1f5f9;
  margin-bottom: 16px;
  font-size: 2.5rem;
  font-weight: 700;
  background: linear-gradient(135deg, #60a5fa, #a78bfa);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.upload-prompt p {
  color: #cbd5e1;
  margin-bottom: 32px;
  font-size: 1.2rem;
  line-height: 1.6;
}

.upload-section {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 16px;
  align-items: center;
  margin-bottom: 32px;
}

.upload-btn-large,
.saved-btn-large {
  background: linear-gradient(135deg, #3b82f6, #6366f1);
  color: white;
  border: none;
  padding: 16px 32px;
  border-radius: 16px;
  font-size: 1.1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 8px 25px rgba(59, 130, 246, 0.4);
  text-decoration: none;
  display: inline-flex;
  align-items: center;
  gap: 12px;
  min-width: 200px;
  justify-content: center;
}

.upload-btn-large:hover,
.saved-btn-large:hover {
  transform: translateY(-3px);
  box-shadow: 0 12px 35px rgba(59, 130, 246, 0.6);
}

.format-info {
  text-align: left;
  margin-top: 32px;
  background: rgba(30, 41, 59, 0.8);
  padding: 24px;
  border-radius: 16px;
  border: 1px solid rgba(59, 130, 246, 0.3);
}

.format-info h3 {
  color: #f1f5f9;
  margin-bottom: 16px;
  font-size: 1.3rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 8px;
}

.format-info h3::before {
  content: "📋";
  font-size: 1.2em;
}

.format-example {
  background: rgba(51, 65, 85, 0.8);
  padding: 20px;
  border-radius: 12px;
  overflow-x: auto;
  font-family: 'Fira Code', 'Monaco', 'Consolas', monospace;
  font-size: 0.9rem;
  line-height: 1.6;
  border: 1px solid rgba(59, 130, 246, 0.2);
  color: #e2e8f0;
  position: relative;
}

.format-example::before {
  content: 'JSON';
  position: absolute;
  top: 8px;
  right: 12px;
  background: rgba(59, 130, 246, 0.8);
  color: white;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 0.7rem;
  font-weight: 600;
}

.format-note {
  margin-top: 16px;
  color: #cbd5e1;
  font-size: 0.95rem;
  padding: 16px;
  background: rgba(59, 130, 246, 0.1);
  border-radius: 12px;
  border-left: 4px solid #60a5fa;
  display: flex;
  align-items: flex-start;
  gap: 12px;
}

.format-note::before {
  content: "💡";
  font-size: 1.2em;
  flex-shrink: 0;
}

.format-note strong {
  color: #60a5fa;
}

.xss-demo-section {
  margin-top: 32px;
  padding: 24px;
  background: rgba(127, 29, 29, 0.1);
  border: 2px solid rgba(239, 68, 68, 0.3);
  border-radius: 16px;
  text-align: left;
}

.xss-demo-section h3 {
  color: #fca5a5;
  margin-bottom: 16px;
  font-size: 1.3rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 8px;
}

.xss-warning {
  background: rgba(245, 158, 11, 0.1);
  border: 1px solid rgba(245, 158, 11, 0.3);
  border-radius: 8px;
  padding: 12px;
  margin-bottom: 20px;
  color: #fbbf24;
  font-size: 0.9rem;
}

.xss-warning code {
  background: rgba(0, 0, 0, 0.2);
  padding: 2px 6px;
  border-radius: 4px;
  font-family: 'Courier New', monospace;
  color: #fcd34d;
}

.xss-input-section {
  margin-bottom: 20px;
}

.input-group {
  margin-bottom: 12px;
}

.input-group label {
  display: block;
  margin-bottom: 8px;
  color: #e2e8f0;
  font-weight: 500;
}

.xss-input {
  width: 100%;
  padding: 12px;
  border: 2px solid rgba(239, 68, 68, 0.3);
  border-radius: 8px;
  background: rgba(15, 23, 42, 0.8);
  color: #e2e8f0;
  font-family: 'Courier New', monospace;
  font-size: 0.9rem;
}

.xss-input:focus {
  outline: none;
  border-color: #ef4444;
  box-shadow: 0 0 0 3px rgba(239, 68, 68, 0.2);
}

.xss-buttons {
  display: flex;
  gap: 12px;
}

.demo-btn {
  background: linear-gradient(135deg, #dc3545, #c82333);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
  box-shadow: 0 4px 12px rgba(220, 53, 69, 0.3);
}

.demo-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(220, 53, 69, 0.4);
}

.demo-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  transform: none;
}

.clear-btn {
  background: linear-gradient(135deg, #6b7280, #4b5563);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
  box-shadow: 0 4px 12px rgba(107, 114, 128, 0.3);
}

.clear-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(107, 114, 128, 0.4);
}

.xss-output-section {
  margin: 20px 0;
}

.xss-output-section h4 {
  color: #e2e8f0;
  margin: 16px 0 8px 0;
  font-size: 1rem;
}

.xss-output {
  padding: 12px;
  border-radius: 8px;
  margin-bottom: 12px;
  font-family: 'Courier New', monospace;
  font-size: 0.9rem;
  min-height: 40px;
  word-break: break-all;
}

.xss-output.vulnerable {
  background: rgba(127, 29, 29, 0.2);
  border: 1px solid rgba(239, 68, 68, 0.5);
  color: #fca5a5;
}

.xss-output.safe {
  background: rgba(6, 78, 59, 0.2);
  border: 1px solid rgba(16, 185, 129, 0.5);
  color: #34d399;
}

.xss-explanation {
  background: rgba(30, 41, 59, 0.8);
  padding: 20px;
  border-radius: 12px;
  border: 1px solid rgba(59, 130, 246, 0.3);
  margin-top: 20px;
}

.xss-explanation h4 {
  color: #60a5fa;
  margin-bottom: 12px;
  font-size: 1.1rem;
}

.xss-explanation ul {
  list-style: none;
  padding: 0;
}

.xss-explanation li {
  margin-bottom: 8px;
  color: #cbd5e1;
  font-size: 0.9rem;
  line-height: 1.5;
}

.xss-explanation code {
  background: rgba(0, 0, 0, 0.3);
  padding: 2px 6px;
  border-radius: 4px;
  font-family: 'Courier New', monospace;
  color: #fcd34d;
}

@media (max-width: 768px) {
  .upload-prompt {
    padding: 30px 20px;
    margin: 20px;
  }

  .upload-prompt h2 {
    font-size: 2rem;
  }

  .upload-prompt p {
    font-size: 1.1rem;
  }

  .upload-section {
    flex-direction: column;
    gap: 12px;
  }

  .upload-btn-large,
  .saved-btn-large {
    width: 100%;
    padding: 14px 24px;
    font-size: 1rem;
  }

  .format-info {
    padding: 20px;
  }

  .xss-demo-section {
    padding: 20px;
  }

  .xss-buttons {
    flex-direction: column;
  }

  .demo-btn,
  .clear-btn {
    width: 100%;
  }
}

@media (max-width: 480px) {
  .upload-prompt h2 {
    font-size: 1.8rem;
  }

  .upload-prompt p {
    font-size: 1rem;
  }

  .format-example {
    font-size: 0.8rem;
    padding: 16px;
  }
}
</style>