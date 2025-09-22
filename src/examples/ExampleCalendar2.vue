<template>
  <div class="m-6">
    <p class="mb-4">Fecha seleccionada: {{ selectedDate }}</p>
    <Datapicker v-model="selectedDate" :min="min" :max="max" label="Calendario"
      helper="Fecha mínima y máxima seleccionable" />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { Datapicker } from '@/lib'


// I get the selected date
const selectedDate = ref<string | null>(null)

// first enabled date on the calendar
const min = ref('2025-09-10')

// last enabled date of the calendar
const max = ref('2025-10-15')
</script>