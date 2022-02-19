<template>
    <div
        v-if="isShowing"
        ref="interactElement"
        :class="{
            isAnimating: isInteractAnimating,
            isCurrent: isCurrent,
        }"
        class="card"
        :style="{ transform: transformString }"
        @click="handleOnClick"
    >
        <!-- should be a slot wtih binding -->
        <img alt="犬" :src="card.url" class="image" />
    </div>
</template>

<script setup lang="ts">
//TODO: make into composable
// TODO: interact is fine?
import interact from 'interactjs'
import Interact from '@interactjs/types'
import { ref, computed, onMounted, onBeforeUnmount, PropType } from 'vue'
import { CARD_STATES } from '../../enums'

interface Position {
    x?: number
    rotation?: number
}

const interactMaxRotation = 15
// TODO: Check this
const interactOutOfSightXCoordinate = 500
const interactXThreshold = 100

const props = defineProps({
    card: {
        type: Object as PropType<{ dog: boolean; url: string }>,
        required: true,
    },
    isCurrent: {
        type: Boolean,
        required: true,
    },
})

const emit = defineEmits([
    CARD_STATES.ACCEPT_CARD,
    CARD_STATES.REJECT_CARD,
    CARD_STATES.HIDE_CARD,
])

const interactElement = ref()
const isShowing = ref(true)
const isInteractAnimating = ref(true)
const isInteractDragged = ref(false)
const isInteractMoving = ref(false)
const interactPosition = ref({ x: 0, rotation: 0 })

// TODO: Do we need this/ if not animating or is dragging change styles
const transformString = computed(() => {
    if (!isInteractAnimating.value || isInteractDragged.value) {
        const { x, rotation } = interactPosition.value

        return `translate3D(${x}px, 0px, 0) rotate(${rotation}deg)`
    }

    return ''
})

onMounted(() => {
    interact(interactElement.value).draggable({
        onstart: () => {
            isInteractAnimating.value = false
            isInteractMoving.value = true
        },

        onmove: (event: Interact.InteractEvent) => {
            const x = interactPosition.value.x + event.dx

            let rotation = interactMaxRotation * (x / interactXThreshold)

            if (rotation > interactMaxRotation) rotation = interactMaxRotation
            else if (rotation < -interactMaxRotation)
                rotation = -interactMaxRotation

            interactSetPosition({ x, rotation })
        },

        onend: () => {
            const { x } = interactPosition.value
            isInteractAnimating.value = true

            if (x > interactXThreshold) {
                playCard(CARD_STATES.ACCEPT_CARD)
            } else if (x < -interactXThreshold) {
                playCard(CARD_STATES.REJECT_CARD)
            } else resetCardPosition()
        },
    })
})

onBeforeUnmount(() => {
    interact(interactElement.value).unset()
})

const hideCard = () => {
    setTimeout(() => {
        isShowing.value = false
        emit(CARD_STATES.HIDE_CARD, props.card)
    }, 300)
}

const playCard = (interaction: CARD_STATES) => {
    interactUnsetElement()

    switch (interaction) {
        case CARD_STATES.ACCEPT_CARD:
            interactSetPosition({
                x: interactOutOfSightXCoordinate,
                rotation: interactMaxRotation,
            })
            emit(CARD_STATES.ACCEPT_CARD)
            break
        case CARD_STATES.REJECT_CARD:
            interactSetPosition({
                x: -interactOutOfSightXCoordinate,
                rotation: -interactMaxRotation,
            })
            emit(CARD_STATES.REJECT_CARD)
            break
    }

    hideCard()
}

const interactSetPosition = (coordinates: Position) => {
    const { x = 0, rotation = 0 } = coordinates
    interactPosition.value = { x, rotation }
}

const interactUnsetElement = () => {
    interact(interactElement.value).unset()
    isInteractDragged.value = true
}

const resetCardPosition = () => {
    interactSetPosition({ x: 0, rotation: 0 })
}

const handleOnClick = () => {
    if (!isInteractMoving.value) {
        playCard(CARD_STATES.ACCEPT_CARD)
    } else {
        isInteractMoving.value = false
    }
}
</script>

<style scoped lang="scss">
@import '../../styles/index.scss';

$cardsTotal: 3;
$cardsWidth: 300px;
$cardsPositionOffset: 55vh * 0.06;
$cardsScaleOffset: 0.08;
$defaultTranslation: $cardsPositionOffset * $cardsTotal;
$defaultScale: 1 - ($cardsScaleOffset * $cardsTotal);
$fs-card-title: 1.125em;

.card {
    @include card();
    @include absolute(0);
    @include sizing(100% 80vw);
    @include flex-center();

    @include after() {
        @include sizing(21px 3px);
        @include absolute(right 0 bottom 11px left 0);

        margin: auto;
        border-radius: 100px;
        background: rgba($c-black, 0.3);
    }

    display: flex;
    max-height: 450px;
    margin: auto;
    font-size: $fs-h2;
    font-weight: $fw-bold;
    color: $c-white;
    background-image: linear-gradient(
        -180deg,
        $primary-gradient-start 2%,
        $primary-gradient-end 100%
    );

    opacity: 0;
    transform: translateY($defaultTranslation) scale($defaultScale);
    transform-origin: 50%, 100%;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    user-select: none;
    pointer-events: none;
    will-change: transform, opacity;

    height: 100vw;

    .image {
        max-height: 100%;
    }

    &.isCurrent {
        pointer-events: auto;
    }

    &.isAnimating {
        transition: transform 0.7s $ease-out-back;
    }
}

.cardTitle {
    margin: 0 0 15px;
    font-size: $fs-card-title;
}

@for $i from 1 through $cardsTotal {
    $index: $i - 1;
    $translation: $cardsPositionOffset * $index;
    $scale: 1 - ($cardsScaleOffset * $index);

    .card:nth-child(#{$i}) {
        z-index: $cardsTotal - $index;
        opacity: 1;
        transform: translateY($translation) scale($scale);

        @if $i == 3 {
            color: $c-red-25;
            background-color: $c-red-25;
        } @else if $i == 2 {
            color: $c-red-50;
            background-color: $c-red-50;
        }

        @if $i != 1 {
            background-image: none;

            @include after() {
                @include sizing(0 0);
            }
        }
    }
}
</style>
