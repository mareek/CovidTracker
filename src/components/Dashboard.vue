<template>
  <div>
    <p>
      <label for="selectGraph">Type de graphique : </label>
      <select id="selectGraph" v-model="selectedChartProp">
        <option
          v-for="chartType in chartTypes"
          :key="chartType.prop"
          :value="chartType.prop"
        >
          {{ chartType.label }}
        </option>
      </select>
    </p>
    <p>
      <label for="selectPeriod">Période affichée : </label>
      <select id="selectPeriod" v-model="selectedPeriod">
        <option
          v-for="period in periods"
          :key="period.code"
          :value="period.code"
        >
          {{ period.label }}
        </option>
      </select>
    </p>
    <p>{{ chartTitle }} au {{ latestDate }} : {{ latestValue }}</p>
    <Graphique :title="chartTitle" :values="chartData" />
  </div>
</template>

<script>
import Graphique from "./Graphique.vue";
import _ from "lodash";

const etalabUrl = "https://dashboard.covid19.data.gouv.fr/data/code-FRA.json";

export default {
  name: "Dashboard",
  components: { Graphique },
  data() {
    return {
      rawData: [],
      selectedChartProp: "decesHebdomadaire",
      selectedPeriod: "last30days",
    };
  },
  computed: {
    chartTypes() {
      return [
        { label: "Décès par semaine", prop: "decesHebdomadaire" },
        { label: "Nouveau cas par semaine", prop: "casHebdomadaire" },
        { label: "Positivité", prop: "positiviteHebdomadaire" },
        { label: "Indice de Circulation", prop: "circulationHebdomadaire" },
        { label: "Hospitalisés", prop: "hospitalises" },
        { label: "Réanimation", prop: "reanimation" },
      ];
    },
    periods() {
      return [
        { label: "Depuis mars 2020", code: "wholeData" },
        { label: "7 derniers jours", code: "last7days" },
        { label: "30 derniers jours", code: "last30days" },
      ];
    },
    chartData() {
      const selectedChartData = this[this.selectedChartProp];
      switch (this.selectedPeriod) {
        case "last7days":
          return selectedChartData.slice(-7);
        case "last30days":
          return selectedChartData.slice(-30);
        default:
          return selectedChartData;
      }
    },
    chartTitle() {
      const chartType = this.chartTypes.find(
        (c) => c.prop === this.selectedChartProp
      );
      return chartType.label;
    },
    hasData() {
      return this.rawData && this.rawData.length;
    },
    latestDate() {
      const sortedData = this.sortedData;
      const latestData = this.hasData && sortedData[sortedData.length - 1];
      return latestData.date;
    },
    latestValue() {
      const chartData = this.chartData;
      const latestValue = this.chartData && chartData[chartData.length - 1];
      return this.formatValue(latestValue);
    },
    sortedData() {
      return _.orderBy(this.rawData, ["date"]);
    },
    deces() {
      return this.getDailyValues("deces");
    },
    decesQuotidien() {
      return this.getDailyDelta(this.deces);
    },
    decesHebdomadaire() {
      return this.getWeeklyData(this.decesQuotidien);
    },
    casConfirmes() {
      return this.getDailyValues("casConfirmes");
    },
    casQuotidien() {
      return this.getDailyDelta(this.casConfirmes);
    },
    casHebdomadaire() {
      return this.getWeeklyData(this.casQuotidien);
    },
    hospitalises() {
      return this.getDailyValues("hospitalises");
    },
    reanimation() {
      return this.getDailyValues("reanimation");
    },
    testsRealises() {
      return this.getDailyValues("testsRealises");
    },
    testsPositifs() {
      return this.getDailyValues("testsPositifs");
    },
    positivite() {
      return _.zipWith(
        this.testsRealises,
        this.testsPositifs,
        (r, p) => p / r || 0
      );
    },
    positiviteHebdomadaire() {
      return this.getWeeklyData(this.positivite).map((d) => (d * 100) / 7);
    },
    circulationHebdomadaire() {
      return _.zipWith(
        this.casHebdomadaire,
        this.positiviteHebdomadaire,
        (c, p) => c * p
      );
    },
  },
  async mounted() {
    try {
      this.rawData = await this.fetchData(etalabUrl);
    } catch {
      this.rawData = await this.fetchData("/data/code-FRA.json");
    }
  },
  methods: {
    async fetchData(url) {
      const response = await fetch(url);
      return await response.json();
    },
    getDailyValues(propertyName) {
      if (!this.sortedData.length) {
        return [];
      }

      const result = [this.sortedData[0][propertyName] || 0];
      for (let i = 1; i < this.sortedData.length; i++) {
        const dailyValue = this.sortedData[i][propertyName];
        result[i] = dailyValue || result[i - 1];
      }
      return result;
    },
    getDailyDelta(dailyValues) {
      const result = [];
      for (let i = 0; i < dailyValues.length; i++) {
        result[i] = dailyValues[i] - (dailyValues[i - 1] || 0);
      }
      return result;
    },
    getWeeklyData(dailyValues) {
      const result = [];
      for (let i = 0; i < dailyValues.length; i++) {
        const lowerBound = Math.max(0, i - 7);
        result.push(_.sum(dailyValues.slice(lowerBound, i) || 0));
      }
      return result;
    },
    formatValue(value) {
      switch (this.selectedChartProp) {
        case "positiviteHebdomadaire":
          return value && value.toFixed(1) + "%";
        case "circulationHebdomadaire":
          return value && value.toFixed(0);
        default:
          return value;
      }
    },
  },
};
</script>
