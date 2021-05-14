<template>
  <div class="container">
    <div class="row mt-5">
      <div class="col">
        <h1 class="text-center">COVID-19 DATA</h1>
        <input type="text" v-model="country" class="col-md-2 mt-3" />
        <button @click="consultarDadosCovid(country)" class="ml-2">
          Carregue os dados
        </button>
        <button @click="consultarDadosComplexos(country)" disabled class="ml-2">
          Complexos
        </button>
        <button @click="limparDados" class="ml-2">Limpar consulta</button>
      </div>
    </div>
    <div class="row"><div class="row mt-5" v-if="arrPositive.length > 0">
      <div class="col">
        <h2 class="text-center">Positivos</h2>
        <line-chart
          :chartData="arrPositive"
          :options="chartOptions"
          :chartColors="positiveChartColors"
          label="Positive"
        />
      </div>
    </div>
    <div class="row mt-5" v-if="arrRecovered.length > 0">
      <div class="col">
        <h2 class="text-center">Recuperados</h2>
        <line-chart
          :chartData="arrRecovered"
          :options="chartOptions"
          :chartColors="recoveredColors"
          label="Recovered"
        />
      </div>
    </div>

    <div class="row mt-5 mb-5" v-if="arrDeaths.length > 0">
      <div class="col">
        <h2 class="text-center">Mortes totais</h2>
        <line-chart
          v-if="arrDeaths.length > 0"
          :chartData="arrDeaths"
          :options="chartOptions"
          :chartColors="deathColors"
          label="Deaths"
        />
      </div>
    </div></div>
    <div class="row mt-5 mb-5" v-if="arrTodayDeaths.length > 0">
      <div class="col">
        <h2 class="text-center">Mortes no dia de Hoje {{this.date}}</h2>
        <multiple-chart
          :chartData="arrTodayDeaths"
          :options="chartOptions"
          :label="date"
          :chartColors="arrTodayDeaths"
        />
      </div>
    </div>
       <div class="row mt-5 mb-5" v-if="arrComplex.length > 0">
      <div class="col">
        <h2 class="text-center">Testes para cada milhão de habitantes</h2>
        <pie
          :chartData="arrComplex"
          :options="chartOptions"
          :label="arrComplex.label"
          :chartColors="arrTodayDeaths"
        />
      </div>
    </div>
       <div class="row mt-5 mb-5" v-if="arrDeathsOne.length > 0">
      <div class="col">
        <h2 class="text-center">Mortes para cada milhão de habitantes</h2>
        <doughnut
          :chartData="arrDeathsOne"
          :options="chartOptions"
          :label="arrDeathsOne.label"
          :chartColors="arrTodayDeaths"
        />
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import moment from "moment";

import LineChart from "./components/LineChart";
import MultipleChart from './components/MultipleChart.vue';
import Pie from './components/Pie.vue';
import Doughnut from './components/doughnut.vue';


