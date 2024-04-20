<script setup>
import { ref, onMounted } from 'vue'
//
import html2pdf from 'html2pdf.js'
//
import VueEscarapela from './components/VueEscarapela.vue'
//
// import allData from './assets/data/data_test_3.json'
// import allData from './assets/data/data_real_2.json'
// import allData from './assets/data/data_real_3.json'
// import allData2 from './assets/data/data_real_4.json'
// import assistantData from './assets/data/data_asistente.json'
// import genericData from './assets/data/data_generics.json'

import jsonEvaluadores from './assets/json/evaluadores_ok.json'
import jsonGenericos from './assets/json/genericos_ok.json'
import jsonMonitores from './assets/json/monitores_ok.json'
import jsonOrganizadores from './assets/json/organizadores_ok.json'
import jsonPonentes from './assets/json/ponentes_ok.json'

const groupedBySedePonentes = jsonPonentes.reduce((acc, current) => {
  const { sede } = current
  if (!acc[sede]) {
    acc[sede] = []
  }
  acc[sede].push(current)
  return acc
}, {})
const groupedArrayPnentes = []
Object.entries(groupedBySedePonentes).forEach(([sede, elementos]) => {
  groupedArrayPnentes.push(...elementos)
}
)

const allData = [
  ...jsonEvaluadores,
  ...jsonMonitores,
  ...jsonOrganizadores,
  // ...jsonPonentes,
  ...jsonGenericos,
  ...groupedArrayPnentes,
]

// const blanckCount = 3
// for (let i = 0; i < blanckCount; i++) {
//   allData.push({
//     type: 'blank'
//   })
// }
// add assistant
// for (let i = 0; i < assistantData.length; i++) {
//   allData.push(assistantData[i])
// }
// for (let i = 0; i < allData2.length; i++) {
//   allData.push(allData2[i])
// }
// //
// for (let i = 0; i < genericData.length; i++) {
//   allData.push(genericData[i])
// }
//
const paginatedData = ref([])
const currentPage = ref(1)
const downloadedPages = ref([])
const downloadStatus = ref(false)
//
const OFFSET = 8
//
const validatePageNum = () => {
  if (currentPage.value > Math.ceil(allData.length / OFFSET)) {
    currentPage.value = Math.ceil(allData.length / OFFSET)
  }
  if (currentPage.value < 1) {
    currentPage.value = 1
  }
  makePaginatedData()
}
const decreasePage = () => {
  if (currentPage.value <= 1) {
    return
  }
  currentPage.value--
  makePaginatedData()
}
const increasePage = () => {
  if (currentPage.value >= Math.ceil(allData.length / OFFSET)) {
    return
  }
  currentPage.value++
  makePaginatedData()
}

const makePaginatedData = () => {
  const startIndex = (currentPage.value - 1) * OFFSET
  const endIndex = startIndex + OFFSET
  paginatedData.value = allData.slice(startIndex, endIndex)
  putInLocalStorage()
}
//
const makePdf = async () => {
  const w = 500
  const h = 300
  const factor = 1.5

  downloadStatus.value = true
  const contentToPrint = document.getElementById('contentToPrint')

  await html2pdf()
    .set({
      filename: 'escarapelas_' + currentPage.value + '.pdf',
      image: { type: 'webp', quality: 1 },
      jsPDF: { format: [w * factor, h * factor], orientation: 'landscape' }
    })
    .from(contentToPrint)
    .save()

  downloadStatus.value = false
  // mark page
  downloadedPages.value.push(currentPage.value)
  putInLocalStorage()
}

const makeAllPdf = async () => {
  for (let i = 1; i <= Math.ceil(allData.length / OFFSET); i++) {
    currentPage.value = i
    const startIndex = (currentPage.value - 1) * OFFSET
    const endIndex = startIndex + OFFSET
    paginatedData.value = allData.slice(startIndex, endIndex)
    await makePdf()
    if (downloadStatus.value) {
      break
    }
  }
}

