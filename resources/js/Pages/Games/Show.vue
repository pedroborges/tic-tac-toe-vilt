<script setup>
import { computed, ref } from 'vue';
import { Head } from '@inertiajs/vue3';
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import Modal from '@/Components/Modal.vue';
import PrimaryButton from '@/Components/PrimaryButton.vue';
import { useGameState, gameStates } from "@/Composables/useGameState.js";

defineProps({
    game: Object,
});

const boardState = ref([0, 0, 0, 0, 0, 0, 0, 0, 0])
const gameState = useGameState();

const xTurn = computed(() => boardState.value.reduce((carry, value) => carry + value, 0) === 0)

const lines = [
    // rows
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    // columns
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    // diagonals
    [0, 4, 8],
    [2, 4, 6],
];

function fillSquare(index) {
    boardState.value[index] = xTurn.value ? -1 : 1

    checkForVictory()
}

function checkForVictory() {
    const winningLine = lines
        .map((line) => line.reduce((carry, index) => carry + boardState.value[index], 0))
        .find((sum) => Math.abs(sum) === 3)

    if (winningLine === -3) {
        gameState.change(gameStates.XWins)
        return
    }

    if (winningLine === 3) {
        gameState.change(gameStates.OWins)
        return
    }

    if (! boardState.value.includes(0)) {
        gameState.change(gameStates.Stalemate)
    }
}

function resetGame() {
    boardState.value = [0, 0, 0, 0, 0, 0, 0, 0, 0]
    gameState.change(gameStates.InProgress)
}
</script>

<template>
    <Head title="Game" />

    <AuthenticatedLayout>
        <ul class="grid grid-cols-3 gap-1.5 w-0 min-w-fit mx-auto mt-12">
            <li v-for="(square, index) in boardState" class="grid place-items-center size-32 bg-gray-300">
                <button v-if="square === 0" @click="fillSquare(index)" class="place-self-stretch bg-gray-200 hover:bg-gray-300 transition-colors"></button>
                <span v-else v-text="square === -1 ? 'X' : 'O'" class="text-4xl font-medium"></span>
            </li>
        </ul>
        <Modal @close="resetGame()" :show="gameState.hasEnded()">
            <div class="p-6">
                <div class="text-6xl font-bold text-center my-12 font-mono uppercase">
                    <span v-if="gameState.current() === gameStates.XWins" class="text-green-600">X has won!</span>
                    <span v-else-if="gameState.current() === gameStates.OWins" class="text-green-600">O has won!</span>
                    <span v-else class="text-orange-600">Stalemate!</span>
                </div>

                <div class="mt-6 flex justify-end">
                    <PrimaryButton @click="resetGame()">Play again</PrimaryButton>
                </div>
            </div>
        </Modal>
    </AuthenticatedLayout>
</template>
