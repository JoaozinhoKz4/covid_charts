<template>
  <div class="container">
    <div class="row mt-5">
      <div class="col">
        <h1 class="text-center">COVID-19 DATA</h1>
        <input type="text" v-model="country" class="col-md-2 mt-3" />
        <button @click="consultarDadosCovid(country)" class="ml-2">
          Carregue os dados
        </button>
        <button @click="limparDados" class="ml-2">Limpar consulta</button>
      </div>
    </div>
    <div class="row mt-5" v-if="arrPositive.length > 0">
      <div class="col">
        <h2 class="text-center">Positive</h2>
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
        <h2 class="text-center">Recovered</h2>
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
        <h2 class="text-center">Deaths</h2>
        <line-chart
          v-if="arrDeaths.length > 0"
          :chartData="arrDeaths"
          :options="chartOptions"
          :chartColors="deathColors"
          label="Deaths"
        />
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import moment from "moment";

import LineChart from "./components/LineChart";

export default {
  components: {
    LineChart,
  },
  data() {
    return {
      country: null,
      countryNames: [],
      arrPositive: [],
      positiveChartColors: {
        borderColor: "#077187",
        pointBorderColor: "#0E1428",
        pointBackgroundColor: "#AFD6AC",
        backgroundColor: "#74A57F",
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
      console.log(date);
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
              })
              .catch((error) => {
                console.log(error);
              })
              .then(() => {
                this.country = "";
              });
          } else {
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
  },
  watch: {},
};
</script>

<style>
</style>
