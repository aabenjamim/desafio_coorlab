<template>
  <div class="title">
    <b-navbar toggleable="lg" type="dark" variant="info">
      <b-navbar-brand class="ml-2">
        <b>{{ appName }}</b>
      </b-navbar-brand>
    </b-navbar>

    <div class="container">

      <div class="top">
        <img src="/favicon.ico" alt="Ícone" />
        <h1>Melhor Frete</h1>
      </div>

      <div class="menu">

        <div class="choices">
          <h1>Insira o destino e o peso</h1>
          <p>Destino</p>
          <select v-model="selectedCity">
            <option value="" disabled selected>Selecione uma cidade</option>
            <option v-for="city in transport" :value="city.city" :key="city.id">{{ city.city }}</option>
          </select>
          <p>Peso</p>
          <input v-model="loadWeight" placeholder="Peso da carga em kg" type="number"/>
          <button @click="analyze">Analisar</button>
        </div>

        <div class="views">

        </div>

      </div>

    </div>

  </div>

</template>

<script>
import {
  BNavbar,
  BNavbarBrand,
} from 'bootstrap-vue'
import axios from 'axios'

export default {
  components: {
    BNavbar,
    BNavbarBrand,
  },
  data() {
    const appName = ''

    return {
      appName,
      transport: [],
      selectedCity: '',
      loadWeight: null,
      economicShipping: null,
      fastShipping: null,
    }
  },
  created() {
    // Implemente aqui o GET dos dados da API REST
    // para que isso ocorra na inicialização da pagina
    this.appName = 'Melhor Frete'

    axios.get('http://localhost:3000/transport')
    .then(res => {
      this.transport = res.data;
    })
    .catch(err => {
      console.error(err);
    });
  },
  methods: {
    // Implemente aqui os metodos utilizados na pagina
    methodFoo() {
      console.log(this.appName)
    },
    analyze() {
      const weight = parseInt(this.loadWeight);
      const city = this.selectedCity;

      if (!isNaN(weight) && city) {
        const filteredFreight = this.transport.filter(item => item.city === city);

        if (filteredFreight.length > 0) {

          filteredFreight.sort((a, b) => {
            const costA = weight <= 100 ? parseFloat(a.cost_transport_light.slice(3)) : parseFloat(a.cost_transport_heavy.slice(3));
            const costB = weight <= 100 ? parseFloat(b.cost_transport_light.slice(3)) : parseFloat(b.cost_transport_heavy.slice(3));
            return costA - costB;
          });
          this.economicShipping = filteredFreight[0];

          filteredFreight.sort((a, b) => {
            const timeA = parseInt(a.lead_time);
            const timeB = parseInt(b.lead_time);
            return timeA - timeB;
          });
          
          this.fastShipping = filteredFreight[0];
        } else {
          this.economicShipping = null;
          this.fastShipping = null;
        }
      } else {
        this.economicShipping = null;
        this.fastShipping = null;
      }
    },
  }
}
</script>

<style scoped>
.title .navbar {
  background-color: #00aca6 !important;
}

.title .navbar-brand {
  margin-left: 20px;
}

.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin-top: 80px;
  width: 80%;
  border-radius: 30px;
}

.top {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: start;
  gap: 15px;
  padding-left: 15px;
  background-color: #89B6C2;
  border-top-left-radius: 15px;
  border-top-right-radius: 15px;
  width: 100%;
  height: 60px;
  & h1{
    font-size: 20px;
    color: #343D32;
  }
  & img{
    width: 25px;
  }
}

.menu{
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items:center;
  width: 100%;
  padding: 15px;
  height: 450px;
  & h1{
    font-size: 20px;
    color: #343D32;
  }
}

.choices{
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 20px;
  background-color: #EEEEEF;
  border-radius: 15px;
  width: 40%;
  height: 90%;
  & h1{
    margin-bottom: 20px;
    color: #2E4850;
  }
  & p{
    font-size: 15px;
    color: #2E4850;
    font-weight: bold;
  }
  & select, input{
    width: 80%;
    margin-bottom: 18px;
    height: 30px;
    border-radius: 5px;
    border: 1px solid #FFFEEE;
    background-color: #FFFFFF;
    outline: none;
    padding: 0.5%;
    font-size: 15px;
  }
  & button{
    border: none;
    background-color: #89B6C2;
    border-radius: 5px;
    width: 90px;
    height: 30px;
    color: #2E4850;
  }
  & option{
    border: none;
  }
}

.views{
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items:center;
  width: 60%;
}

</style>