export default {
  components: {
    LineChart,
    MultipleChart,
    Doughnut,
    Pie
    },
  data() {
    return {
      datasets: [],
      date: null,
      country: null,
      countryNames: [],
      arrPositive: [],
      positiveChartColors: {
        borderColor: "#077187",
        pointBorderColor: "#0E1428",
        pointBackgroundColor: "#AFD6AC",
        backgroundColor: "#74A57F",
      },
      arrTodayDeaths: [],
      todayColors: {
        borderColor: "#00042A",
        pointBorderColor: "#0E1428",
        pointBackgroundColor: "#AFD6AC",
        backgroundColor: "#00042A",
      },
      arrRecovered: [],
      recoveredColors: {
        borderColor: "#4E5E66",
        pointBorderColor: "#4E5E66",
        pointBackgroundColor: "#31E981",
        backgroundColor: "#31E981",
      },
      arrDeaths: [],
      deathColors: {
        borderColor: "#E06D06",
        pointBorderColor: "#E06D06",
        pointBackgroundColor: "#402A2C",
        backgroundColor: "#402A2C",
      },
      arrComplex: [],
      arrDeathsOne: [],
      chartOptions: {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          yAxes: [
            {
              ticks: {
                beginAtZero: true,
                callback: function (value) {
                  if (value % 1 === 0 || value >= 1000) {
                    return value
                      .toString()
                      .replace(/\B(?=(\d{3})+(?!\d))/g, ".");
                  }
                },
              },
            },
          ],
        },
      },
    };
  },
  methods: {
    limparDados() {
      this.arrPositive = [];
      this.arrDeaths = [];
      this.arrRecovered = [];
    },
    consultarDadosCovid(country) {
      this.countryNames = [];
      let date = new Date();
      date = moment(date).format("DD/MM/YYYY");
      this.date = date;
      const label = "global";
      axios({
        url: "https://coronavirus-19-api.herokuapp.com/countries/",
        method: "GET",
      })
        .then((response) => {
          response.data.forEach((element) => {
            this.countryNames.push(element.country);
          });
          if (country !== null) {
            if (!this.countryNames.includes(country)) {
              alert("País não encontrado");
              return false;
            }
            axios({
              url: `https://coronavirus-19-api.herokuapp.com/countries/${country}`,
              method: "GET",
            })
              .then((resp) => {
                this.arrPositive.push({
                  label: resp.data.country,
                  total: resp.data.cases,
                });
                this.arrDeaths.push({
                  label: resp.data.country,
                  total: resp.data.deaths,
                });
                this.arrRecovered.push({
                  label: resp.data.country,
                  total: resp.data.recovered,
                });
                this.arrTodayDeaths.push({
                  label: resp.data.country,
                  total: resp.data.todayDeaths,
                  backgroundColor: this.getRandomColor()
                })
                this.arrComplex.push({
                  label: resp.data.country,
                  total: resp.data.testsPerOneMillion,
                  backgroundColor: this.getRandomColor()
                })
                this.arrDeathsOne.push({
                  label: resp.data.country,
                  total: resp.data.deathsPerOneMillion,
                  backgroundColor: this.getRandomColor()
                })
              })
              .catch((error) => {
                console.log(error);
              })
              .then(() => {
                this.country = "";
              });
          } else if (country === "" || country === null) {
            axios({
              url: `https://coronavirus-19-api.herokuapp.com/all`,
              method: "GET",
            })
              .then((resp) => {
                this.arrPositive.push({ label, total: resp.data.cases });
                this.arrDeaths.push({ label, total: resp.data.deaths });
                this.arrRecovered.push({ label, total: resp.data.recovered });
              })
              .catch((error) => {
                console.log(error);
              });
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    getRandomColor() {
      var letters = '0123456789ABCDEF';
      var color = '#';
      for (var i = 0; i < 6; i++) {
        color += letters[Math.floor(Math.random() * 16)];
      }
      return color;
    },
    consultarDadosComplexos(country) {
      
      axios({
              url: `https://coronavirus-19-api.herokuapp.com/countries/${country}`,
              method: "GET",
            })
              .then((resp) => {
                this.datasets.push({
                  label: resp.data.country,
                  data: resp.data.cases,
                  ...this.positiveChartColors
                });
                this.datasets.push({
                  label: resp.data.country,
                  data: resp.data.deaths,
                  ...this.deathColors
                });
                this.datasets.push({
                  label: resp.data.country,
                  data: resp.data.recovered,
                  ...this.recoveredColors
                });
                console.log(this.datasets);
              })
              .catch((error) => {
                console.log(error);
              })
              .then(() => {
                this.country = "";
              });
    },
    translateCountryName(country) {
      axios({
          url : "https://translation.googleapis.com/language/translate/v2",
          method: "POST",
          headers: "Bearer gcloud auth application-default print-access-token",
          data: {
            q: country,
            source: "en",
            target: "pt",
            format: "text"
          }})
          .then((response)=>{
            return response
          })
          .catch((error)=> {
            console.log(error);
          })
    }
  },
  watch: {},
};
</script>

<style>
</style>
