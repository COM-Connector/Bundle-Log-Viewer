<script setup lang="ts">
import FileTree from '@/components/FileTree.vue'
import {useRoute} from 'vue-router';

const route = useRoute();
</script>

<template>
    <div class="slv-sidebar h-100 overflow-hidden">
        <header class="slv-header-height slv-header bg-body d-flex align-items-center px-2">
            <i class="bi bi-substack me-2"></i>
            <span>Log Viewer</span>
        </header>

        <FileTree/>
    </div>
    <RouterView :key="route.fullPath"></RouterView>
</template>

<style scoped>
.slv-sidebar {
    display: grid;
    grid-template-rows: auto 1fr;
}
</style>
