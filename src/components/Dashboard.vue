<template>
  <div>
    <h1>Décès hebdomadaires</h1>
    <Trend smooth auto-draw :data="decesHebdomadaire" />
    <hr/>
  </div>
</template>

<script>
import Trend from "vuetrend";
import staticData from "../data/code-FRA.json";
import _ from "lodash";

const etalabUrl = "https://dashboard.covid19.data.gouv.fr/data/code-FRA.json";

export default {
  name: "Dashboard",
  components: { Trend },
  data() {
    return {
      rawData: [],
    };
  },
  computed: {
    sortedData() {
      return _.orderBy(this.rawData, ["date"]);
    },
    deces() {
      return this.getDailyValues("deces");
    },
    casConfirmes() {
      return this.getDailyValues("casConfirmes");
    },
    hospitalises() {
      return this.getDailyValues("hospitalises");
    },
    decesQuotidien() {
      return this.getDailyDelta(this.deces);
    },
    decesHebdomadaire() {
      return this.getWeeklyData(this.decesQuotidien);
    },
    /* raw data
{
  "casConfirmes": 10995,
  "deces": 372,
  "reanimation": 1122,
  "hospitalises": 4461,
  "gueris": 1300,
  "date": "2020-03-19",
  "code": "FRA",
  "nom": "France",
  "testsRealises": 2098,
  "testsPositifs": 528,
  "nouvellesHospitalisations": 2229,
  "nouvellesReanimations": 438,
  "tauxIncidence": 5.57082271463596,
  "tauxOccupationRea": 19.8
}
*/
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
      for (let i = 7; i < dailyValues.length; i++) {
        result.push(_.sum(dailyValues.slice(i - 7, i) || 0));
      }
      return result;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
