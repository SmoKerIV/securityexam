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
            <h1 class="company-name">Berb & Husseh Security+</h1>
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
    backdrop-filter: blur(10px);
    padding: 20px 0;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
    border-bottom: 3px solid #667eea;
    position: sticky;
    top: 0;
    z-index: 100;
}

.header-content {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 20px;
}

.company-name {
    font-size: 2.5rem;
    font-weight: bold;
    background: linear-gradient(45deg, #667eea, #764ba2);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
    margin: 0;
}

.day-counter {
    display: flex;
    flex-direction: column;
    align-items: center;
    background: linear-gradient(45deg, #667eea, #764ba2);
    color: white;
    padding: 15px 25px;
    border-radius: 15px;
    box-shadow: 0 5px 15px rgba(102, 126, 234, 0.3);
}

.day-counter .label {
    font-size: 0.9rem;
    opacity: 0.9;
    margin-bottom: 5px;
}

.day-counter .count {
    font-size: 2rem;
    font-weight: bold;
}

@media (max-width: 768px) {
    .company-name {
        font-size: 1.8rem;
    }

    .header-content {
        justify-content: center;
        text-align: center;
    }

    .day-counter {
        margin-top: 10px;
    }
}
</style>