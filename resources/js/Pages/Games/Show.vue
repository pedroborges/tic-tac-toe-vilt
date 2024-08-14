<script setup>
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import { Head } from '@inertiajs/vue3';

defineProps({
    game: Object,
});
</script>

<template>
    <Head title="Game" />

    <AuthenticatedLayout>
        {{ $page.props }}
    </AuthenticatedLayout>
</template>
