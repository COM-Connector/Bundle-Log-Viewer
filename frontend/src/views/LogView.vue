<script setup lang="ts">
import LogRecord from '@/components/LogRecord.vue';
import PerformanceDetails from '@/components/PerformanceDetails.vue';
import SearchForm from '@/components/SearchForm.vue';
import ParameterBag from '@/models/ParameterBag';
import Repeater from '@/services/Repeater.ts';
import {useBrowserStore} from '@/stores/browser.ts';
import {useFolderStore} from '@/stores/folders';
import {useHostsStore} from '@/stores/hosts';
import {useLogRecordStore} from '@/stores/log_records';
import {useSearchStore} from '@/stores/search';
import {computed, onMounted, onUnmounted, ref} from 'vue';
import {type LocationQueryValueRaw, useRoute, useRouter} from 'vue-router';

const router         = useRouter();
const route          = useRoute();
const logRecordStore = useLogRecordStore();
const folderStore    = useFolderStore();
const hostsStore     = useHostsStore();
const searchStore    = useSearchStore();
const browserStore   = useBrowserStore();
const repeater       = new Repeater(5000, () => load(false));

const searchRef   = ref<InstanceType<typeof SearchForm>>()
const offset      = ref(0);
const selectedFiles = computed(() => folderStore.folders
        .flatMap(folder => folder.files)
        .filter(file => searchStore.files.includes(file.identifier)));
const badRequest  = ref(false);

const navigate = () => {
    const fileOffset = offset.value > 0 && logRecordStore.records.paginator?.direction !== searchStore.sort ? 0 : offset.value;
    const params     = new ParameterBag()
            .set('host', hostsStore.selected, 'localhost')
            .set('file', searchStore.files.join(','))
            .set('query', searchStore.query, '')
            .set('between', searchStore.between, '')
            .set('per_page', searchStore.perPage, '100')
            .set('sort', searchStore.sort, 'desc')
            .set('offset', fileOffset, 0);
    router.push({query: <Record<string, LocationQueryValueRaw>>params.all()});
}

const load = (loadingIndicator: boolean) => {
    badRequest.value = false;
    logRecordStore
            .fetch(
                    new ParameterBag()
                            .set('host', hostsStore.selected, 'localhost')
                            .set('file', searchStore.files.join(','))
                            .set('query', searchStore.query, '')
                            .set('between', searchStore.between, '')
                            .set('per_page', searchStore.perPage, '100')
                            .set('sort', searchStore.sort, 'desc')
                            .set('offset', offset.value, 0)
                            .set('time_zone', Intl.DateTimeFormat().resolvedOptions().timeZone),
                    loadingIndicator
            )
            .catch((error: Error) => {
                if (error.message === 'bad-request') {
                    badRequest.value = true;
                    return;
                }
                router.push({name: error.message});
            })
            .finally(() => {
                searchRef.value?.focus();
            });
}

onMounted(() => {
    repeater.start(browserStore.autorefresh);
    hostsStore.selected = String(route.query.host ?? 'localhost');
    searchStore.files   = String(route.query.file).split(',');
    searchStore.query   = String(route.query.query ?? '');
    searchStore.between = String(route.query.between ?? '');
    searchStore.perPage = String(route.query.per_page ?? '100');
    searchStore.sort    = String(route.query.sort ?? 'desc');
    offset.value        = parseInt(String(route.query.offset ?? '0'));
    load(true);
});

onUnmounted(() => {
    repeater.stop();
})

function onSearchRequest(value: string) {
    searchStore.query = value;
    navigate();
}
</script>

<template>
    <teleport to="#slv-header-actions" defer>
        <search-form class="flex-grow-1"
                     ref="searchRef"
                     :bad-request="badRequest"
                     v-model:query="searchStore.query"
                     v-model:between="searchStore.between"
                     v-model:sort="searchStore.sort"
                     v-model:perPage="searchStore.perPage"
                     @navigate="navigate"></search-form>
    </teleport>

    <div class="card h-100 overflow-hidden">
        <div class="card-header d-flex align-items-center">
            <div class="flex-grow-1 text-truncate">
                <span class="me-3" v-for="file in selectedFiles" :key="file.identifier">
                    <i class="bi bi-file-earmark-text me-1"></i>{{ file.name }}
                    <span class="text-body-secondary small ms-1">{{ file.size_formatted }}</span>
                </span>
            </div>
            <button class="btn btn-sm btn-outline-secondary ms-2"
                    type="button"
                    aria-label="Auto refresh every 5 seconds"
                    title="Auto refresh every 5 seconds"
                    @click="browserStore.autorefresh = !browserStore.autorefresh; repeater.start(browserStore.autorefresh)">
                <i class="bi" :class="{'bi-play-fill': !browserStore.autorefresh, 'bi-pause-fill': browserStore.autorefresh}"></i>
            </button>
            <button class="btn btn-sm btn-outline-secondary ms-1" type="button" aria-label="Refresh" title="Refresh" @click="load(true)">
                <i class="bi bi-arrow-clockwise"></i>
            </button>
        </div>

        <main class="flex-grow-1 overflow-auto d-none d-md-block slv-loadable" v-bind:class="{ 'slv-loading': logRecordStore.loading }">
            <div class="list-group list-group-flush">
                <log-record :logRecord="record"
                            v-for="(record, index) in logRecordStore.records.logs ?? []"
                            v-bind:key="index"
                            @search="onSearchRequest"></log-record>
            </div>
        </main>

        <footer class="card-footer d-flex align-items-center" v-show="!logRecordStore.loading">
            <button class="btn btn-sm btn-outline-secondary"
                    @click="offset = 0; navigate()"
                    v-bind:disabled="logRecordStore.records.paginator?.first !== true">
                First
            </button>
            <button class="ms-2 btn btn-sm btn-outline-secondary"
                    @click="offset = logRecordStore.records.paginator?.offset ?? 0; navigate()"
                    v-bind:disabled="logRecordStore.records.paginator?.more !== true">
                Next {{ searchStore.perPage }}
            </button>

            <div class="flex-grow-1"></div>

            <performance-details :performance="logRecordStore.records.performance"></performance-details>
        </footer>
    </div>
</template>
