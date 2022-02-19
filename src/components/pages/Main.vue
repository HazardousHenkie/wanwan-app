<template>
    <div
        class="flex flex-col items-center justify-center h-screen"
        style="margin-top: -200px"
    >
        <div v-if="rightChoice === 'true'">
            <CheckCircleIcon class="icon green" :size="400" />
        </div>
        <div v-else-if="rightChoice === 'false'">
            <CancelIcon class="icon red" color="red" :size="400" />
        </div>
        <h1 style="font-size: 2rem">Waar is het hondje?</h1>
        <h2>Score: {{ score }}</h2>

        <div class="flex">
            <CardsStack :cards="leftDeck" @hide-card="removeCardFromDecks" />

            <CardsStack :cards="rightDeck" @hide-card="removeCardFromDecks" />
        </div>
    </div>
</template>

<script setup lang="ts">
import axios from 'axios'
import { ref, watch } from 'vue'
import CancelIcon from 'vue-material-design-icons/Cancel.vue'
import CheckCircleIcon from 'vue-material-design-icons/CheckCircle.vue'

// TODO: abstract this compoennt en abstract fetching

const dogs = ref<string[]>([])
const cats = ref<string[]>([])
const score = ref(0)
const rightChoice = ref('')
const leftDeck = ref<{ dog: boolean; url: string }[]>([])
const rightDeck = ref<{ dog: boolean; url: string }[]>([])

// TODO: abstract this one
const removeCardFromDecks = (selectedCard: { dog: boolean; url: string }) => {
    if (selectedCard.dog) {
        rightChoice.value = 'true'
        score.value++
    } else {
        rightChoice.value = 'false'
        score.value--
    }

    leftDeck.value.shift()
    rightDeck.value.shift()
}

watch(rightChoice, () => {
    if (rightChoice.value !== '') {
        setTimeout(() => {
            rightChoice.value = ''
        }, 1000)
    }
})

watch(leftDeck.value, () => {
    if (leftDeck.value.length === 0) {
        score.value = 0
        fillDecks()
    }
})

const fetchDogImage = async () => {
    try {
        const response = await axios.get(
            'https://dog.ceo/api/breeds/image/random'
        )

        dogs.value.push(response.data.message)
    } catch (error) {
        console.error(error)
    }
}

const fetchCatImage = async () => {
    try {
        const response = await axios.get('https://cataas.com/cat?json=true')

        cats.value.push(`https://cataas.com/${response.data.url}`)
    } catch (error) {
        console.error(error)
    }
}

const fillDecks = () => {
    let promiseList = []

    for (let i = 0; i < 5; i++) {
        promiseList.push(fetchDogImage())
        promiseList.push(fetchCatImage())
    }

    Promise.all(promiseList).then(() => {
        dogs.value.map((dog) => {
            if (Math.random() < 0.5) {
                leftDeck.value.push({ dog: true, url: dog })
                rightDeck.value.push({ dog: false, url: cats.value[0] })
            } else {
                leftDeck.value.push({ dog: false, url: cats.value[0] })
                rightDeck.value.push({ dog: true, url: dog })
            }
            cats.value.shift()
        })
    })
}

fillDecks()
</script>

<style scoped>
.icon {
    position: absolute;
    z-index: 500;
    left: 50%;
    transform: translate(-50%, 0);
}

.icon.red {
    color: red;
}
.icon.green {
    color: green;
}
</style>
