<script setup>
import { computed, ref } from 'vue';
import { Head } from '@inertiajs/vue3';
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';

defineProps({
    game: Object,
});

const boardState = ref([0, 0, 0, 0, 0, 0, 0, 0, 0])

const xTurn = computed(() => boardState.value.reduce((carry, value) => carry + value, 0) === 0)

const winningLines = [
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

    const winningLine = winningLines
        .map((lines) => lines.reduce((carry, index) => carry + boardState.value[index], 0))
        .find((sum) => Math.abs(sum) === 3)

    if (winningLine === -3) {
        return alert('X has won!')
    }

    if (winningLine === 3) {
        return alert('O has won!')
    }

    if (! boardState.value.includes(0)) {
        alert('Stalemate!')
    }
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
    </AuthenticatedLayout>
</template>
