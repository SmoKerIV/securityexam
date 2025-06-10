<script setup lang="ts">
interface Props {
    score: number
    total: number
}

defineProps<Props>()
const emit = defineEmits<{
    resetQuiz: []
    saveQuiz: []
    exportQuiz: []
    shareQuiz: []
    showSavedQuizzes: []
}>()
</script>

<template>
    <div class="final-results">
        <h2>Quiz Complete!</h2>
        <div class="score">
            <p>Final Score: {{ score }} out of {{ total }}</p>
            <p class="percentage">{{ Math.round((score / total) * 100) }}%</p>
        </div>

        <div class="result-actions">
            <button class="reset-btn" @click="emit('resetQuiz')">Try Again</button>
            <button @click="emit('saveQuiz')" class="save-btn">💾 Save Quiz</button>
            <button @click="emit('exportQuiz')" class="export-btn">📥 Export Quiz</button>
            <button @click="emit('shareQuiz')" class="share-btn">🔗 Share Quiz</button>
            <button @click="emit('showSavedQuizzes')" class="saved-btn">📂 Saved Quizzes</button>
        </div>
    </div>
</template>

<style scoped>
.final-results {
    text-align: center;
    padding: 40px;
    background: rgba(15, 23, 42, 0.95);
    backdrop-filter: blur(20px);
    border-radius: 20px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
    border: 1px solid rgba(59, 130, 246, 0.2);
}

.final-results h2 {
    color: #f1f5f9;
    margin-bottom: 30px;
    font-size: 2.2rem;
    font-weight: 700;
}

.score {
    background: rgba(15, 23, 42, 0.95);
    backdrop-filter: blur(20px);
    border-radius: 20px;
    padding: 40px;
    margin-bottom: 30px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
    border: 1px solid rgba(59, 130, 246, 0.2);
}

.score p {
    font-size: 1.6rem;
    color: #f1f5f9;
    margin-bottom: 16px;
    font-weight: 600;
}

.percentage {
    font-size: 3rem !important;
    font-weight: 800 !important;
    background: linear-gradient(135deg, #60a5fa, #a78bfa);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    text-shadow: none;
}

.result-actions {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 12px;
    margin-top: 24px;
}

.reset-btn,
.save-btn,
.export-btn,
.saved-btn {
    background: linear-gradient(135deg, #3b82f6, #6366f1);
    color: white;
    border: none;
    padding: 16px 32px;
    border-radius: 12px;
    font-size: 1.1rem;
    font-weight: 600;
    cursor: pointer;
    margin: 8px;
    transition: all 0.3s ease;
    box-shadow: 0 4px 15px rgba(59, 130, 246, 0.4);
}

.share-btn {
    background: linear-gradient(135deg, #10b981, #059669);
    color: white;
    border: none;
    padding: 16px 32px;
    border-radius: 12px;
    font-size: 1.1rem;
    font-weight: 600;
    cursor: pointer;
    margin: 8px;
    transition: all 0.3s ease;
    box-shadow: 0 4px 15px rgba(16, 185, 129, 0.4);
}

.reset-btn:hover,
.save-btn:hover,
.export-btn:hover,
.saved-btn:hover {
    transform: translateY(-3px);
    box-shadow: 0 8px 25px rgba(59, 130, 246, 0.6);
}

.share-btn:hover {
    transform: translateY(-3px);
    box-shadow: 0 8px 25px rgba(16, 185, 129, 0.6);
}

@media (max-width: 480px) {
    .final-results h2 {
        font-size: 1.8rem;
    }

    .percentage {
        font-size: 2rem !important;
    }
}
</style>