const putInLocalStorage = () => {
  localStorage.setItem(
    'dwnl',
    JSON.stringify({
      downloadedPages: downloadedPages.value,
      currentPage: currentPage.value
    })
  )
}
const getFromLocalStorage = () => {
  const dwnl = JSON.parse(localStorage.getItem('dwnl'))
  if (dwnl) {
    downloadedPages.value = dwnl.downloadedPages ?? []
    currentPage.value = dwnl.currentPage ?? 1
  }
}
//
onMounted(() => {
  getFromLocalStorage()
  makePaginatedData()
})
</script>

<template>
  <div class="hidden-">
    <div class="outer-container" id="contentToPrint">
      <div class="inner-container">
        <!--  -->
        <div v-for="(person, index) in paginatedData" :key="index">
          <VueEscarapela
            :vname="person.name ?? null"
            :vrole="person.rol"
            :vies="person.ies"
            :vsede="person.sede"
            :vvirtagHash="person.virtag_hash"
          />
        </div>
      </div>
    </div>
  </div>

  <div class="floating-button-container">
    <div style="display: flex; justify-content: center; min-width: 100%">
      <div class="informer">
        <span>Page:</span>
        <span
          ><input
            class="input"
            type="number"
            v-model="currentPage"
            @input="validatePageNum"
        /></span>
        <!-- {{ currentPage }} -->
        <span>/{{ Math.ceil(allData.length / OFFSET) }}</span>
        <span>-{{ downloadedPages.includes(currentPage) ? "✓" : "x" }}</span>
      </div>
    </div>

    <button
      class="button"
      :class="{ disabled: downloadStatus }"
      @click="makeAllPdf"
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 24 24"
        fill="currentColor"
        class="svg-icon"
        :class="{ hidden: downloadStatus }"
      >
        <path
          fill-rule="evenodd"
          d="M9.75 6.75h-3a3 3 0 00-3 3v7.5a3 3 0 003 3h7.5a3 3 0 003-3v-7.5a3 3 0 00-3-3h-3V1.5a.75.75 0 00-1.5 0v5.25zm0 0h1.5v5.69l1.72-1.72a.75.75 0 111.06 1.06l-3 3a.75.75 0 01-1.06 0l-3-3a.75.75 0 111.06-1.06l1.72 1.72V6.75z"
          clip-rule="evenodd"
        />
        <path
          d="M7.151 21.75a2.999 2.999 0 002.599 1.5h7.5a3 3 0 003-3v-7.5c0-1.11-.603-2.08-1.5-2.599v7.099a4.5 4.5 0 01-4.5 4.5H7.151z"
        />
      </svg>
      <span class="spinner" :class="{ hidden: !downloadStatus }"></span>
    </button>

    <button
      class="button tick"
      :class="{ disabled: currentPage <= 1 }"
      @click="decreasePage"
    >
      <svg
        class="svg-icon"
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 24 24"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          d="M15.75 19.5L8.25 12l7.5-7.5"
        />
      </svg>
    </button>

    <button
      class="button"
      :class="{ disabled: downloadStatus }"
      @click="makePdf"
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke-width="1.5"
        stroke="currentColor"
        class="svg-icon"
        :class="{ hidden: downloadStatus }"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          d="M3 16.5v2.25A2.25 2.25 0 005.25 21h13.5A2.25 2.25 0 0021 18.75V16.5M16.5 12L12 16.5m0 0L7.5 12m4.5 4.5V3"
        />
      </svg>
      <span class="spinner" :class="{ hidden: !downloadStatus }"></span>
    </button>

    <button
      class="button tick"
      :class="{ disabled: currentPage >= Math.ceil(allData.length / OFFSET) }"
      @click="increasePage"
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke-width="1.5"
        stroke="currentColor"
        class="svg-icon"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          d="M8.25 4.5l7.5 7.5-7.5 7.5"
        />
      </svg>
    </button>
  </div>
</template>

<style scoped></style>
