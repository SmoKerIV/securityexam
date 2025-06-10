<script setup lang="ts">
interface Question {
    id: number
    question: string
    options: string[]
    correctAnswer: number
}

interface Props {
    question: Question
    selectedAnswer?: number
    isAnswered: boolean
}

const props = defineProps<Props>()
const emit = defineEmits<{
    selectAnswer: [answerIndex: number]
}>()

const isCorrect = (optionIndex: number) => {
    return props.question.correctAnswer === optionIndex
}

const isSelected = (optionIndex: number) => {
    return props.selectedAnswer === optionIndex
}
</script>

<template>
    <div class="question" :class="{ 'answered': isAnswered }">
        <h3>{{ question.id }}. {{ question.question }}</h3>
        <div class="options">
            <label v-for="(option, index) in question.options" :key="index" class="option" :class="{
                'correct': isAnswered && isCorrect(index),
                'incorrect': isAnswered && isSelected(index) && !isCorrect(index),
                'disabled': isAnswered
            }">
                <input type="radio" :name="`question-${question.id}`" :value="index" :disabled="isAnswered"
                    @change="emit('selectAnswer', index)" />
                <span>{{ option }}</span>
                <span v-if="isAnswered && isCorrect(index)" class="feedback correct-mark">✓</span>
                <span v-if="isAnswered && isSelected(index) && !isCorrect(index)"
                    class="feedback incorrect-mark">✗</span>
            </label>
        </div>

        <div v-if="isAnswered" class="instant-feedback">
            <p v-if="selectedAnswer === question.correctAnswer" class="correct-feedback">
                ✓ Correct!
            </p>
            <p v-else class="incorrect-feedback">
                ✗ Incorrect. The correct answer is: {{ question.options[question.correctAnswer] }}
            </p>
        </div>
    </div>
</template>

<style scoped>
.question {
    background: rgba(15, 23, 42, 0.9);
    backdrop-filter: blur(20px);
    border-radius: 20px;
    padding: 32px;
    margin-bottom: 24px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
    border: 1px solid rgba(59, 130, 246, 0.2);
    transition: all 0.3s ease;
}

.question:hover {
    transform: translateY(-2px);
    box-shadow: 0 15px 40px rgba(0, 0, 0, 0.4);
    border-color: rgba(59, 130, 246, 0.3);
}

.question.answered {
    border-left: 6px solid #10b981;
    background: rgba(15, 23, 42, 0.95);
    box-shadow: 0 12px 35px rgba(16, 185, 129, 0.2);
    transform: scale(1.01);
    border: 2px solid rgba(16, 185, 129, 0.3);
}

.question.answered:hover {
    transform: scale(1.01) translateY(-1px);
    box-shadow: 0 15px 45px rgba(16, 185, 129, 0.25);
}

.question.answered h3 {
    color: #34d399;
    position: relative;
}

.question.answered h3::after {
    content: "✓";
    position: absolute;
    right: 0;
    top: 0;
    color: #10b981;
    font-size: 1.2em;
    font-weight: 700;
    animation: checkMark 0.5s ease-in-out;
}

@keyframes checkMark {
    0% {
        opacity: 0;
        transform: scale(0) rotate(-180deg);
    }

    50% {
        transform: scale(1.2) rotate(0deg);
    }

    100% {
        opacity: 1;
        transform: scale(1) rotate(0deg);
    }
}

.question h3 {
    color: #f1f5f9;
    margin-bottom: 24px;
    font-size: 1.4rem;
    font-weight: 600;
    line-height: 1.5;
}

.options {
    display: flex;
    flex-direction: column;
    gap: 16px;
}

.option {
    display: flex;
    align-items: center;
    gap: 16px;
    padding: 18px 24px;
    border-radius: 16px;
    cursor: pointer;
    transition: all 0.3s ease;
    background: rgba(30, 41, 59, 0.6);
    border: 2px solid transparent;
    backdrop-filter: blur(10px);
}

.option:hover:not(.disabled) {
    background: rgba(30, 41, 59, 0.8);
    transform: translateX(8px);
    border-color: rgba(59, 130, 246, 0.4);
}

.option input[type="radio"] {
    margin: 0;
    width: 20px;
    height: 20px;
    accent-color: #60a5fa;
}

.option span {
    color: #e2e8f0;
    font-size: 1.1rem;
    font-weight: 500;
    flex: 1;
}

