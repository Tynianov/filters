<template>
  <div id="app">
    <div v-for="(filter, index) in visibleFilters" :key="index">
      <p>{{ filter.title }}</p>
      <date-picker
          v-if="filter.type === 'timeframe'"
          v-model="date"
          :range="true"
          :disabled="filter.isDisabled"
          @input="(val) => date = val"
      />
      <v-select
        v-else
        :value="availableFilter[filter.id]"
        :options="filter.values"
        :clearable="false"
        :disabled="filter.isDisabled"
        @input="selectValue($event, filter.id)"
      />
    </div>
  </div>
</template>

<script>

import vSelect from "vue-select";
import DatePicker from 'vue2-datepicker';
import 'vue2-datepicker/index.css';
import "../assets/vue-select.css";
import jsonFile from "../data/fake_data.json"

export default {
  name: 'App',
  components: {
    DatePicker,
    vSelect
  },
  data: () => ({
    availableFilter: {},
    visibleFilters: [],
    update: false,
    date: null
  }),
  computed: {
    filtersFromFile(){
      return jsonFile;
    }
  },
  created() {
    this.filtersFromFile.forEach(item => {
      if (item.type === 'combobox')
        this.availableFilter[item.id] = item.default?
            item.values.filter(val => val.id === item.default)[0]:
            item.values[0];
      else {
        if (item.default)
          this.date = Object.values(item.default).map(d => {
            return new Date(d * 1000)
          });
        this.availableFilter[item.id] = {id: item.id, model: this.date}
      }
    })
    this.updateVisibleFilters();
  },
  methods: {
    selectValue(item, id){
      this.availableFilter[id] = {
        id: item.id,
        label: item.label
      }
      this.update = true;
      this.$forceUpdate();
    },
    checkIfFilterIsEnabled(f){
      f.isDisabled = false;
      if (Array.isArray(f.enabled)){
        f.enabled.forEach(enabled => {
          let entries = Object.entries(enabled)
          entries.forEach(e => {
            if (this.availableFilter[e[0]].id !== e[1])
              f.isDisabled = true;
          })
        })
      }
      else
        f.disabled = f.enabled;
      return f
    },
    checkIfFilterIsVisible(f){
      let isVisible = true;
      if (f.visible === true)
        return true
      else {
        f.visible.forEach(vis => {
          let entries = Object.entries(vis)
          entries.forEach(e => {
            if (this.availableFilter[e[0]].id !== e[1])
              isVisible = false;
          })
        })
        return isVisible;
      }
    },
    updateVisibleFilters(){
      let filters = [];
      this.filtersFromFile.forEach(f => {
        f = this.checkIfFilterIsEnabled(f);

        if (this.checkIfFilterIsVisible(f))
          filters.push(f)
      })
      this.visibleFilters = [...filters];
    }
  },
  watch: {
    update: function (val){
      if (val) {
        this.update = false;
        this.updateVisibleFilters();
      }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
