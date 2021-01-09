<template>
  <div>
    <div>
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
    </div>
    <Graphique :title="chartTitle" :values="chartData" />
  </div>
</template>

<script>
import Graphique from "./Graphique.vue";
import staticData from "../data/code-FRA.json";
import _ from "lodash";

const etalabUrl = "https://dashboard.covid19.data.gouv.fr/data/code-FRA.json";

export default {
  name: "Dashboard",
  components: { Graphique },
  data() {
    return {
      rawData: [],
      selectedChartProp: "positiviteHebdomadaire",
    };
  },
  computed: {
    chartTypes() {
      return [
        { label: "Décès", prop: "decesHebdomadaire" },
        { label: "Nouveau cas", prop: "casHebdomadaire" },
        { label: "Positivité", prop: "positiviteHebdomadaire" },
        { label: "Indice de Circulation", prop: "circulationHebdomadaire" },
        { label: "Positivité", prop: "positiviteHebdomadaire" },
        { label: "Hospitalisés", prop: "hospitalises" },
        { label: "Réanimation", prop: "reanimation" },
      ];
    },
    chartData() {
      return this[this.selectedChartProp];
    },
    chartTitle() {
      const chartType = this.chartTypes.find(
        (c) => c.prop === this.selectedChartProp
      );
      return chartType.label;
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
      return this.getDailyValues("testsRealises", () => 0);
    },
    testsPositifs() {
      return this.getDailyValues("testsPositifs", () => 0);
    },
    positivite() {
      return _.zipWith(
        this.testsRealises,
        this.testsPositifs,
        (r, p) => p / r || 0
      );
    },
    positiviteHebdomadaire() {
      return this.getWeeklyData(this.positivite);
    },
    circulationHebdomadaire() {
      return _.zipWith(
        this.casHebdomadaire,
        this.positiviteHebdomadaire,
        (c, p) => c * p
      );
    },
  },
  created() {
    //this.selectedChartType=this.chartTypes[0];
  },
  async mounted() {
    try {
      const response = await fetch(etalabUrl);
      this.rawData = await response.json();
    } catch {
      this.rawData = staticData;
    }
  },
  methods: {
    getDailyValues(propertyName, getDefaultValue) {
      if (!this.sortedData.length) {
        return [];
      }

      const result = [this.sortedData[0][propertyName] || 0];
      getDefaultValue = getDefaultValue || ((i) => result[i - 1]);
      for (let i = 1; i < this.sortedData.length; i++) {
        const dailyValue = this.sortedData[i][propertyName];
        result[i] = dailyValue || getDefaultValue(i);
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
  },
};
</script>
