<template>
  <div class="m-6">
    <p class="mb-4">Fecha seleccionada: {{ selectedDate }}</p>
    <Datapicker v-model="selectedDate" :holiday="holidais" label="Calendario"
      helper="Deshabilito días del calendarios" />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { Datapicker } from '@/lib'


// I get the selected date
const selectedDate = ref<string | null>(null)

const holidais = ref(['2025-09-05', '2025-09-10', '2025-09-15'])
</script>