.option.correct {
    background: rgba(6, 78, 59, 0.9);
    border: 2px solid #10b981;
    color: #34d399;
    font-weight: 700;
    transform: translateX(8px) scale(1.01);
    box-shadow: 0 6px 20px rgba(16, 185, 129, 0.3);
    position: relative;
    overflow: hidden;
    backdrop-filter: blur(4px);
}

.option.correct::before {
    content: "";
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(90deg, transparent, rgba(16, 185, 129, 0.2), transparent);
    animation: shimmer 1.5s ease-in-out;
}

@keyframes shimmer {
    0% {
        left: -100%;
    }

    100% {
        left: 100%;
    }
}

.option.incorrect {
    background: rgba(127, 29, 29, 0.9);
    border: 2px solid #ef4444;
    color: #fca5a5;
    font-weight: 700;
    transform: translateX(8px) scale(1.01);
    box-shadow: 0 6px 20px rgba(239, 68, 68, 0.3);
    animation: shake 0.5s ease-in-out;
    backdrop-filter: blur(4px);
}

@keyframes shake {

    0%,
    100% {
        transform: translateX(8px) scale(1.01);
    }

    25% {
        transform: translateX(4px) scale(1.01);
    }

    75% {
        transform: translateX(12px) scale(1.01);
    }
}

.option.disabled {
    cursor: not-allowed;
    opacity: 0.85;
    transition: all 0.2s ease;
}

.option.disabled:not(.correct):not(.incorrect) {
    background: linear-gradient(135deg, rgba(51, 65, 85, 0.6) 0%, rgba(71, 85, 105, 0.4) 100%);
    color: #64748b;
    border: 2px solid rgba(71, 85, 105, 0.4);
    opacity: 0.7;
    transform: translateX(2px);
    filter: grayscale(0.3);
}

.feedback {
    margin-left: auto;
    font-weight: 700;
    font-size: 1.4rem;
    text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.correct-mark {
    color: #28a745;
}

.incorrect-mark {
    color: #dc3545;
}

.instant-feedback {
    margin-top: 24px;
    padding: 24px 28px;
    border-radius: 16px;
    font-weight: 700;
    font-size: 1.1rem;
    border-left: 6px solid;
    backdrop-filter: blur(4px);
    animation: slideInUp 0.6s cubic-bezier(0.4, 0, 0.2, 1);
    position: relative;
    overflow: hidden;
}

@keyframes slideInUp {
    from {
        opacity: 0;
        transform: translateY(20px) scale(0.95);
    }

    to {
        opacity: 1;
        transform: translateY(0) scale(1);
    }
}

.correct-feedback {
    color: #34d399;
    background: rgba(6, 78, 59, 0.9);
    border-left-color: #10b981;
    box-shadow: 0 10px 30px rgba(16, 185, 129, 0.2);
    border: 2px solid rgba(16, 185, 129, 0.3);
}

.correct-feedback::before {
    content: "🎉";
    position: absolute;
    right: 20px;
    top: 50%;
    transform: translateY(-50%);
    font-size: 1.5em;
    animation: celebrate 2s ease-in-out infinite;
}

@keyframes celebrate {

    0%,
    100% {
        transform: translateY(-50%) scale(1) rotate(0deg);
    }

    50% {
        transform: translateY(-50%) scale(1.1) rotate(10deg);
    }
}

.incorrect-feedback {
    color: #fca5a5;
    background: rgba(127, 29, 29, 0.9);
    border-left-color: #ef4444;
    box-shadow: 0 10px 30px rgba(239, 68, 68, 0.2);
    border: 2px solid rgba(239, 68, 68, 0.3);
}

.incorrect-feedback::before {
    content: "💡";
    position: absolute;
    right: 20px;
    top: 50%;
    transform: translateY(-50%);
    font-size: 1.5em;
    animation: pulse 2s ease-in-out infinite;
}

@keyframes pulse {

    0%,
    100% {
        opacity: 1;
        transform: translateY(-50%) scale(1);
    }

    50% {
        opacity: 0.7;
        transform: translateY(-50%) scale(1.1);
    }
}

@media (max-width: 768px) {
    .question {
        padding: 20px;
        margin-bottom: 16px;
    }

    .question h3 {
        font-size: 1.2rem;
    }

    .option {
        padding: 14px 18px;
    }

    .option span {
        font-size: 1rem;
    }
}
</style>
