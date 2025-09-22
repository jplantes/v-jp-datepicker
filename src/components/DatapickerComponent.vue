<template>
  <div class="relative inline-block" ref="root">
    <label v-if="label" class="block text-sm font-medium text-slate-700 mb-1">{{ label }}</label>

    <div class="flex items-center gap-3">
      <button type="button" :disabled="propsDisabled" class="w-56 text-left flex items-center justify-between gap-2 px-3 py-2 bg-white border border-slate-300 rounded-md shadow-sm transition-colors
               focus:outline-none focus:ring-2 focus:ring-indigo-500" :class="[
                propsDisabled ? 'opacity-50 cursor-not-allowed' : 'hover:border-slate-400',
              ]" @click="toggle" :aria-expanded="open" aria-haspopup="dialog">
        <span class="text-sm text-slate-900" v-if="displayValue">{{ displayValue }}</span>
        <span class="text-sm text-slate-400" v-else>Seleccionar fecha</span>
        <svg class="w-4 h-4 text-slate-400" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
          <path fill-rule="evenodd"
            d="M5.23 7.21a.75.75 0 011.06.02L10 10.94l3.71-3.71a.75.75 0 011.08 1.04l-4.25 4.25a.75.75 0 01-1.06 0L5.21 8.27a.75.75 0 01.02-1.06z"
            clip-rule="evenodd" />
        </svg>
      </button>

      <button v-if="value && !propsDisabled" type="button" @click="clear" class="inline-flex items-center justify-center h-9 px-3 rounded-md border border-slate-200 bg-slate-50 text-slate-700
               hover:bg-slate-100 transition-colors" title="Limpiar fecha">
        Limpiar
      </button>
    </div>

    <p v-if="helper" class="mt-2 text-xs text-slate-500">{{ helper }}</p>

    <!-- Popover (absolute to avoid layout shift) -->
    <div v-if="open"
      class="absolute left-0 top-full mt-2 w-64 bg-white border border-slate-200 rounded-md shadow-lg p-3 z-50">
      <div class="flex items-center justify-between mb-2">
        <button type="button" class="p-1 rounded hover:bg-slate-100" @click="prevMonth" aria-label="Mes anterior">
          <svg class="w-5 h-5 text-slate-600" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd"
              d="M12.03 15.03a.75.75 0 01-1.06 0L6 10.06l4.97-4.97a.75.75 0 111.06 1.06L8.12 10l3.91 3.97a.75.75 0 010 1.06z"
              clip-rule="evenodd" />
          </svg>
        </button>

        <div class="text-sm font-medium text-slate-700 capitalize">
          {{ monthLabel }}
        </div>

        <button type="button" class="p-1 rounded hover:bg-slate-100" @click="nextMonth" aria-label="Mes siguiente">
          <svg class="w-5 h-5 text-slate-600" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd"
              d="M7.97 4.97a.75.75 0 011.06 0L14 10l-4.97 5.03a.75.75 0 11-1.06-1.06L11.88 10 7.97 5.97a.75.75 0 010-1.06z"
              clip-rule="evenodd" />
          </svg>
        </button>
      </div>

      <div class="grid grid-cols-7 gap-1 text-xs text-slate-500 mb-2">
        <div class="text-center">Do</div>
        <div class="text-center">Lu</div>
        <div class="text-center">Ma</div>
        <div class="text-center">Mi</div>
        <div class="text-center">Ju</div>
        <div class="text-center">Vi</div>
        <div class="text-center">Sa</div>
      </div>

      <div class="grid grid-cols-7 gap-1 text-sm">
        <button v-for="cell in cells" :key="cell.key" type="button"
          class="h-9 flex items-center justify-center rounded-md focus:outline-none text-center"
          :class="cellClass(cell)" @click="selectCell(cell)" :disabled="cell.disabled || propsDisabled"
          :title="cellTitle(cell)" :aria-disabled="cell.disabled || propsDisabled">
          <span>{{ cell.date.getDate() }}</span>
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted, onBeforeUnmount } from 'vue'
import '../assets/output.css';

interface Props {
  modelValue?: string | null
  label?: string
  helper?: string
  min?: string
  max?: string
  disabled?: boolean
  minAddDays?: number // suma días a la fecha min
  holiday?: string[] // array de 'yyyy-mm-dd' que serán no seleccionables
  weekend?: boolean // si true, deshabilita sábados y domingos (por defecto false)
}
const props = defineProps<Props>()
const emit = defineEmits<{
  (e: 'update:modelValue', value: string | null): void
}>()

