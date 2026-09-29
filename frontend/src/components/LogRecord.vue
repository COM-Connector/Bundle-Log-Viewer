<script setup lang="ts">
import JsonData from '@/components/json/JsonData.vue';
import {nl2br, escapeHtml} from '@/services/Strings.ts';
import type LogRecord from '@/models/LogRecord';
import {isEmptyJson, prettyFormatJson} from '@/services/JsonFormatter';
import {ref} from 'vue';

const expanded = ref(false);
const styled   = ref(true);
const copied   = ref(false);
const props    = defineProps<{ logRecord: LogRecord }>()
const emit = defineEmits(['search']);

const levelClasses: Record<string, string> = {
    debug: 'bg-secondary-subtle text-secondary-emphasis',
    info: 'bg-info-subtle text-info-emphasis',
    notice: 'bg-primary-subtle text-primary-emphasis',
    warning: 'text-bg-warning',
    error: 'text-bg-danger',
    critical: 'slv-level-severe',
    alert: 'slv-level-severe',
    emergency: 'slv-level-severe',
};

const levelClass = (level: string): string => levelClasses[level.toLowerCase()] ?? levelClasses.info;

function click(value: string) {
    emit('search', value);
}

function copy() {
    const blocks = [props.logRecord.context, props.logRecord.extra].filter(data => !isEmptyJson(data)).map(prettyFormatJson);
    navigator.clipboard.writeText(blocks.join('\n\n')).then(() => {
        copied.value = true;
        setTimeout(() => copied.value = false, 1500);
    });
}
</script>

<template>
    <div class="slv-log-record list-group-item p-0" :class="{'opacity-50': logRecord.context_line && !expanded}" :aria-expanded="expanded">
        <div class="slv-list-link list-group-item-action px-3 py-2"
             :class="{ 'text-nowrap': !expanded, 'overflow-hidden': !expanded }"
             @click="expanded = !expanded">
            <i class="slv-indicator bi bi-chevron-right me-1"></i>
            <span class="slv-time pe-2 text-body-secondary">{{ logRecord.datetime }}</span>
            <span class="badge bg-secondary-subtle text-secondary-emphasis me-1" v-if="logRecord.channel.length > 0">{{ logRecord.channel }}</span>
            <span :class="['badge me-2', levelClass(logRecord.level_name)]">{{ logRecord.level_name }}</span>

            <!-- log message -->
            <span v-if="!expanded" v-text="logRecord.text.substring(0, 500)"></span>
            <span v-if="expanded" v-html="nl2br(escapeHtml(logRecord.text))"></span>
        </div>
        <div class="px-3 pb-2" v-if="expanded">
            <div class="bg-body-tertiary border rounded p-2 position-relative">
                <div class="slv-btn-actions">
                <button class="btn btn-sm btn-outline-secondary" type="button" title="Copy" aria-label="Copy" @click="copy">
                    <i class="bi" :class="copied ? 'bi-check2' : 'bi-clipboard'"></i>
                </button>
                <button class="btn btn-sm btn-outline-secondary ms-1" @click="styled = !styled">{{ styled ? 'raw' : 'styled' }}</button>
            </div>
                <div v-if="!isEmptyJson(logRecord.context)">
                    <div class="fw-bold">Context:</div>
                    <json-data v-if="styled" path="context:" :data=logRecord.context @click="click"></json-data>
                    <div v-else>
                        <pre class="ms-0"><code>{{ prettyFormatJson(logRecord.context) }}</code></pre>
                    </div>
                </div>
                <div v-if="!isEmptyJson(logRecord.extra)">
                    <div class="fw-bold">Extra:</div>
                    <json-data v-if="styled" path="extra:" :data=logRecord.extra @click="click"></json-data>
                    <div v-else>
                        <pre class="ms-0"><code>{{ prettyFormatJson(logRecord.extra) }}</code></pre>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.slv-log-record:nth-child(even) {
    background-color: var(--bs-tertiary-bg);
}

.slv-list-link {
    cursor: pointer;
}

.slv-time {
    font-variant-numeric: tabular-nums;
}

.slv-btn-actions {
    position: absolute;
    top: 5px;
    right: 5px;
}

.slv-level-severe {
    background-color: var(--bs-danger-text-emphasis);
    color: var(--bs-body-bg);
}
</style>
