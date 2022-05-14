<script setup lang="ts">
import 'vue-good-table-next/dist/vue-good-table-next.css'

import { VueGoodTable } from 'vue-good-table-next'
import { ref } from 'vue'
import type { ReceivedStatusUpdate } from 'webxdc'
import { stump_data } from '~/stump'
import { toggleDark } from '~/composables/dark'

const columns = [
  {
    label: 'Timestamp',
    field: 'ts',
    type: 'date',
    dateInputFormat: 'T',
    dateOutputFormat: 'E H:m:ss',
    sortable: false,
  },
  {
    label: 'Type',
    field: 'event_type',
    type: 'number',
    sortable: false,
  },
  {
    label: 'Data 1',
    field: 'data1',
    type: 'number',
    sortable: false,
  },
  {
    label: 'Data 2',
    field: 'data2',
    type: 'number',
    sortable: false,
  },
]

const data = ref([] as usedData[])

//https://borisflesch.github.io/vue-good-table-next/guide/configuration/pagination-options.html
const pagination_options = {
  enabled: true,
  perPage: 20,
  perPageDropdown: [20, 40, 60, 150],
}

// restore old logs from cache
const cached_data = localStorage.getItem('log_data')
if (cached_data) {
  data.value = JSON.parse(cached_data)
}

const options = {
  enabled: true,
  skipDiacritics: true,
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
    for (let i = 0; i < 5; i++) {
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
  div
    VueGoodTable(
      :columns="columns"
      :rows="data"
      :search-options = "options"
      :pagination-options = "pagination_options"
      compactMode
    )
</template>

<style lang="sass">

table.vgt-table td
  padding: 2px !important

thead
  display: none

.vgt-right-align
  text-align: left !important

.vgt-global-search__input
  padding-left: 5px !important

.magnifying-glass
  display: none !important
</style>
