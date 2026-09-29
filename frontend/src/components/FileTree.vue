<script setup lang="ts">
import LogFolder from '@/components/LogFolder.vue';
import bus from '@/services/EventBus';
import {useFolderStore} from '@/stores/folders';
import {useHostsStore} from '@/stores/hosts';
import {watch} from 'vue';

const folderStore = useFolderStore();
const hostsStore  = useHostsStore();

watch(() => hostsStore.selected, () => folderStore.update());

bus.on('file-deleted', () => folderStore.update());
bus.on('folder-deleted', () => folderStore.update());
</script>

<template>
    <!-- FileTree -->
    <div class="card h-100 overflow-hidden">
        <div class="card-header border-bottom-0 d-flex align-items-center gap-2">
            <select class="form-select form-select-sm w-auto"
                    v-model="hostsStore.selected"
                    v-if="Object.keys(hostsStore.hosts).length > 0">
                <option v-for="(name, key) in hostsStore.hosts" :value="key" :key="key">{{ name }}</option>
            </select>
            <select class="form-select form-select-sm w-auto ms-auto" aria-label="Sort direction" v-model="folderStore.direction" v-on:change="folderStore.update">
                <option value="desc">Newest First</option>
                <option value="asc">Oldest First</option>
            </select>
        </div>

        <div class="slv-loadable overflow-auto" v-bind:class="{ 'slv-loading': folderStore.loading }">
            <log-folder :folder="folder" :expand="true" :key="index" v-for="(folder, index) in folderStore.folders"/>
        </div>
    </div>
</template>