const root = ref<HTMLElement | null>(null)
const open = ref(false)

const value = ref<string | null>(props.modelValue ?? null)
watch(() => props.modelValue, (v) => { value.value = v ?? null })

function formatYMD(d: Date | null): string | null {
  if (!d) return null
  const y = d.getFullYear()
  const m = String(d.getMonth() + 1).padStart(2, '0')
  const day = String(d.getDate()).padStart(2, '0')
  return `${y}-${m}-${day}`
}
function parseYMD(s: string | null): Date | null {
  if (!s) return null
  const m = /^(\d{4})-(\d{2})-(\d{2})$/.exec(s)
  if (!m) return null
  const y = Number(m[1]), mo = Number(m[2]) - 1, d = Number(m[3])
  const dt = new Date(y, mo, d)
  dt.setHours(0, 0, 0, 0)
  return dt
}

const selectedDate = ref<Date | null>(parseYMD(value.value))
watch(selectedDate, (d) => {
  const v = formatYMD(d)
  value.value = v
  emit('update:modelValue', v)
})

watch(value, (v) => {
  if (v !== formatYMD(selectedDate.value)) {
    selectedDate.value = parseYMD(v)
  }
})

// viewDate controls the month shown in calendar
const viewDate = ref<Date>(selectedDate.value ? new Date(selectedDate.value) : new Date())
viewDate.value.setHours(0, 0, 0, 0)

watch(selectedDate, (d) => {
  if (d) viewDate.value = new Date(d)
}, { immediate: true })

const monthLabel = computed(() => {
  const opts: Intl.DateTimeFormatOptions = { month: 'long', year: 'numeric' }
  return viewDate.value.toLocaleDateString(undefined, opts)
})

function startOfMonth(dt: Date) {
  return new Date(dt.getFullYear(), dt.getMonth(), 1)
}
function addDays(dt: Date, days: number) {
  const n = new Date(dt)
  n.setDate(n.getDate() + days)
  n.setHours(0, 0, 0, 0)
  return n
}

interface Cell {
  key: string
  date: Date
  inMonth: boolean
  disabled: boolean
  isHoliday?: boolean
  isWeekend?: boolean
}

// computed set of holiday timestamps for fast lookup
const holidaysSet = computed<Set<number>>(() => {
  const arr = props.holiday ?? []
  const s = new Set<number>()
  for (const str of arr) {
    const d = parseYMD(str)
    if (d) s.add(d.getTime())
  }
  return s
})

const effectiveMin = computed(() => {
  const minD = parseYMD(props.min ?? null)
  if (!minD) return null
  const offset = Number(props.minAddDays ?? 0)
  return offset ? addDays(minD, offset) : minD
})

const effectiveMax = computed(() => {
  return parseYMD(props.max ?? null)
})

// weekend flag default false
const propsWeekend = computed(() => Boolean(props.weekend))

const cells = computed<Cell[]>(() => {
  const first = startOfMonth(viewDate.value)
  const startDay = first.getDay() // 0..6
  const start = addDays(first, -startDay)
  const arr: Cell[] = []
  for (let i = 0; i < 42; i++) {
    const d = addDays(start, i)
    const inMonth = d.getMonth() === viewDate.value.getMonth()
    let disabled = false
    const minD = effectiveMin.value
    if (minD && d.getTime() < minD.getTime()) disabled = true
    const maxD = effectiveMax.value
    if (maxD && d.getTime() > maxD.getTime()) disabled = true
    const isHoliday = holidaysSet.value.has(d.getTime())
    if (isHoliday) disabled = true
    const day = d.getDay() // 0 = Sunday, 6 = Saturday
    const isWeekend = day === 0 || day === 6
    if (propsWeekend.value && isWeekend) disabled = true
    arr.push({ key: `${d.getFullYear()}-${d.getMonth()}-${d.getDate()}`, date: d, inMonth, disabled, isHoliday, isWeekend })
  }
  return arr
})

function cellClass(cell: Cell) {
  const base: string[] = ['text-center', 'h-9', 'flex', 'items-center', 'justify-center', 'rounded-md']
  if (!cell.inMonth) base.push('text-slate-400')
  else base.push('text-slate-700')
  if (cell.disabled) base.push('cursor-not-allowed', 'opacity-50')
  else base.push('cursor-pointer', 'hover:bg-indigo-50')
  if (cell.isHoliday) {
    base.push('text-red-600') // visual cue for holiday
  }
  if (cell.isWeekend && propsWeekend.value) {
    base.push('text-slate-400') // visual cue for weekend when disabled
  }
  if (selectedDate.value && cell.date.getTime() === selectedDate.value.getTime()) {
    base.push('bg-indigo-600', 'text-white')
  } else {
    base.push('bg-transparent')
  }
  return base.join(' ')
}

