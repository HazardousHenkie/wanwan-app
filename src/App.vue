<template>
    <div class="flex items-center justify-center h-screen">
        <img alt="犬" :src="dogImage" @click="fetchDogImage" />
    </div>
</template>

<script setup lang="ts">
import axios from 'axios'
import { ref } from 'vue'

const dogImage = ref('')

const fetchDogImage = async () => {
    try {
        const response = await axios.get(
            'https://dog.ceo/api/breeds/image/random'
        )

        dogImage.value = response.data.message
    } catch (error) {
        console.error(error)
    }
}

fetchDogImage()
</script>
