<script setup>
import { computed, watch, ref, onMounted, toRefs } from 'vue'
import QRCode from 'qrcode'

const props = defineProps({
  vname: String,
  vroleType: String,
  vrole: String,
  vrh: String,
  vsede: String,
  vdocType: String,
  vcod: String,
  vvirtagHash: String
})

// const { vname, vrole, vrh, vsede, vdocType, vcod } = props
const { vname, vrole, vrh, vsede, vdocType, vcod } = toRefs(props)

const capitalizeWords = (strtop) => {
  strtop = strtop.trim().toLowerCase()
  return strtop.split(' ').map(word => word.charAt(0).toUpperCase() + word.slice(1)).join(' ')
}

const showShortName = computed(() => {
  // const lname = capitalizeWords(vname.value)
  let lname = vname.value
  if (!lname) {
    return ''
  }
  lname = capitalizeWords(lname)
  let cutName = ''

  const charlimit = 24
  if (lname.length <= charlimit) {
    return lname
  }
  const words = lname.split(' ')

  if (words[0].length > charlimit) {
    return words[0].substring(0, charlimit)
  }

  let charCount = 0
  for (const word of words) {
    if (charCount + word.length <= charlimit) {
      cutName += `${word} `
      charCount += word.length + 1 // +1 para el espacio
    } else {
      break
    }
  }
  return cutName.trim()
})
// make async computed qr
const genQr = () => {
  QRCode.toDataURL(props.vvirtagHash, {
    errorCorrectionLevel: 'H',
    width: 464,
    height: 464
  }).then((data) => {
    vqrCode.value = data
  })
}
watch(
  () => props.vvirtagHash,
  () => {
    genQr()
  }
)

const vqrCode = ref('')
onMounted(() => {
  genQr()
})
</script>

<template >
  <div class="box">
    <img class="svg-logo" src="../assets/imgs/logo_uni.svg" alt="unilogo" />

    <div class="spacer">
      <div>
        <div class="title-name">{{ showShortName }}</div>
        <div class="title-description">
          <span>{{vroleType}}:</span> <span>{{ vrole }}</span>
          <span v-if="vrh">
            <span> / </span>
            <span>Rh:</span> <span>{{ vrh }}</span>
          </span>
        </div>
      </div>
      <div>
        <div class="title-sede">
          Sede: <span>{{ vsede }}</span>
        </div>
        <div class="title-id">
          <span>{{ vdocType }}:</span> <span>{{ vcod }}</span>
        </div>
      </div>
      <div>
        <div class="qr-code">
          <img :src="vqrCode"  alt="qr"/>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.qr-code {
  position: absolute;
  bottom: 34px;
  left: 95px;
  overflow: hidden;
  border-radius: 5%;
  display: flex;
  /* display: flex;
  justify-content: center;
  align-items: center; */
  width: 116px;
  height: 116px;
}
.qr-code img {
  /* object-fit: cover; */
margin:-4px;
}
</style>