function cellTitle(cell: Cell) {
  if (cell.isHoliday) return 'Feriado / No seleccionable'
  if (cell.isWeekend && propsWeekend.value) return 'Fin de semana / No seleccionable'
  return ''
}

const propsDisabled = computed(() => Boolean(props.disabled))

function selectCell(cell: Cell) {
  if (cell.disabled || propsDisabled.value) return
  selectedDate.value = new Date(cell.date)
  open.value = false
}

function prevMonth() {
  viewDate.value = new Date(viewDate.value.getFullYear(), viewDate.value.getMonth() - 1, 1)
}
function nextMonth() {
  viewDate.value = new Date(viewDate.value.getFullYear(), viewDate.value.getMonth() + 1, 1)
}

function toggle() {
  if (propsDisabled.value) return
  open.value = !open.value
}

function clear() {
  if (propsDisabled.value) return
  selectedDate.value = null
  open.value = false
}

const displayValue = computed(() => {
  if (!selectedDate.value) return null
  return formatYMD(selectedDate.value)
})

// --- Mejor manejo de cierre al hacer click fuera + Escape ---
function onDocumentPointerDown(e: PointerEvent) {
  if (!root.value) return
  // prefer composedPath para compatibilidad con shadow DOM
  const path = (e.composedPath && e.composedPath()) as EventTarget[] | undefined
  if (path && path.length) {
    if (!path.includes(root.value)) open.value = false
    return
  }
  const target = e.target
  if (target instanceof Node && !root.value.contains(target)) {
    open.value = false
  }
}

function onDocumentKeyDown(e: KeyboardEvent) {
  if (e.key === 'Escape') open.value = false
}

onMounted(() => {
  document.addEventListener('pointerdown', onDocumentPointerDown)
  document.addEventListener('keydown', onDocumentKeyDown)
})

onBeforeUnmount(() => {
  document.removeEventListener('pointerdown', onDocumentPointerDown)
  document.removeEventListener('keydown', onDocumentKeyDown)
})
</script>

<style lang="css">
/*! tailwindcss v4.1.13 | MIT License | https://tailwindcss.com */
@layer properties;
@layer theme, base, components, utilities;

@layer theme {

  :root,
  :host {
    --font-sans: ui-sans-serif, system-ui, sans-serif, "Apple Color Emoji",
      "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
    --font-mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono",
      "Courier New", monospace;
    --color-red-600: oklch(57.7% 0.245 27.325);
    --color-indigo-50: oklch(96.2% 0.018 272.314);
    --color-indigo-500: oklch(58.5% 0.233 277.117);
    --color-indigo-600: oklch(51.1% 0.262 276.966);
    --color-slate-50: oklch(98.4% 0.003 247.858);
    --color-slate-100: oklch(96.8% 0.007 247.896);
    --color-slate-200: oklch(92.9% 0.013 255.508);
    --color-slate-300: oklch(86.9% 0.022 252.894);
    --color-slate-400: oklch(70.4% 0.04 256.788);
    --color-slate-500: oklch(55.4% 0.046 257.417);
    --color-slate-600: oklch(44.6% 0.043 257.281);
    --color-slate-700: oklch(37.2% 0.044 257.287);
    --color-slate-900: oklch(20.8% 0.042 265.755);
    --color-white: #fff;
    --spacing: 0.25rem;
    --text-xs: 0.75rem;
    --text-xs--line-height: calc(1 / 0.75);
    --text-sm: 0.875rem;
    --text-sm--line-height: calc(1.25 / 0.875);
    --font-weight-medium: 500;
    --radius-md: 0.375rem;
    --default-transition-duration: 150ms;
    --default-transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
    --default-font-family: var(--font-sans);
    --default-mono-font-family: var(--font-mono);
  }
}

