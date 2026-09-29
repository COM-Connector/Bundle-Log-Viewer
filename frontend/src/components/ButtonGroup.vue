<script setup lang="ts">
import ClickOutside from '@/services/ClickOutside';
import {setRelativeTo} from '@/services/Positioner';
import {onMounted, onUpdated, ref, watch} from 'vue';

const active       = ref<boolean>(false);
const dropdownRef  = ref();
const clickOutside = new ClickOutside([], () => toggle(false));
const props        = defineProps<{ alignment: 'left' | 'right', hideOnSelected?: boolean }>();

const toggle = (forceActive: boolean | null = null): void => {
    active.value = forceActive ?? !active.value;
}

watch(active, () => setTimeout(() => clickOutside.enable(active.value), 1));

onMounted(() => {
    if (props.hideOnSelected !== true) {
        clickOutside.addElement(dropdownRef.value);
    }
})
onUpdated(() => {
    if (active.value === false) {
        return;
    }
    setRelativeTo(dropdownRef.value.parentElement, dropdownRef.value, props.alignment);
});
defineExpose({toggle});
</script>

<template>
    <div class="slv-btn-group dropdown">
        <slot name="btn_left"></slot>
        <ul class="dropdown-menu" :class="{'d-block': active}" ref="dropdownRef">
            <slot name="dropdown"></slot>
        </ul>
    </div>
</template>
