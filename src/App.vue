<template>
  <div id="app">
    <h1>VUE Slider component</h1>
    <Range :config="config" />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import Range from './components/range.vue';

@Component({
  components: {
    Range,
  },
})
export default class App extends Vue {
  private config = {
    min: 5000,
    max: 150000,
    ticks: [6000, 15000, 50000, 100000, 140000],
  };
}
</script>

<style lang="scss">
  h1 {
    color: white;
  }
</style>
