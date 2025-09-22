# v-jp-datepicker

This is a simple datepicker component developer in Vue 3 and Tailwindcss 4

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## How to use this component?

### Install component

```sh
npm npm i v-jp-datepicker
```

### Basic configuration

```javascript
<template>
  <div class="m-6">
    <Datapicker v-model="selectedDate" label="Calendario" />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { Datapicker } from 'v-jp-datepicker'


// I get the selected date
const selectedDate = ref<string | null>(null)
</script>
```

### Enable date range in calendar

```javascript
<template>
  <div class="m-6">
    <Datapicker v-model="selectedDate" label="Calendario" :min="min" :max="max" />
  </div>
</template>

<script setup lang="ts">
import { Datapicker } from 'v-jp-datepicker'
import { ref } from 'vue';


// I get the selected date
const selectedDate = ref<string | null>(null)

// first enabled date on the calendar
const min = ref('2025-09-10')

// last enabled date of the calendar
const max = ref('2025-10-15')
</script>
```

### Add days to the minimum enabled date

```javascript
<template>
  <div class="m-6">
    <Datapicker v-model="selectedDate" label="Calendario" :min="min" :min-add-days="maxAddDays" />
  </div>
</template>

<script setup lang="ts">
import { Datapicker } from 'v-jp-datepicker'
import { ref } from 'vue';


// I get the selected date
const selectedDate = ref<string | null>(null)

// first enabled date on the calendar
const min = ref('2025-09-10')

// Add days to the minimum enabled date
const maxAddDays = ref(1)
</script>
```

### Disable weekends

```javascript
<template>
  <div class="m-6">
    <Datapicker v-model="selectedDate" label="Calendario" :weekend="weekend" />
  </div>
</template>

<script setup lang="ts">
import { Datapicker } from 'v-jp-datepicker'
import { ref } from 'vue';


// I get the selected date
const selectedDate = ref<string | null>(null)

// Disable weekends
const weekend = ref(true)
</script>
```

### Disable days in the calendar

```javascript
<template>
  <div class="m-6">
    <Datapicker v-model="selectedDate" label="Calendario" :holiday="holidais" />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { Datapicker } from 'v-jp-datepicker'


// I get the selected date
const selectedDate = ref<string | null>(null)

// disable days in the calendar
const holidais = ref(['2025-09-05', '2025-09-10', '2025-09-15'])

</script>
```

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Type-Check, Compile and Minify for Production

```sh
npm run build
```

### Lint with [ESLint](https://eslint.org/)

```sh
npm run lint
```