@layer base {

  *,
  ::after,
  ::before,
  ::backdrop,
  ::file-selector-button {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
    border: 0 solid;
  }

  html,
  :host {
    line-height: 1.5;
    -webkit-text-size-adjust: 100%;
    tab-size: 4;
    font-family: var(--default-font-family, ui-sans-serif, system-ui, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji");
    font-feature-settings: var(--default-font-feature-settings, normal);
    font-variation-settings: var(--default-font-variation-settings, normal);
    -webkit-tap-highlight-color: transparent;
  }

  hr {
    height: 0;
    color: inherit;
    border-top-width: 1px;
  }

  abbr:where([title]) {
    -webkit-text-decoration: underline dotted;
    text-decoration: underline dotted;
  }

  h1,
  h2,
  h3,
  h4,
  h5,
  h6 {
    font-size: inherit;
    font-weight: inherit;
  }

  a {
    color: inherit;
    -webkit-text-decoration: inherit;
    text-decoration: inherit;
  }

  b,
  strong {
    font-weight: bolder;
  }

  code,
  kbd,
  samp,
  pre {
    font-family: var(--default-mono-font-family, ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace);
    font-feature-settings: var(--default-mono-font-feature-settings, normal);
    font-variation-settings: var(--default-mono-font-variation-settings, normal);
    font-size: 1em;
  }

  small {
    font-size: 80%;
  }

  sub,
  sup {
    font-size: 75%;
    line-height: 0;
    position: relative;
    vertical-align: baseline;
  }

  sub {
    bottom: -0.25em;
  }

  sup {
    top: -0.5em;
  }

  table {
    text-indent: 0;
    border-color: inherit;
    border-collapse: collapse;
  }

  :-moz-focusring {
    outline: auto;
  }

  progress {
    vertical-align: baseline;
  }

  summary {
    display: list-item;
  }

  ol,
  ul,
  menu {
    list-style: none;
  }

  img,
  svg,
  video,
  canvas,
  audio,
  iframe,
  embed,
  object {
    display: block;
    vertical-align: middle;
  }

  img,
  video {
    max-width: 100%;
    height: auto;
  }

  button,
  input,
  select,
  optgroup,
  textarea,
  ::file-selector-button {
    font: inherit;
    font-feature-settings: inherit;
    font-variation-settings: inherit;
    letter-spacing: inherit;
    color: inherit;
    border-radius: 0;
    background-color: transparent;
    opacity: 1;
  }

  :where(select:is([multiple], [size])) optgroup {
    font-weight: bolder;
  }

  :where(select:is([multiple], [size])) optgroup option {
    padding-inline-start: 20px;
  }

  ::file-selector-button {
    margin-inline-end: 4px;
  }

  ::placeholder {
    opacity: 1;
  }

  @supports (not (-webkit-appearance: -apple-pay-button)) or (contain-intrinsic-size: 1px) {
    ::placeholder {
      color: currentcolor;

      @supports (color: color-mix(in lab, red, red)) {
        color: color-mix(in oklab, currentcolor 50%, transparent);
      }
    }
  }

  textarea {
    resize: vertical;
  }

  ::-webkit-search-decoration {
    -webkit-appearance: none;
  }

  ::-webkit-date-and-time-value {
    min-height: 1lh;
    text-align: inherit;
  }

  ::-webkit-datetime-edit {
    display: inline-flex;
  }

  ::-webkit-datetime-edit-fields-wrapper {
    padding: 0;
  }

  ::-webkit-datetime-edit,
  ::-webkit-datetime-edit-year-field,
  ::-webkit-datetime-edit-month-field,
  ::-webkit-datetime-edit-day-field,
  ::-webkit-datetime-edit-hour-field,
  ::-webkit-datetime-edit-minute-field,
  ::-webkit-datetime-edit-second-field,
  ::-webkit-datetime-edit-millisecond-field,
  ::-webkit-datetime-edit-meridiem-field {
    padding-block: 0;
  }

  ::-webkit-calendar-picker-indicator {
    line-height: 1;
  }

  :-moz-ui-invalid {
    box-shadow: none;
  }

  button,
  input:where([type="button"], [type="reset"], [type="submit"]),
  ::file-selector-button {
    appearance: button;
  }

  ::-webkit-inner-spin-button,
  ::-webkit-outer-spin-button {
    height: auto;
  }

  [hidden]:where(:not([hidden="until-found"])) {
    display: none !important;
  }
}

@layer utilities {
  .absolute {
    position: absolute;
  }

  .relative {
    position: relative;
  }

  .top-full {
    top: 100%;
  }

  .left-0 {
    left: calc(var(--spacing) * 0);
  }

  .z-50 {
    z-index: 50;
  }

  .mt-2 {
    margin-top: calc(var(--spacing) * 2);
  }

  .mb-1 {
    margin-bottom: calc(var(--spacing) * 1);
  }

  .mb-2 {
    margin-bottom: calc(var(--spacing) * 2);
  }

  .block {
    display: block;
  }

  .flex {
    display: flex;
  }

  .grid {
    display: grid;
  }

  .inline-block {
    display: inline-block;
  }

  .inline-flex {
    display: inline-flex;
  }

  .h-4 {
    height: calc(var(--spacing) * 4);
  }

  .h-5 {
    height: calc(var(--spacing) * 5);
  }

  .h-9 {
    height: calc(var(--spacing) * 9);
  }

  .w-4 {
    width: calc(var(--spacing) * 4);
  }

  .w-5 {
    width: calc(var(--spacing) * 5);
  }

  .w-56 {
    width: calc(var(--spacing) * 56);
  }

  .w-64 {
    width: calc(var(--spacing) * 64);
  }

  .cursor-not-allowed {
    cursor: not-allowed;
  }

  .cursor-pointer {
    cursor: pointer;
  }

  .grid-cols-7 {
    grid-template-columns: repeat(7, minmax(0, 1fr));
  }

  .items-center {
    align-items: center;
  }

  .justify-between {
    justify-content: space-between;
  }

  .justify-center {
    justify-content: center;
  }

  .gap-1 {
    gap: calc(var(--spacing) * 1);
  }

  .gap-2 {
    gap: calc(var(--spacing) * 2);
  }

  .gap-3 {
    gap: calc(var(--spacing) * 3);
  }

  .rounded {
    border-radius: 0.25rem;
  }

  .rounded-md {
    border-radius: var(--radius-md);
  }

  .border {
    border-style: var(--tw-border-style);
    border-width: 1px;
  }

  .border-slate-200 {
    border-color: var(--color-slate-200);
  }

  .border-slate-300 {
    border-color: var(--color-slate-300);
  }

  .bg-indigo-600 {
    background-color: var(--color-indigo-600);
  }

  .bg-slate-50 {
    background-color: var(--color-slate-50);
  }

  .bg-transparent {
    background-color: transparent;
  }

  .bg-white {
    background-color: var(--color-white);
  }

  .p-1 {
    padding: calc(var(--spacing) * 1);
  }

  .p-3 {
    padding: calc(var(--spacing) * 3);
  }

  .px-3 {
    padding-inline: calc(var(--spacing) * 3);
  }

  .py-2 {
    padding-block: calc(var(--spacing) * 2);
  }

  .text-center {
    text-align: center;
  }

  .text-left {
    text-align: left;
  }

  .text-sm {
    font-size: var(--text-sm);
    line-height: var(--tw-leading, var(--text-sm--line-height));
  }

  .text-xs {
    font-size: var(--text-xs);
    line-height: var(--tw-leading, var(--text-xs--line-height));
  }

  .font-medium {
    --tw-font-weight: var(--font-weight-medium);
    font-weight: var(--font-weight-medium);
  }

  .text-red-600 {
    color: var(--color-red-600);
  }

  .text-slate-400 {
    color: var(--color-slate-400);
  }

  .text-slate-500 {
    color: var(--color-slate-500);
  }

  .text-slate-600 {
    color: var(--color-slate-600);
  }

  .text-slate-700 {
    color: var(--color-slate-700);
  }

  .text-slate-900 {
    color: var(--color-slate-900);
  }

  .text-white {
    color: var(--color-white);
  }

  .capitalize {
    text-transform: capitalize;
  }

  .opacity-50 {
    opacity: 50%;
  }

  .shadow {
    --tw-shadow: 0 1px 3px 0 var(--tw-shadow-color, rgb(0 0 0 / 0.1)), 0 1px 2px -1px var(--tw-shadow-color, rgb(0 0 0 / 0.1));
    box-shadow: var(--tw-inset-shadow), var(--tw-inset-ring-shadow), var(--tw-ring-offset-shadow), var(--tw-ring-shadow), var(--tw-shadow);
  }

  .shadow-lg {
    --tw-shadow: 0 10px 15px -3px var(--tw-shadow-color, rgb(0 0 0 / 0.1)), 0 4px 6px -4px var(--tw-shadow-color, rgb(0 0 0 / 0.1));
    box-shadow: var(--tw-inset-shadow), var(--tw-inset-ring-shadow), var(--tw-ring-offset-shadow), var(--tw-ring-shadow), var(--tw-shadow);
  }

  .shadow-sm {
    --tw-shadow: 0 1px 3px 0 var(--tw-shadow-color, rgb(0 0 0 / 0.1)), 0 1px 2px -1px var(--tw-shadow-color, rgb(0 0 0 / 0.1));
    box-shadow: var(--tw-inset-shadow), var(--tw-inset-ring-shadow), var(--tw-ring-offset-shadow), var(--tw-ring-shadow), var(--tw-shadow);
  }

  .transition-colors {
    transition-property: color, background-color, border-color, outline-color, text-decoration-color, fill, stroke, --tw-gradient-from, --tw-gradient-via, --tw-gradient-to;
    transition-timing-function: var(--tw-ease, var(--default-transition-timing-function));
    transition-duration: var(--tw-duration, var(--default-transition-duration));
  }

  .hover\:border-slate-400 {
    &:hover {
      @media (hover: hover) {
        border-color: var(--color-slate-400);
      }
    }
  }

  .hover\:bg-indigo-50 {
    &:hover {
      @media (hover: hover) {
        background-color: var(--color-indigo-50);
      }
    }
  }

  .hover\:bg-slate-100 {
    &:hover {
      @media (hover: hover) {
        background-color: var(--color-slate-100);
      }
    }
  }

  .focus\:ring-2 {
    &:focus {
      --tw-ring-shadow: var(--tw-ring-inset, ) 0 0 0 calc(2px + var(--tw-ring-offset-width)) var(--tw-ring-color, currentcolor);
      box-shadow: var(--tw-inset-shadow), var(--tw-inset-ring-shadow), var(--tw-ring-offset-shadow), var(--tw-ring-shadow), var(--tw-shadow);
    }
  }

  .focus\:ring-indigo-500 {
    &:focus {
      --tw-ring-color: var(--color-indigo-500);
    }
  }

  .focus\:outline-none {
    &:focus {
      --tw-outline-style: none;
      outline-style: none;
    }
  }
}

@property --tw-border-style {
  syntax: "*";
  inherits: false;
  initial-value: solid;
}

@property --tw-font-weight {
  syntax: "*";
  inherits: false;
}

@property --tw-shadow {
  syntax: "*";
  inherits: false;
  initial-value: 0 0 #0000;
}

@property --tw-shadow-color {
  syntax: "*";
  inherits: false;
}

@property --tw-shadow-alpha {
  syntax: "<percentage>";
  inherits: false;
  initial-value: 100%;
}

@property --tw-inset-shadow {
  syntax: "*";
  inherits: false;
  initial-value: 0 0 #0000;
}

@property --tw-inset-shadow-color {
  syntax: "*";
  inherits: false;
}

@property --tw-inset-shadow-alpha {
  syntax: "<percentage>";
  inherits: false;
  initial-value: 100%;
}

@property --tw-ring-color {
  syntax: "*";
  inherits: false;
}

@property --tw-ring-shadow {
  syntax: "*";
  inherits: false;
  initial-value: 0 0 #0000;
}

@property --tw-inset-ring-color {
  syntax: "*";
  inherits: false;
}

@property --tw-inset-ring-shadow {
  syntax: "*";
  inherits: false;
  initial-value: 0 0 #0000;
}

@property --tw-ring-inset {
  syntax: "*";
  inherits: false;
}

@property --tw-ring-offset-width {
  syntax: "<length>";
  inherits: false;
  initial-value: 0px;
}

@property --tw-ring-offset-color {
  syntax: "*";
  inherits: false;
  initial-value: #fff;
}

@property --tw-ring-offset-shadow {
  syntax: "*";
  inherits: false;
  initial-value: 0 0 #0000;
}

@layer properties {
  @supports ((-webkit-hyphens: none) and (not (margin-trim: inline))) or ((-moz-orient: inline) and (not (color:rgb(from red r g b)))) {

    *,
    ::before,
    ::after,
    ::backdrop {
      --tw-border-style: solid;
      --tw-font-weight: initial;
      --tw-shadow: 0 0 #0000;
      --tw-shadow-color: initial;
      --tw-shadow-alpha: 100%;
      --tw-inset-shadow: 0 0 #0000;
      --tw-inset-shadow-color: initial;
      --tw-inset-shadow-alpha: 100%;
      --tw-ring-color: initial;
      --tw-ring-shadow: 0 0 #0000;
      --tw-inset-ring-color: initial;
      --tw-inset-ring-shadow: 0 0 #0000;
      --tw-ring-inset: initial;
      --tw-ring-offset-width: 0px;
      --tw-ring-offset-color: #fff;
      --tw-ring-offset-shadow: 0 0 #0000;
    }
  }
}
</style>