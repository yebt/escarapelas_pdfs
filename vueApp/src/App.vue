<script setup>
import { ref, onMounted } from 'vue'
//
import html2pdf from 'html2pdf.js'
//
import VueEscarapela from './components/VueEscarapela.vue'
//
// import allData from './assets/data/data_test_3.json'
import allData from './assets/data/data_real.json'
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
  const factor = 2

  downloadStatus.value = true
  const contentToPrint = document.getElementById('contentToPrint')

  await html2pdf().set({
    filename: 'escarapela_' + Date.now() + '.pdf',
    image: { type: 'webp', quality: 1 },
    jsPDF: { format: [w * factor, h * factor], orientation: 'landscape' }
  }).from(contentToPrint).save()

  downloadStatus.value = false
  // mark page
  downloadedPages.value.push(currentPage.value)
  putInLocalStorage()
}

const putInLocalStorage = () => {
  localStorage.setItem('dwnl', JSON.stringify(
    {
      downloadedPages: downloadedPages.value,
      currentPage: currentPage.value
    }))
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
          <VueEscarapela :vname="person.name" :vroleType="person.role ? 'Rol' : 'Disciplica'"
            :vrole="person.sport ?? person.role" :vrh="person.rh ?? null" :vsede="person.sede"
            :vdocType="person.role ? 'CC' : 'Cod'" :vcod="person.role ? person.document_number : person.student_code"
            :vvirtagHash="person.virtag_hash" />
        </div>

      </div>
    </div>

  </div>

  <div class="floating-button-container">
    <div style="display: flex; justify-content: center; min-width: 100%;">
      <div class="informer">
        <span>Page:</span> <span><input class="input" type="number" v-model="currentPage" @input="validatePageNum" /></span>
         <!-- {{ currentPage }} -->
        <span>/{{ Math.ceil(allData.length / OFFSET) }}</span>
        <span>-{{ downloadedPages.includes(currentPage) ? '✓' : 'x' }}</span>
      </div>
    </div>

    <button class="button tick" :class="{ disabled: currentPage <= 1 }" @click="decreasePage">
      <svg class="svg-icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" d="M15.75 19.5L8.25 12l7.5-7.5" />
      </svg>
    </button>

    <button class="button" :class="{ disabled: downloadStatus }" @click="makePdf">
      <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"
        class="svg-icon" :class="{ hidden: downloadStatus }">
        <path stroke-linecap="round" stroke-linejoin="round"
          d="M3 16.5v2.25A2.25 2.25 0 005.25 21h13.5A2.25 2.25 0 0021 18.75V16.5M16.5 12L12 16.5m0 0L7.5 12m4.5 4.5V3" />
      </svg>
      <span class="spinner" :class="{ hidden: !downloadStatus }"></span>
    </button>

    <button class="button tick" :class="{ disabled: currentPage >= Math.ceil(allData.length / OFFSET) }"
      @click="increasePage">
      <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"
        class="svg-icon">
        <path stroke-linecap="round" stroke-linejoin="round" d="M8.25 4.5l7.5 7.5-7.5 7.5" />
      </svg>
    </button>

  </div>
</template>

<style scoped></style>
