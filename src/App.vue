<script setup>
import { ref, reactive, onMounted, computed  } from "vue"
import Alerta from "./components/Alerta.vue"


  const monedas = ref([
      { codigo: 'USD', texto: 'Dolar de Estados Unidos'},
      { codigo: 'MXN', texto: 'Peso Mexicano'},
      { codigo: 'EUR', texto: 'Euro'},
      { codigo: 'GBP', texto: 'Libra Esterlina'},
  ])

  const criptomonedas = ref ([])
  const error = ref('')
  const cotizar = reactive({
    moneda: '',
    criptomoneda: ''
  })
  const cotizacion = ref({})

  onMounted(() => {
    const url = 'https://min-api.cryptocompare.com/data/top/mktcapfull?limit=20&tsym=USD'
    fetch(url)
      .then(respuesta => respuesta.json())
      .then(({Data}) => {
      criptomonedas.value = Data
    })
  })

const cotizarCripto = () => {

  if(Object.values(cotizar).includes('')){
     error.value = 'Todos los campos son obligatorios'
     setTimeout(() => {
      error.value = ''
     },3000)
    }
    obtenerCotizacion()
}

const obtenerCotizacion =  async() =>{

  const { moneda, criptomoneda } = cotizar
  const url = `https://min-api.cryptocompare.com/data/pricemultifull?fsyms=${criptomoneda}&tsyms=${moneda}`
  console.log(url)

  const respuesta = await fetch(url)
  const data = await respuesta.json()

  cotizacion.value = data.DISPLAY[criptomoneda][moneda]
  console.log(cotizacion)
}

  const mostrarResultado = computed(() => {
    return Object.values(cotizacion.value). length > 0
  })
</script>

<template>
  <div class="ml-20 mt-20">
    <h1 class="font-extrabold text-white text-4xl">Cotizador de
      <br>
      <span class="text-emerald-400 text-5xl">
        Criptomonedas
      </span>
    </h1>
    <div class="my-10 bg-white py-10 px-15 rounded-xl w-2/3 xl:w-1/3">
      <form @submit.prevent="cotizarCripto"
            class="flex flex-col gap-8 font-semibold"
      >
        <!-- Alerta -->
        <Alerta v-if="error">
        {{ error }}
        </Alerta>
        <!-- Moneda -->
        <div class="flex flex-col gap-5">
          <label class="" for="moneda">Moneda: </label>
          <select id="moneda"
                  v-model="cotizar.moneda"
                  class="cursor-pointer bg-gray-300 rounded-xl p-2 hover:border-blue-500 italic font-light">
            <option value="">-- Seleccione --</option>
            <option v-for="moneda in monedas" :value="moneda.codigo"> {{ moneda.texto }}</option>
          </select>
        </div>
        <!-- Criptomonedas -->
        <div class="flex flex-col gap-5">
          <label class="" for="cripto">Criptomoneda: </label>
          <select id="cripto"
                  v-model="cotizar.criptomoneda"
                  class="cursor-pointer bg-gray-300 rounded-xl p-2 hover:border-blue-500 italic font-light">
            <option value="">-- Seleccione --</option>
            <option v-for="criptomoneda in criptomonedas" :value="criptomoneda.CoinInfo.Name"> {{ criptomoneda.CoinInfo.FullName }}</option>
          </select>
        </div>

        <input  type="submit" value="COTIZAR"
                class="bg-emerald-400 hover:bg-emerald-500 hover:text-white p-2 rounded-xl cursor-pointer"
        >
      </form>
      <!-- Cotización -->
      <div class="flex flex-col justify-center items-center mt-10"
            v-if="mostrarResultado">
        <h2 class="font-extrabold text-4xl">Cotización</h2>
        <div class="flex mt-5 gap-5 items-center mx-auto justify-center">
          <img
                :src="'https://cryptocompare.com' + cotizacion.IMAGEURL"
                alt="Imagen de Criptomoneda"
                class="w-1/3"
          >
          <div>
            <p> Precio de: <span class="font-black">{{ cotizacion.PRICE }}</span></p>
            <p> Precio más alto del día: <span class="font-black">{{ cotizacion.HIGHDAY }}</span></p>
            <p> Precio más bajo del día: <span class="font-black">{{ cotizacion.LOWDAY }}</span></p>
            <p> Porcentaje de variación últimas 24hrs: <span class="font-black">{{ cotizacion.CHANGEPCT24HOUR }}%</span></p>
            <p class="italic text-sm mt-5"> Última Actualización: <span class="font-black">{{ cotizacion.LASTUPDATE }}</span></p>

          </div>
        </div>
      </div>
    </div>
  </div>
</template>
