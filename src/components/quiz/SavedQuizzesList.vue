<script setup lang="ts">
interface SavedQuiz {
    id: string
    title: string
    fileName: string
}

interface Props {
    quizzes: SavedQuiz[]
}

defineProps<Props>()
const emit = defineEmits<{
    loadQuiz: [quizId: string]
    downloadQuiz: [quizId: string]
    shareQuiz: [quizId: string]
    deleteQuiz: [quizId: string]
}>()
</script>

<template>
    <div v-if="quizzes.length === 0" class="no-saved-quizzes">
        <p>No saved quizzes found.</p>
    </div>

    <div v-else class="quiz-list">
        <div v-for="quiz in quizzes" :key="quiz.id" class="quiz-item">
            <div class="quiz-info">
                <h3>{{ quiz.title }}</h3>
                <p class="file-name">{{ quiz.fileName }}</p>
            </div>
            <div class="quiz-actions">
                <button @click="emit('loadQuiz', quiz.id)" class="load-btn">Load</button>
                <button @click="emit('downloadQuiz', quiz.id)" class="download-btn">Download</button>
                <button @click="emit('shareQuiz', quiz.id)" class="share-btn">Share</button>
                <button @click="emit('deleteQuiz', quiz.id)" class="delete-btn">Delete</button>
            </div>
        </div>
    </div>
</template>

<style scoped>
.quiz-list {
    margin-top: 24px;
}

.quiz-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 20px;
    margin-bottom: 16px;
    background: rgba(30, 41, 59, 0.8);
    border-radius: 16px;
    border: 1px solid rgba(59, 130, 246, 0.2);
    transition: all 0.3s ease;
}

.quiz-item:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.3);
    background: rgba(30, 41, 59, 0.9);
    border-color: rgba(59, 130, 246, 0.3);
}

.quiz-item:last-child {
    margin-bottom: 0;
}

.quiz-info h3 {
    color: #f1f5f9;
    margin-bottom: 4px;
    font-size: 1.2rem;
    font-weight: 600;
}

.file-name {
    color: #94a3b8;
    font-size: 0.9rem;
    margin: 0;
}

.quiz-actions {
    display: flex;
    gap: 8px;
}

.load-btn,
.download-btn,
.share-btn,
.delete-btn {
    color: white;
    border: none;
    padding: 8px 16px;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.3s ease;
    font-size: 0.85rem;
    font-weight: 600;
}

.load-btn,
.download-btn,
.share-btn {
    background: linear-gradient(135deg, #3b82f6, #6366f1);
    box-shadow: 0 2px 8px rgba(59, 130, 246, 0.3);
}

.load-btn:hover,
.download-btn:hover,
.share-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 15px rgba(59, 130, 246, 0.5);
}

.delete-btn {
    background: linear-gradient(135deg, #dc3545, #c82333);
    box-shadow: 0 2px 8px rgba(220, 53, 69, 0.3);
}

.delete-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 15px rgba(220, 53, 69, 0.5);
}

.no-saved-quizzes {
    text-align: center;
    padding: 40px 20px;
    color: #cbd5e1;
    font-size: 1.1rem;
}

@media (max-width: 768px) {
    .quiz-item {
        flex-direction: column;
        align-items: flex-start;
        gap: 12px;
    }

    .quiz-actions {
        width: 100%;
        justify-content: flex-start;
    }
}
</style>
