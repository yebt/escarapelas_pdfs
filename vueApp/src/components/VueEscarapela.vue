<script setup>
import { computed, watch, ref, onMounted, toRefs } from 'vue'
import QRCode from 'qrcode'

const props = defineProps({
  vname: String,
  vrole: String,
  vies: String,
  vsede: String,
  vvirtagHash: String
})

const { vname, vrole, vies, vsede, vvirtagHash } = toRefs(props)

const capitalizeWords = (strtop) => {
  strtop = strtop.trim().toLowerCase()
  return strtop
    .split(' ')
    .map((word) => word.charAt(0).toUpperCase() + word.slice(1))
    .join(' ')
}

const showSede = computed(() => {
  return capitalizeWords(vsede.value.toLocaleLowerCase())
})
const showRol = computed(() => {
  return capitalizeWords(vrole.value.toLocaleLowerCase())
})

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
  if (!props.vvirtagHash) {
    console.log(props)
    vqrCode.value = ''
    return
  }
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
    console.log(props.vvirtagHash)
    genQr()
  }
)

const vqrCode = ref('')
onMounted(() => {
  genQr()
})
</script>

<template>
  <div class="box">
    <div class="spacer">
      <img
        class="svg-univalle"
        src="../assets/imgs/logos/univalle.svg"
        alt="univalle logo"
      />
      <img
        class="svg-uceva"
        src="../assets/imgs/logos/uceva.svg"
        alt="uceva logo"
      />
      <img
        class="svg-uniminuto"
        src="../assets/imgs/logos/unuminuto.svg"
        alt="Uniminuto logo"
      />
      <img
        class="svg-remington"
        src="../assets/imgs/logos/reminton.svg"
        alt="remington logo"
      />
      <img
        class="svg-reditool"
        src="../assets/imgs/logos/reditul.svg"
        alt="reditool logo"
      />
      <div>

        <div  class="title-name">
          <span v-if="vrole.toLocaleLowerCase() === 'ponente'">{{ showShortName }}</span>
          <span v-else class="placeholder">
            <div class="placeholder-name"></div>
          </span>
        </div>

        <div class="title-description">
          <span>IES:</span> <span>{{ vies }}</span>
          <span>
            <span> / </span>
            <span>Sede:</span> <span>{{ showSede }}</span>
          </span>
        </div>
        <div class="title-role">{{ showRol }}</div>
      </div>
      <div>
        <div  class="qr-code">
          <img :src="vqrCode" alt="qr" />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.placeholder {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 70px;
}
.placeholder-name {
  width: 440px;
  height: 54px;
  background-color: #eee;
  border-radius: 5px;
}
.svg-univalle {
  position: absolute;
  bottom: 26px;
  left: 41px;
  width: 46px;
  height: 79px;
}
.svg-uceva {
  position: absolute;
  bottom: 30px;
  left: 105px;
  width: 134px;
  height: auto;
}
.svg-uniminuto {
  position: absolute;
  bottom: 31px;
  left: 341px;
  width: 103px;
  height: auto;
}
.svg-remington {
  position: absolute;
  bottom: 25px;
  left: 460px;
  width: 118px;
  height: auto;
}

.svg-reditool {
  position: absolute;
  bottom: 127px;
  left: 202px;
  width: 81px;
  height: auto;
}
.qr-code {
  position: absolute;
  bottom: 121px;
  left: 42px;
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
  margin: -4px;
}
</style>
