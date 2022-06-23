<script setup lang="ts">
import { ref } from 'vue'
import type { ReceivedStatusUpdate } from 'webxdc'
import { stump_data } from '~/stump'
import { toggleDark } from '~/composables/dark'
import { VirtualList } from 'vue3-virtual-list';

const data = ref([] as LogData[])

// restore old logs from cache
const cached_data = localStorage.getItem('log_data')
if (cached_data) {
  data.value = JSON.parse(cached_data)
}

interface LogData {
  data1: any
  data2: any
  event_type: any
  ts: Number
}

function receiveUpdate(log_data: ReceivedStatusUpdate<LogData>) {
  data.value.push(log_data.payload)
}

const reversed = computed(() => data.value.slice().reverse())

window.addEventListener('unload', () => {
  console.log('saving data to local storage')
})

const height = ref(0)
const hi = ref( null as unknown as HTMLElement)

onMounted(() => {
  window.webxdc.setUpdateListener(receiveUpdate, 0)

  // add stump data in dev
  if (import.meta.env.DEV) {
    for (let i = 0; i < 1000; i++) {
      stump_data.forEach((row) => {
        data.value.push(row.payload)
      })
    }
  }

  height.value = window.innerHeight - hi.value.scrollHeight
})

</script>

<template lang="pug">
div.h-screen.overflow-hidden
  div.flex.justify-between(ref="hi")
    h1.text-2xl.leading-none.mb-1 Current device logs
    button(@click="() => toggleDark()" key="2")
      div(i="carbon-sun dark:carbon-moon")
  
  table.items-wrapper(:style="{height: height + 'px'}")
    VirtualList(:data="reversed")
      template(v-slot="{item}")
        tr 
          td {{item.ts}}
          td {{item.event_type}}
          td {{item.data1}}
          td {{item.data2}}

    
</template>

<style lang="sass">

.dark body 
  background: var(--dark-bg) !important

.items-wrapper
  width: 100%

.item-container
  height: 40px

th, td
  padding: 2px

tr:nth-child(even) 
  background-color: #f2f2f2

.root
  max-height: 100vh
</style>
