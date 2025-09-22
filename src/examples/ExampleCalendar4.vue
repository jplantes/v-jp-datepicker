<template>
  <div class="m-6">
    <p class="mb-4">Fecha seleccionada: {{ selectedDate }}</p>
    <Datapicker v-model="selectedDate" :weekend="weekend" label="Calendario" helper="Deshabilito los fines de semana" />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { Datapicker } from '@/lib'


// I get the selected date
const selectedDate = ref<string | null>(null)

// Disable weekends
const weekend = ref(true)
</script>