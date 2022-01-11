<template>
  <div>
    <h1>{{ title }}</h1>
    <Trend smooth auto-draw :min="0" :data="values" />
    <hr />
  </div>
</template>
<script>
import Trend from "vuetrend";

export default {
  name: "Graphique",
  components: { Trend },
  props: {
    title: String,
    values: { type: Array, default: () => [] },
  },
};
</script>