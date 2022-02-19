<template>
    <div class="flex items-center justify-center h-screen">
        <CardsStack :cards="visibleCards" @hide-card="removeCardFromDeck" />
    </div>
</template>

<script setup lang="ts">
import axios from 'axios'
import { ref } from 'vue'

const visibleCards = ref<string[]>([])

const removeCardFromDeck = () => {
    visibleCards.value.shift()
}

const fetchDogImage = async () => {
    try {
        const response = await axios.get(
            'https://dog.ceo/api/breeds/image/random'
        )

        visibleCards.value.push(response.data.message)
    } catch (error) {
        console.error(error)
    }
}

let promiseList = []

for (let i = 0; i < 5; i++) {
    promiseList.push(fetchDogImage())
}

Promise.all(promiseList)
</script>
