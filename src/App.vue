<template>
  <mainComponent msg="detector" />
  <div v-if="code" v-show="!camStatus" class="text-center bg-transparent text-dark p-3 mb-4"> Codigo Lido {{ code }}</div>
  <div v-if="product" v-show="!camStatus" class="text-center bg-transparent text-dark p-3 mb-4"> Produto: {{ product }}</div>
  

  <div>
    <button @click="initReader" class="btn btn-primary">Iniciar Leitura</button>
  </div>

  <div class="text-center" v-show="camStatus" id="reader"></div>
</template>

<script>
import { ref } from 'vue';
import mainComponent from './components/mainComponent.vue'
import Quagga from 'quagga';
import 'bootstrap/dist/css/bootstrap.min.css';
import 'bootstrap/dist/js/bootstrap.bundle.min.js';
import axios from 'axios';

export default {
  name: 'App',
  components: {
    mainComponent
  },
  setup() {

    const code = ref('');
    const camStatus = ref('');
    const product = ref('');
    const initReader = () => {
      camStatus.value = true;
      Quagga.init({
        inputStream: {
          name: "Live",
          type: "LiveStream",
          target: document.querySelector('#reader')
        },
        decoder: {
          readers: ["code_128_reader", 'ean_reader',]
        }
      }, function (err) {
        if (err) {
          console.log(err);
          return
        }
        console.log("Initialization finished. Ready to start");
        Quagga.start();
        Quagga.onDetected((data) => {
          onDetected(data);
        })
      });
    }

    const stopReader = () => {
      camStatus.value = false;
      Quagga.stop()
    }

     const getProduct = async (barcode) => {

      try {
        const response = await axios.get(`https://world.openfoodfacts.org/api/v0/product/${barcode}.json`);
        product.value = response.data.product.product_name;
        console.log("Produto --");
        console.log(response.data);
        
      } catch (error) {
        console.error("Erro ao buscar produto:", error);
      }

    }

    const onDetected = (data) => {
      code.value = data.codeResult.code;
      stopReader();
      getProduct(data.codeResult.code);
    }
    return {
      initReader,
      stopReader,
      camStatus,
      code,
      product
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

/* .btn-primary {
  background-color: rgb(138, 138, 241);
  border: none;
  border-radius: 5px;
  padding: 15px;
  size: 5px;
  cursor: pointer;
  color: white;
} */

#reader {
  margin-top: 60px;
  justify-content: center;
  border: red 15px;
  text-align: center;
}
</style>
