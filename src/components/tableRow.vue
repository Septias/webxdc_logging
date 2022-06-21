<template lang="pug">
tr
    td data.ts
    td data.event_type
    td data.data1
    td data.data2

</template>

<script lang="ts" setup>
import type { PropType } from 'vue';
import type { usedData } from '~/composables/types';


defineProps({
    data: {} as PropType<usedData>
})
</script>