<script setup lang="ts">
import { ref, onMounted } from 'vue'

const dayCount = ref(1)

const updateDayCounter = () => {
    const startDate = new Date('2025-06-03') // June 3, 2025
    const currentDate = new Date()
    const timeDifference = currentDate.getTime() - startDate.getTime()
    const daysDifference = Math.floor(timeDifference / (1000 * 60 * 60 * 24)) + 1

    dayCount.value = daysDifference
}

onMounted(() => {
    updateDayCounter()
    // Update the counter every day at midnight
    setInterval(updateDayCounter, 1000 * 60 * 60 * 24)
})
</script>

<template>
    <header class="header">
        <div class="header-content">
            <h1 class="company-name">Berb, Hola & Husseh Security+</h1>
            <div class="day-counter">
                <span class="label">Days Active</span>
                <span class="count">{{ dayCount }}</span>
            </div>
        </div>
    </header>
</template>

<style scoped>
.header {
    background: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(20px);
    padding: 24px 0;
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.12);
    border-bottom: 1px solid rgba(102, 126, 234, 0.2);
    position: sticky;
    top: 0;
    z-index: 100;
    border-radius: 0 0 24px 24px;
}

.header-content {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 32px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 24px;
}

.company-name {
    font-size: 2.8rem;
    font-weight: 800;
    background: linear-gradient(135deg, #667eea, #764ba2);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    margin: 0;
    font-family: 'Inter', 'Segoe UI', sans-serif;
    letter-spacing: -0.02em;
}

.day-counter {
    display: flex;
    flex-direction: column;
    align-items: center;
    background: linear-gradient(135deg, #667eea, #764ba2);
    color: white;
    padding: 20px 28px;
    border-radius: 20px;
    box-shadow: 0 8px 25px rgba(102, 126, 234, 0.4);
    transition: all 0.3s ease;
}

.day-counter:hover {
    transform: translateY(-2px);
    box-shadow: 0 12px 35px rgba(102, 126, 234, 0.6);
}

.day-counter .label {
    font-size: 0.95rem;
    opacity: 0.9;
    margin-bottom: 8px;
    font-weight: 500;
    letter-spacing: 0.5px;
    text-transform: uppercase;
}

.day-counter .count {
    font-size: 2.2rem;
    font-weight: 800;
    text-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

@media (max-width: 768px) {
    .company-name {
        font-size: 2.2rem;
    }

    .header-content {
        justify-content: center;
        text-align: center;
        padding: 0 20px;
    }

    .day-counter {
        margin-top: 16px;
    }
}
</style>