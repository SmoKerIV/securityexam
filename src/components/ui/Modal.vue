<script setup lang="ts">
interface Props {
    show: boolean
    title: string
    maxWidth?: string
}

defineProps<Props>()
const emit = defineEmits<{
    close: []
}>()
</script>

<template>
    <div v-if="show" class="modal-overlay" @click="emit('close')">
        <div class="modal-content" :style="{ maxWidth }" @click.stop>
            <div class="modal-header">
                <h2>{{ title }}</h2>
                <button @click="emit('close')" class="close-btn">
                    <svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2"
                        stroke-linecap="round" stroke-linejoin="round">
                        <line x1="18" y1="6" x2="6" y2="18"></line>
                        <line x1="6" y1="6" x2="18" y2="18"></line>
                    </svg>
                </button>
            </div>
            <div class="modal-body">
                <slot />
            </div>
        </div>
    </div>
</template>

<style scoped>
.modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.8);
    backdrop-filter: blur(8px);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;
    animation: fadeIn 0.3s ease;
}

.modal-content {
    background: rgba(15, 23, 42, 0.95);
    backdrop-filter: blur(20px);
    border-radius: 20px;
    padding: 32px;
    width: 90%;
    max-width: 700px;
    max-height: 80vh;
    overflow-y: auto;
    position: relative;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
    border: 1px solid rgba(59, 130, 246, 0.2);
    animation: modalSlideIn 0.3s ease;
}

.modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 24px;
}

.modal-header h2 {
    color: #f1f5f9;
    font-size: 2rem;
    font-weight: 700;
    margin: 0;
}

.close-btn {
    background: rgba(148, 163, 184, 0.1);
    color: #94a3b8;
    border: none;
    width: 36px;
    height: 36px;
    border-radius: 12px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.2rem;
    font-weight: 400;
    transition: all 0.3s ease;
    padding: 0;
    flex-shrink: 0;
}

.close-btn:hover {
    background: rgba(239, 68, 68, 0.15);
    color: #ef4444;
    transform: scale(1.05);
}

.close-btn:active {
    transform: scale(0.95);
}

.close-btn svg {
    transition: all 0.2s ease;
}

.close-btn:hover svg {
    transform: rotate(90deg);
}

@keyframes fadeIn {
    from {
        opacity: 0;
    }

    to {
        opacity: 1;
    }
}

@keyframes modalSlideIn {
    from {
        transform: scale(0.9) translateY(-20px);
        opacity: 0;
    }

    to {
        transform: scale(1) translateY(0);
        opacity: 1;
    }
}
</style>
