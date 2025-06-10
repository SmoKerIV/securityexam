<script setup lang="ts">
const emit = defineEmits<{
    uploadFile: [event: Event]
    showSavedQuizzes: []
    saveQuiz: []
    exportQuiz: []
    shareQuiz: []
}>()
</script>

<template>
    <div class="quiz-actions">
        <div class="upload-section">
            <label for="quiz-upload" class="upload-btn">
                📁 Upload New Quiz
            </label>
            <input id="quiz-upload" type="file" @change="emit('uploadFile', $event)" accept=".json"
                style="display: none;" />
            <button @click="emit('showSavedQuizzes')" class="saved-btn">
                📂 Saved Quizzes
            </button>
            <button @click="emit('saveQuiz')" class="save-btn">
                💾 Save Quiz
            </button>
            <button @click="emit('exportQuiz')" class="export-btn">
                📥 Export Quiz
            </button>
            <button @click="emit('shareQuiz')" class="share-btn">
                🔗 Share Quiz
            </button>
        </div>
    </div>
</template>

<style scoped>
.quiz-actions {
    margin-bottom: 30px;
    padding: 20px;
    background: rgba(15, 23, 42, 0.8);
    backdrop-filter: blur(15px);
    border-radius: 16px;
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.2);
    border: 1px solid rgba(59, 130, 246, 0.2);
}

.upload-section {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 12px;
    align-items: center;
}

.upload-btn,
.saved-btn,
.save-btn,
.export-btn {
    background: linear-gradient(135deg, #3b82f6, #6366f1);
    color: white;
    border: none;
    padding: 12px 24px;
    border-radius: 12px;
    font-size: 0.95rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 4px 15px rgba(59, 130, 246, 0.3);
    text-decoration: none;
    display: inline-flex;
    align-items: center;
    gap: 8px;
}

.share-btn {
    background: linear-gradient(135deg, #10b981, #059669);
    color: white;
    border: none;
    padding: 12px 24px;
    border-radius: 12px;
    font-size: 0.95rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 4px 15px rgba(16, 185, 129, 0.3);
    text-decoration: none;
    display: inline-flex;
    align-items: center;
    gap: 8px;
}

.upload-btn:hover,
.saved-btn:hover,
.save-btn:hover,
.export-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(59, 130, 246, 0.5);
}

.share-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(16, 185, 129, 0.5);
}

@media (max-width: 768px) {
    .upload-section {
        flex-direction: column;
    }

    .quiz-actions {
        padding: 15px;
    }

    .upload-btn,
    .saved-btn,
    .save-btn,
    .export-btn,
    .share-btn {
        padding: 10px 20px;
        font-size: 0.9rem;
    }
}
</style>
