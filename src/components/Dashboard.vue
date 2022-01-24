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

const dataGouvUrl =
  "https://www.data.gouv.fr/fr/datasets/r/f335f9ea-86e3-4ffa-9684-93c009d5e617";

export default {
  name: "Dashboard",
  components: { Graphique },
  data() {
    return {
      rawData: [],
      selectedChartProp: "decesHebdomadaire",
      selectedPeriod: "wholeData",
    };
  },
  computed: {
    chartTypes() {
      return [
        { label: "Décès par semaine", prop: "decesHebdomadaire" },
        { label: "Nouveau cas par semaine", prop: "casHebdomadaire" },
        { label: "Positivité", prop: "positivite" },
        { label: "Indice de Circulation", prop: "circulationHebdomadaire" },
        { label: "Hospitalisés", prop: "hospitalises" },
        { label: "Réanimation", prop: "reanimation" },
      ];
    },
    periods() {
      return [
        { label: "Depuis mars 2020", code: "wholeData" },
        { label: "12 derniers mois", code: "last12Months" },
        { label: "6 derniers mois", code: "last6Months" },
        { label: "90 derniers jours", code: "last90days" },
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
        case "last90days":
          return selectedChartData.slice(-90);
        case "last6Months":
          return selectedChartData.slice(-182);
        case "last12Months":
          return selectedChartData.slice(-365);
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
    casJ1() {
      return this.getDailyValues("casJ1");
    },
    casQuotidien() {
      return _.zipWith(
        this.getDailyDelta(this.casConfirmes),
        this.casJ1,
        (a, b) => a || b
      );
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
    positivite() {
      return this.getDailyValues("positivite");
    },
    circulationHebdomadaire() {
      return _.zipWith(this.casHebdomadaire, this.positivite, (c, p) => c * p);
    },
  },
  async mounted() {
    this.rawData = await this.fetchCsvData(dataGouvUrl);
  },
  methods: {
    async fetchCsvData(url) {
      const response = await fetch(url);
      return this.parseCsv(await response.text());
    },
    parseCsv(csvData) {
      return csvData
        .split("\n")
        .filter((l) => l.length > 3)
        .filter((l) => l.startsWith("202"))
        .map((l) => this.parseCsvLine(l));
    },
    parseCsvLine(csvLine) {
      const splittedLine = csvLine.split(",");
      return {
        date: splittedLine[0],
        deces: parseInt(splittedLine[8]),
        casConfirmes: parseInt(splittedLine[13]),
        casJ1: parseInt(splittedLine[14]),
        hospitalises: parseInt(splittedLine[6]),
        reanimation: parseInt(splittedLine[5]),
        positivite: parseFloat(splittedLine[1]) / 100,
      };
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
        result[i] = Math.max(0, dailyValues[i] - (dailyValues[i - 1] || 0));
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
        case "positivite":
          return value && (value * 100).toFixed(1) + "%";
        case "circulationHebdomadaire":
          return value && value.toFixed(0);
        default:
          return value;
      }
    },
  },
};
</script>
