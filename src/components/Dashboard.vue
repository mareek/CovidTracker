<template>
  <div>
    <Trend smooth auto-draw :data="casHebdomadaire" />
    <p>{{ rawData }}</p>
  </div>
</template>

<script>
import Trend from "vuetrend";
import staticData from "../data/code-FRA.json";
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
    casHebdomadaire() {
      return [8, 5, 7, 3, 12];
    },
  },
  async mounted() {
    try {
      const response = await fetch(etalabUrl);
      this.rawData = await response.json();
    } catch {
      this.rawData = staticData;
    }
  },
};

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
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
