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
          <b-modal v-model="modalShow" hide-footer hide-header centered>
            <div class="modal-header">
              <b-icon-exclamation-square></b-icon-exclamation-square>
            </div>
            <div class="modal-body">
              Insira os valores para realizar a análise.
            </div>
            <div class="modal-footer">
              <b-button @click="modalShow = false">Fechar</b-button>
            </div>
          </b-modal>
          <div v-if="!showResults">Nenhum dado selecionado</div>
          <div v-else class="painel">
            <p>Estas são as melhores alternativas de frete que encontramos para você</p>
              <div class="horizontal">git
                <div class="economic data">
                  <div class="icon">
                    <b-icon-cash-coin></b-icon-cash-coin>
                  </div>
                  <div class="inform">
                    <h1>Frete com menor valor</h1>
                    <p>Trasportadora: {{this.economicShipping.name}}</p>
                    <p>Tempo estimado: {{this.economicShipping.lead_time}}</p>
                  </div>
                </div>
                <div class="price-economic">
                  <h1>Preço</h1>
                  <p>R$ {{this.totalPriceEconomic}}</p>
                </div>
              </div>
              <div class="horizontal">
                <div class="fast data">
                  <div class="icon">
                    <b-icon-clock></b-icon-clock>
                  </div>
                  <div class="inform">
                    <h1>Frete mais rápido</h1>
                    <p>Trasportadora: {{this.fastShipping.name}}</p>
                    <p>Tempo estimado: {{this.fastShipping.lead_time}}</p>
                  </div>
                </div>
                <div class="price-fast">
                  <h1>Preço</h1>
                  <p>R$ {{this.totalPriceFast}}</p>
                </div>
              </div>
              <button>Limpar</button>
          </div>
        </div>
      </div>
    </div>
  </div>

</template>

<script>
import {
  BNavbar,
  BNavbarBrand,
  BModal
} from 'bootstrap-vue'
import axios from 'axios'
import { BIconExclamationSquare } from 'bootstrap-vue'

export default {
  components: {
    BNavbar,
    BNavbarBrand,
    BModal,
    BIconExclamationSquare,
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
      modalShow: false,
      showResults: false,
      totalPriceFast: null,
      totalPriceEconomic: null,

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
          if(weight<=100){
            const costEconomicLight = parseFloat(this.economicShipping.cost_transport_light.slice(3))*weight
            this.totalPriceEconomic = costEconomicLight.toFixed(2);
          } else if(weight>100){
            const costEconomicHeavy = parseFloat(this.economicShipping.cost_transport_heavy.slice(3))*weight
            this.totalPriceEconomic = costEconomicHeavy.toFixed(2);
          }

          filteredFreight.sort((a, b) => {
            const timeA = parseInt(a.lead_time);
            const timeB = parseInt(b.lead_time);
            return timeA - timeB;
          });
          
          this.fastShipping = filteredFreight[0];
          if(weight<=100){
            const costFastLight = parseFloat(this.fastShipping.cost_transport_light.slice(3))*weight
            this.totalPriceFast = costFastLight.toFixed(2);
          } else if(weight>100){
            const costFastHeavy = parseFloat(this.fastShipping.cost_transport_heavy.slice(3))*weight
            this.totalPriceFast = costFastHeavy.toFixed(2);
          }

          this.showResults = true;
        } else {
          this.economicShipping = null;
          this.fastShipping = null;
          this.showResults = false;
        }
      } else {
        this.economicShipping = null;
        this.fastShipping = null;
        this.showResults = false;
        this.modalShow = true;
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
  padding: 30px;
  gap: 15px;
  position: relative;
}

.modal-header {
  display: flex;
  justify-content: center;
  align-items: center;
  color: #343D32;
  font-size: 60px;
  border-bottom: none;

}

.modal-body {
  text-align: center;
  font-size: 20px;
}

.modal-footer {
  display: flex;
  justify-content: center;
  border-top: none;
  & button{
    background-color: #89B6C2;
    border: none;
    color: #2E4850;
    width: 90px;
    height: 30px;
    display: flex;
    align-items: center;
    justify-content: center;
  }
}

.data{
  width: 70%;
  display: flex;
  flex-direction: row;
  height: 80px;
  align-items: center;
  border-radius: 8px;
    & .icon{
    width: 25%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 35px;
    color: #343D32;
    border-top-left-radius: 8px;
    border-bottom-left-radius: 8px;
  }
  & .inform{
    display: flex;
    width: 75%;
    height: 100%;
    padding: 5px;
    flex-direction: column;
    justify-content: center;
    gap: 5px;
    & h1{
      font-weight: bold;
      font-size: 13px;
    }
    & p{
      font-size: 13px;
    }
  }
}

.inform h1, .inform p {
  margin: 0;
}

.economic{
  background-color: #D6DADA;
  & .icon{
    background-color: #BACEDA;
  }
}

.fast{
  background-color: #F3F3F0;
  & .icon{
    background-color: #8BB7C2;
  }
}

.price-economic{
  width: 30%;
  height: 80px;
  display: flex;
  flex-direction: column;
  align-items: start;
  justify-content: center;
  background-color: #D6DADA;
  border-radius: 8px;
  padding: 8px;
  & h1{
    font-weight: bold;
    font-size: 16px;
    margin: 0;
  }
  & p{
    font-size: 16px;
    margin: 0;
  }
}

.price-fast{
  width: 30%;
  height: 80px;
  display: flex;
  flex-direction: column;
  align-items: start;
  justify-content: center;
  border-radius: 8px;
  background-color: #F3F3F0;
  padding: 8px;
  & h1{
    font-weight: bold;
    font-size: 16px;
    margin: 0;
  }
  & p{
    font-size: 16px;
    margin: 0;
  }
}

.painel{
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 15px;
  & button{
    background-color: #89B6C2;
    border: none;
    color: #2E4850;
    width: 90px;
    height: 30px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 8px;
    position: absolute;
    bottom: -25px;
    right: 40px;
  }
}

.horizontal{
  display: flex;
  align-items: center;
  gap: 15px;
  width: 100%;
}

</style>
