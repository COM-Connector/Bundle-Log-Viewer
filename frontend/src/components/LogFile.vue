<script setup lang="ts">
import ButtonGroup from '@/components/ButtonGroup.vue';
import type LogFile from '@/models/LogFile';
import ParameterBag from '@/models/ParameterBag';
import bus from '@/services/EventBus';
import {useHostsStore} from '@/stores/hosts';
import {useSearchStore} from '@/stores/search';
import axios from 'axios';
import {ref} from 'vue';
import {useRouter} from 'vue-router';

defineProps<{
    file: LogFile
}>();

const toggleRef   = ref();
const router      = useRouter();
const searchStore = useSearchStore();
const hostsStore  = useHostsStore();

const baseUri    = axios.defaults.baseURL;
const deleteFile = (identifier: string) => {
    const params = new ParameterBag().set('host', hostsStore.selected, 'localhost').all();
    axios.delete('/api/file/' + encodeURI(identifier), {params: params})
        .then(() => {
            searchStore.removeFile(identifier);
            if (searchStore.files.length === 0) {
                router.push({name: 'home'});
            }
            bus.emit('file-deleted', identifier);
        });
}

const navigate = (identifier: string, multiSelect: boolean) => {
    if (multiSelect) {
        searchStore.toggleFile(identifier);
    } else {
        searchStore.setFile(identifier);
    }
    if (searchStore.files.length === 0) {
        router.push({name: 'home'});
        return;
    }
    router.push('/log?' + searchStore.toQueryString());
}
</script>

<template>
    <!-- LogFile -->
    <div class="list-group-item list-group-item-action d-flex align-items-center p-0"
         :class="{'bg-body-secondary': searchStore.files.includes(file.identifier)}">
        <a @click="(event) => {event.preventDefault(); navigate(file.identifier, event.ctrlKey || event.metaKey)}"
           href="javascript:"
           class="slv-file-link d-flex flex-grow-1 px-2 py-1 text-body text-decoration-none"
           :title="file.name">
            <span class="flex-grow-1 text-truncate">{{ file.name }}</span>
            <span class="text-body-secondary small text-nowrap ms-2">{{ file.size_formatted }}</span>
        </a>
        <button-group ref="toggleRef" alignment="right" :hide-on-selected="true">
            <template v-slot:btn_left>
                <button type="button" class="btn btn-link btn-sm text-body" aria-label="File menu" @click="toggleRef.toggle">
                    <i class="bi bi-three-dots-vertical"></i>
                </button>
            </template>
            <template v-slot:dropdown>
                <li>
                    <a class="dropdown-item" href="javascript:" @click="navigate(file.identifier, true)">
                        <i class="bi bi-check2-circle me-3"></i>{{ searchStore.files.includes(file.identifier) ? 'Deselect' : 'Select' }}
                        <code>(ctrl+click)</code>
                    </a>
                </li>
                <li v-if="file.can_download">
                    <a class="dropdown-item"
                       :href="baseUri + 'api/file/' + encodeURI(file.identifier) + '?' + new ParameterBag().set('host', hostsStore.selected, 'localhost').toString()">
                        <i class="bi bi-cloud-download me-3"></i>Download
                    </a>
                </li>
                <li v-if="file.can_delete">
                    <a class="dropdown-item" href="javascript:" @click="deleteFile(file.identifier)">
                        <i class="bi bi-trash3 me-3"></i>Delete
                    </a>
                </li>
            </template>
        </button-group>
    </div>
</template>

<style scoped>
.slv-file-link {
    min-width: 0;
}
</style>
