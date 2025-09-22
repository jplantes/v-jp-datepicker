<template>
  <div class="m-6">
    <p class="mb-4">Fecha seleccionada: {{ selectedDate }}</p>
    <Datapicker v-model="selectedDate" :min="min" :min-add-days="maxAddDays" label="Calendario"
      helper="Simo 'n' días a la fecha mínima seleccionable" />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { Datapicker } from '@/lib'


// I get the selected date
const selectedDate = ref<string | null>(null)

// first enabled date on the calendar
const min = ref('2025-09-10')

// Add days to the minimum enabled date
const maxAddDays = ref(1)
</script>