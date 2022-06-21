<script setup lang="ts">
import 'vue-good-table-next/dist/vue-good-table-next.css'

import { ref } from 'vue'
import type { ReceivedStatusUpdate } from 'webxdc'
import { stump_data } from '~/stump'
import { toggleDark } from '~/composables/dark'
import { VirtualList } from 'vue3-virtual-list';
import tableRow from '~/components/tableRow.vue'

const data = ref([] as usedData[])

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


interface usedData {
  data1: any
  data2: any
  event_type: any
  ts: Number
}

function receiveUpdate(log_data: ReceivedStatusUpdate<LogData>) {
  // insert newest log at the beginning of the list
  data.value.splice(0, 0, log_data.payload)
  localStorage.setItem('lastSerial', log_data.serial.toString())
}

window.addEventListener('unload', () => {
  console.log('saving data to local storage')
  localStorage.setItem('log_data', JSON.stringify(data.value))
})


onMounted(() => {
  if (!localStorage.getItem('lastSerial'))
    localStorage.setItem('lastSerial', '0')

  window.webxdc.setUpdateListener(receiveUpdate, parseInt(localStorage.getItem('lastSerial')!))

  // add stump data in dev
  if (import.meta.env.DEV) {
    for (let i = 0; i < 1000; i++) {
      stump_data.forEach((row) => {
        data.value.push(row.payload)
      })
    }
  }
})
</script>

<template lang="pug">
div
  div.flex.justify-between
    h1.text-2xl.leading-none.mb-1 Current device logs
    button(@click="() => toggleDark()" key="2")
          div(i="carbon-sun dark:carbon-moon")
  
  table
    VirtualList( :data="data")
      template(v-slot="{item, index}")
        tr
          td {{item.ts}}
          td {{item.event_type}}
          td {{item.data1}}
          td {{item.data2}}

    
</template>

<style lang="sass">


.dark body 
  background: var(--dark-bg) !important

.dark .vgt-table tbody 
  background: var(--dark-bg) !important


.dark table.vgt-table td 
  color: white !important

.dark .vgt-global-search 
  background: none

.vgt-wrap__footer 
  padding: 0px !important

</style>
