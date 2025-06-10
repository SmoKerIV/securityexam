<script setup lang="ts">
interface Props {
    title: string
    description: string
    progress: number
    total: number
}

defineProps<Props>()
</script>

<template>
    <div class="quiz-header">
        <h1>{{ title }}</h1>
        <p>{{ description }}</p>
        <div class="progress">
            <span>Progress: {{ progress }} / {{ total }}</span>
            <div class="progress-bar">
                <div class="progress-fill" :style="{ width: `${(progress / total) * 100}%` }"></div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.quiz-header {
    text-align: center;
    margin-bottom: 30px;
    background: rgba(15, 23, 42, 0.95);
    backdrop-filter: blur(20px);
    border-radius: 24px;
    padding: 30px;
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.3);
    border: 1px solid rgba(59, 130, 246, 0.2);
}

.quiz-header h1 {
    color: #f1f5f9;
    margin-bottom: 16px;
    font-size: 3rem;
    font-weight: 800;
    background: linear-gradient(135deg, #60a5fa, #a78bfa);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    line-height: 1.2;
}

.quiz-header p {
    color: #cbd5e1;
    font-size: 1.3rem;
    font-weight: 500;
    line-height: 1.6;
    max-width: 800px;
    margin: 0 auto 20px;
}

.progress {
    margin-top: 20px;
    padding: 16px 24px;
    background: linear-gradient(135deg, rgba(59, 130, 246, 0.15), rgba(147, 51, 234, 0.15));
    border-radius: 12px;
    color: #60a5fa;
    font-weight: 600;
    font-size: 1rem;
    border: 1px solid rgba(59, 130, 246, 0.3);
}

.progress-bar {
    width: 100%;
    height: 8px;
    background: rgba(59, 130, 246, 0.2);
    border-radius: 4px;
    margin-top: 12px;
    overflow: hidden;
}

.progress-fill {
    height: 100%;
    background: linear-gradient(135deg, #60a5fa, #a78bfa);
    border-radius: 4px;
    transition: width 0.3s ease;
}

@media (max-width: 768px) {
    .quiz-header {
        padding: 20px;
    }

    .quiz-header h1 {
        font-size: 2rem;
    }

    .quiz-header p {
        font-size: 1.1rem;
    }
}
</style>
