<script setup>
import { computed, onUnmounted, ref } from 'vue';
import { router } from '@inertiajs/core';
import { Head } from '@inertiajs/vue3';
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import Modal from '@/Components/Modal.vue';
import PrimaryButton from '@/Components/PrimaryButton.vue';
import { useGameState, gameStates } from "@/Composables/useGameState.js";

const props = defineProps(['game']);
const boardState = ref([0, 0, 0, 0, 0, 0, 0, 0, 0])
const players = ref([])
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

Echo.join(`games.${props.game.id}`)
    .here((users) => players.value = users)
    .joining((user) => router.reload({
        onSuccess: () => players.value.push(user),
    }))
    .leaving((user) => players.value = players.value.filter(({ id }) => id !== user.id))

onUnmounted(() => Echo.leave(`games.${props.game.id}`))

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

        <ul class="max-w-sm mx-auto mt-6 space-y-2">
            <li class="flex items-center gap-2">
                <span class="p-1.5 font-bold rounded bg-gray-200">X</span>
                <span>{{ game.player_one.name }}</span>
                <span
                    class="size-2 rounded-full"
                    :class="{
                        'bg-red-500': !players.find(({ id }) => id === game.player_one_id),
                        'bg-green-500': players.find(({ id }) => id === game.player_one_id),
                    }"
                ></span>
            </li>
            <li v-if="game.player_two" class="flex items-center gap-2">
                <span class="p-1.5 font-bold rounded bg-gray-200">O</span>
                <span>{{ game.player_two.name }}</span>
                <span
                    class="size-2 rounded-full"
                    :class="{
                        'bg-red-500': !players.find(({ id }) => id === game.player_two_id),
                        'bg-green-500': players.find(({ id }) => id === game.player_two_id),
                    }"
                ></span>
            </li>
            <li v-else class="flex items-center gap-2">
                <span class="p-1.5 font-bold rounded bg-gray-200">O</span>
                <span>Waiting for your opponent</span>
                <span class="bg-yellow-500 size-2 rounded-full"></span>
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
