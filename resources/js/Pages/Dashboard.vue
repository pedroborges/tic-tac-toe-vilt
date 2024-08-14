<script setup>
import { router } from '@inertiajs/core';
import { Head, Link } from '@inertiajs/vue3';
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';

const props = defineProps([
    'games',
])

Echo.private('lobby')
    .listen('GameJoined', (event) => {
        router.reload({ only: ['games'], onSuccess: () => games.value = props.games.data })
    })
</script>

<template>
    <Head title="Dashboard" />

    <AuthenticatedLayout>
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">Dashboard</h2>
            <ul class="divide-y mt-6">
                <li v-for="game in games.data" :key="game.id" class="px-2 py-1.5 flex justify-between items-center">
                    {{ game.player_one.name }}
                    <Link :href="route('games.join', game)" method="post" as="button" class="inline-flex items-center px-4 py-2 bg-gray-800 border border-transparent rounded-md font-semibold text-xs text-white uppercase tracking-widest hover:bg-gray-700 focus:bg-gray-700 active:bg-gray-900 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 transition ease-in-out duration-150">Join game</Link>
                </li>
            </ul>
        </template>

        <div class="py-12">
            <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
                <div class="p-4 bg-white overflow-hidden shadow-sm sm:rounded-lg">
                    <Link :href="route('games.store')" method="post" as="button" class="inline-flex items-center px-4 py-2 bg-gray-800 border border-transparent rounded-md font-semibold text-xs text-white uppercase tracking-widest hover:bg-gray-700 focus:bg-gray-700 active:bg-gray-900 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 transition ease-in-out duration-150">Create game</Link>
                </div>
            </div>
        </div>
    </AuthenticatedLayout>
</template>
