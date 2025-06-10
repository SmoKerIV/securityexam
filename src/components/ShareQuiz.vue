<!-- filepath: d:\Code\securityexam\src\components\ShareQuiz.vue -->
<script setup lang="ts">
import { ref } from 'vue'

interface Props {
    quizData: any
}

const props = defineProps<Props>()
const isSharing = ref(false)
const shareUrl = ref('')
const showShareUrl = ref(false)

const generateShareLink = () => {
    if (!props.quizData) return

    isSharing.value = true

    try {
        // Encode the quiz data in base64 and add to URL
        const quizJson = JSON.stringify(props.quizData)
        const encodedQuiz = btoa(encodeURIComponent(quizJson))
        const baseUrl = window.location.origin + window.location.pathname
        shareUrl.value = `${baseUrl}?data=${encodedQuiz}`

        // Show the URL immediately instead of trying to copy first
        showShareUrl.value = true

        // Try to copy to clipboard but don't depend on it
        navigator.clipboard.writeText(shareUrl.value).catch(() => {
            // Silently fail - user can still copy manually
            console.log('Clipboard access not available')
        })
    } catch (error) {
        console.error('Error generating share link:', error)
        alert('Error generating share link')
    } finally {
        isSharing.value = false
    }
}

const copyToClipboard = async () => {
    try {
        await navigator.clipboard.writeText(shareUrl.value)
        // Show brief success animation
        const btn = document.querySelector('.copy-btn')
        if (btn) {
            btn.textContent = 'Copied!'
            setTimeout(() => {
                btn.textContent = 'Copy'
            }, 2000)
        }
    } catch (error) {
        // Fallback for older browsers
        const input = document.querySelector('.share-url-input') as HTMLInputElement
        input?.select()
        document.execCommand('copy')
    }
}
</script>

<template>
    <div class="share-quiz">
        <button @click="generateShareLink" :disabled="isSharing || !quizData" class="share-btn">
            <span v-if="isSharing" class="loading-spinner">⏳</span>
            <span v-else class="share-icon">🔗</span>
            <span v-if="isSharing">Generating Link...</span>
            <span v-else>Generate Share Link</span>
        </button>

        <transition name="slide-down">
            <div v-if="showShareUrl" class="share-link-area">
                <div class="success-header">
                    <span class="success-icon">✨</span>
                    <p class="success-text">Share link generated!</p>
                </div>

                <div class="share-url-container">
                    <div class="url-input-wrapper">
                        <input v-model="shareUrl" readonly class="share-url-input"
                            @click="($event.target as HTMLInputElement)?.select()"
                            placeholder="Share link will appear here..." />
                        <span class="input-icon">🔗</span>
                    </div>
                    <button @click="copyToClipboard" class="copy-btn" title="Copy to clipboard">
                        📋 Copy
                    </button>
                </div>

                <div class="share-info">
                    <div class="info-item">
                        <span class="info-icon">👥</span>
                        <span>Anyone with this link can take the same exam</span>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>

<style scoped>
.share-quiz {
    margin-top: 0;
    width: 100%;
}

.share-btn {
    background: linear-gradient(135deg, #10b981 0%, #059669 100%);
    color: white;
    border: none;
    padding: 0.875rem 1.75rem;
    border-radius: 12px;
    font-weight: 600;
    font-size: 1rem;
    cursor: pointer;
    transition: all 0.3s ease;
    display: flex;
    align-items: center;
    gap: 0.5rem;
    justify-content: center;
    width: 100%;
    box-shadow: 0 4px 15px rgba(16, 185, 129, 0.3);
    position: relative;
    overflow: hidden;
}

.share-btn::before {
    content: '';
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
    transition: left 0.5s ease;
}

.share-btn:hover::before {
    left: 100%;
}

.share-btn:hover:not(:disabled) {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(16, 185, 129, 0.5);
}

.share-btn:disabled {
    opacity: 0.7;
    cursor: not-allowed;
    transform: none;
}

.loading-spinner {
    animation: spin 1s linear infinite;
}

@keyframes spin {
    from {
        transform: rotate(0deg);
    }

    to {
        transform: rotate(360deg);
    }
}

.share-icon {
    font-size: 1.1rem;
}

.share-link-area {
    margin-top: 1rem;
    padding: 1.5rem;
    background: linear-gradient(135deg, #1e293b 0%, #334155 100%);
    border: 2px solid rgba(16, 185, 129, 0.3);
    border-radius: 12px;
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.3);
}

.success-header {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    margin-bottom: 1rem;
}

.success-icon {
    font-size: 1.5rem;
    animation: sparkle 2s ease-in-out infinite;
}

@keyframes sparkle {

    0%,
    100% {
        transform: scale(1) rotate(0deg);
    }

    50% {
        transform: scale(1.1) rotate(180deg);
    }
}

.success-text {
    color: #34d399;
    font-weight: 600;
    font-size: 1rem;
    margin: 0;
}

.share-url-container {
    display: flex;
    gap: 0.5rem;
    margin-bottom: 1rem;
    align-items: stretch;
}

.url-input-wrapper {
    flex: 1;
    position: relative;
    display: flex;
    align-items: center;
}

.share-url-input {
    width: 100%;
    padding: 0.875rem 1rem;
    padding-right: 2.5rem;
    border: 2px solid rgba(59, 130, 246, 0.3);
    border-radius: 8px;
    font-size: 0.9rem;
    background: rgba(15, 23, 42, 0.8);
    color: #e2e8f0;
    font-family: 'Courier New', monospace;
    transition: all 0.3s ease;
}

.share-url-input:focus {
    outline: none;
    border-color: #60a5fa;
    box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.2);
}

.input-icon {
    position: absolute;
    right: 0.75rem;
    color: #94a3b8;
    pointer-events: none;
}

.copy-btn {
    padding: 0.875rem 1.5rem;
    background: linear-gradient(135deg, #0ea5e9 0%, #0284c7 100%);
    color: white;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    font-size: 0.9rem;
    font-weight: 600;
    transition: all 0.3s ease;
    white-space: nowrap;
    box-shadow: 0 4px 12px rgba(14, 165, 233, 0.3);
    position: relative;
    overflow: hidden;
}

.copy-btn::before {
    content: '';
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
    transition: left 0.5s ease;
}

.copy-btn:hover::before {
    left: 100%;
}

.copy-btn:hover {
    background: linear-gradient(135deg, #0284c7 0%, #0369a1 100%);
    transform: translateY(-1px);
    box-shadow: 0 6px 16px rgba(14, 165, 233, 0.4);
}

.copy-btn:active {
    transform: translateY(0);
}

.share-info {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
}

.info-item {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    color: #94a3b8;
    font-size: 0.85rem;
    padding: 0.4rem;
    background: rgba(15, 23, 42, 0.6);
    border-radius: 6px;
    border-left: 3px solid #10b981;
}

/* Transition animations */
.slide-down-enter-active,
.slide-down-leave-active {
    transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.slide-down-enter-from {
    opacity: 0;
    transform: translateY(-20px) scale(0.95);
}

.slide-down-leave-to {
    opacity: 0;
    transform: translateY(-10px) scale(0.98);
}

/* Responsive design */
@media (max-width: 640px) {
    .share-link-area {
        padding: 1.5rem;
    }

    .share-url-container {
        flex-direction: column;
        gap: 1rem;
    }

    .copy-btn {
        padding: 1rem;
        font-size: 1rem;
    }

    .share-btn {
        padding: 1rem 1.5rem;
        font-size: 0.95rem;
    }

    .info-item {
        font-size: 0.85rem;
    }
}
</style>