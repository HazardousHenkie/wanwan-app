<template>
    <div class="cards">
        <CardItem
            v-for="(card, index) in cards"
            :key="card"
            :card="card"
            :is-current="index === 0"
            @card-accepted="$emit('cardAccepted')"
            @card-rejected="$emit('cardRejected')"
            @hide-card="$emit('hideCard')"
        />
    </div>
</template>

<script setup lang="ts">
defineProps({
    cards: {
        type: Array,
        required: true,
    },
})
defineEmits(['cardAccepted', 'cardRejected', 'hideCard'])
</script>

<style scoped>
.cards {
    position: relative;
    display: flex;
    margin: 50px;
    width: 300px;
    margin-top: -300px;
}
</style>
