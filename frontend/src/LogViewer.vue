<script setup lang="ts">
import FileTree from '@/components/FileTree.vue'
import {useRoute} from 'vue-router';

const route = useRoute();
</script>

<template>
    <header class="slv-header bg-body border rounded d-flex align-items-center px-3 py-2">
        <i class="bi bi-substack me-2"></i>
        <span class="fw-semibold text-nowrap">Log Viewer</span>
        <!-- target for view specific actions, e.g. the search form -->
        <div id="slv-header-actions" class="d-flex flex-grow-1 ms-4"></div>
    </header>

    <div class="h-100 overflow-hidden">
        <FileTree/>
    </div>
    <RouterView :key="route.fullPath"></RouterView>
</template>

<style scoped>
.slv-header {
    grid-column: 1 / -1;
}
</style>
