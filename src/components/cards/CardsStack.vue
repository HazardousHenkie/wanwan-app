<template>
    <div class="cards">
        <CardItem
            v-for="(card, index) in cards"
            :key="card.url"
            :card="card"
            :is-current="index === 0"
            @card-accepted="$emit('cardAccepted')"
            @card-rejected="$emit('cardRejected')"
            @hide-card="(test: any) => $emit('hideCard', test)"
        />
    </div>
</template>

<script setup lang="ts">
import { PropType } from 'vue'

defineProps({
    cards: {
        type: Array as PropType<{ dog: boolean; url: string }[]>,
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
    width: 400px;
}
</style>
