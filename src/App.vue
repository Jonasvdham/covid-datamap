<template>
  <div id='app'>
    <div class='header'>
    <center>
      <h2>
        <span v-html="labeldict[feature]"></span>
      </h2>
      <select v-model='feature'>
        <option value='new_cases_smoothed_per_million'>New cases per million</option>
        <option value='total_cases_per_million'>Total cases per million</option>
        <option value='new_deaths_smoothed'>New deaths</option>
        <option value='people_vaccinated_per_hundred'>Percentage people vaccinated</option>
      </select>
      </center>
      </div>
    <div style="width:80%; margin:auto">
    <VueSlideBar
      v-model="slider.value"
      :data="slider.data"
      :range="slider.range"
      :labelStyle="{ color: '#000000', backgroundColor: '#555444' }"
      :processStyle="{ backgroundColor: '#fd0' }"
      @callbackRange="callbackRange"
    />
    <datamaps-custom-color :fills='this.fills' :mapData='mapData' :feature='labeldict[feature]'/>
    <center>
    <div id='grad'>
      <div style="float: left; width: 15%; height: auto; line-height: 40px; vertical-align: middle; font-family: Verdana; font-size: 10px; overflow: hidden; text-align: center; color: black;">
        <b><span v-html="min"></span></b>
      </div>
      <div style="float: right; width: 15%; height: auto; line-height: 40px; vertical-align: middle; font-family: Verdana; font-size: 10px; overflow: hidden; text-align: center; color: black;">
        <b><span v-html="max"></span></b>
      </div>
    </div>
    </center>
    </div>
  </div>
</template>

<script>
import DatamapsCustomColor from '@/components/DatamapsCustomColor'
import { csv } from 'd3-fetch'
import * as d3 from 'd3v4'
import { interpolateReds , interpolateBlues } from 'd3-scale-chromatic'
import VueSlideBar from 'vue-slide-bar'

var r = document.querySelector(':root');

export default {
  data: () => ({
    min: 0,
    max: 0,
    min_color: "",
    max_color: "",
    feature: 'new_cases_smoothed_per_million',
    date: '2020-01',
    rawData: [],
    fills: {
      defaultFill: 'rgb(50,50,50)'
    },
    labeldict: {
      "new_cases_smoothed_per_million" : "Daily COVID-19 cases per million",
      "total_cases_per_million": "Total COVID-19 cases per million",
      "new_deaths_smoothed": "Daily deaths resulting from COVID-19",
      "people_vaccinated_per_hundred": "Percentage of population vaccinated"
    },
    indexes: {
      '2020-01': 0,
      '2020-02': 224,
      '2020-03': 448,
      '2020-04': 672,
      '2020-05': 896,
      '2020-06': 1120,
      '2020-07': 1344,
      '2020-08': 1568,
      '2020-09': 1792,
      '2020-10': 2016,
      '2020-11': 2240,
      '2020-12': 2464,
      '2021-01': 2688,
      '2021-02': 2912,
      '2021-03': 3136,
      '2021-04': 3360,
      '2021-05': 3584,
      '2021-06': 3808,
      '2021-07': 4032,
      '2021-08': 4256,
      '2021-09': 4480,
      '2021-10': 4704,
      '2021-11': 4928,
      '2021-12': 5152,
      '2022-01': 5376,
      '2022-02': 5600,
      '2022-03': 5824
    },
    rangeValue: {},
    slider: {
      value: 45,
      data: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec', 'Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec', 'Jan', 'Feb'],
      range: [
        {
          label: "2020-01"
        },
        {
          label: "2020-02",
          isHide: true
        },
        {
          label: "2020-03"
        },
        {
          label: "2020-04",
          isHide: true
        },
        {
          label: "2020-05"
        },
        {
          label: "2020-06",
          isHide: true
        },
        {
          label: "2020-07"
        },
        {
          label: "2020-08",
          isHide: true
        },
        {
          label: "2020-09",
        },
        {
          label: "2020-10",
          isHide: true          
        },
        {
          label: "2020-11",
        },
        {
          label: "2020-12",
          isHide: true          
        },
        {
          label: "2021-01"
        },
        {
          label: "2021-02",
          isHide: true
        },
        {
          label: "2021-03"
        },
        {
          label: "2021-04",
          isHide: true
        },
        {
          label: "2021-05"
        },
        {
          label: "2021-06",
          isHide: true
        },
        {
          label: "2021-07"
        },
        {
          label: "2021-08",
          isHide: true
        },
        {
          label: "2021-09",
        },
        {
          label: "2021-10",
          isHide: true
        },
        {
          label: "2021-11"
        },
        {
          label: "2021-12",
          isHide: true
        },
        {
          label: "2022-01"
        },
        {
          label: "2022-02",
          isHide: true
        }        
      ]
    }
  }),
  components: {
    VueSlideBar,
    DatamapsCustomColor
  },
  mounted () {
    csv('data/monthly_covid_data.csv').then(data => {
      this.rawData = data
    })
  },
  computed: {
    mapData () {
      if (!this.rawData.length) {
        return {};
      }
      else {
        let data = this.filterData()
        return data
      }
    },
  },
  methods: {
    filterData () {
      var result = {}
      let i = this.indexes[this.rangeValue.label]
      let j = i + 224

      this.max = this.rawData[i][this.feature]
      this.min = this.rawData[i][this.feature]
      for (; i < j; i++) {
        if (this.rawData[i][this.feature] > this.max) {
          this.max = this.rawData[i][this.feature]
        }
        if (this.rawData[i][this.feature] < this.min && this.rawData[i][this.feature] >= 0) {
          this.min = this.rawData[i][this.feature]
        }
        result[this.rawData[i].iso_code] = { 
          fillKey: this.rawData[i].iso_code,
          value: this.rawData[i][this.feature],
        }
        i++;
      }
      this.generateColor(result, this.min, this.max)
      this.setLegendColor()
      this.min = Math.round(this.min * 100) / 100
      this.max = Math.round(this.max * 100) / 100
      console.log(this.min, this.max)
      return result
    },
    generateColor (result, min, max) {
      if (this.feature == 'people_vaccinated_per_hundred'){
        for (let [key, item] of Object.entries(result)) {
          this.fills[key] = interpolateBlues(item.value/max)
        }
        this.min_color = interpolateBlues(min/max)
        this.max_color = interpolateBlues(1)
      } else {
        for (let [key, item] of Object.entries(result)) {
          this.fills[key] = interpolateReds(item.value/max)
        }
        this.min_color = interpolateReds(min/max)
        this.max_color = interpolateReds(1)
      }
    },
    callbackRange (val) {
      this.rangeValue = val;
    },
    setLegendColor () {
    // Set the value of variable --blue to another value (in this case "lightblue")
    r.style.setProperty('--min_color', this.min_color);
    r.style.setProperty('--max_color', this.max_color);
    },
  }
}
</script>

<style>
:root {
  --min_color: rgb(255,245,240);
  --max_color: rgb(103,0,13);
}
.header {
  padding: 20px;
  text-align: center;
  background: #1abc9c;
  color: white;
  font-size: 20px;
}
#grad {
  height: 40px;
  background-color: red; /* For browsers that do not support gradients */
  background-image: linear-gradient(to right, var(--min_color), var(--max_color));
  padding: 0;
  margin: 0;
  border-top-left-radius: 10px;
  border-bottom-left-radius: 10px;
  border-top-right-radius: 10px;
  border-bottom-right-radius: 10px; 
}
.body {
  background-color: rgb(50, 50, 50);
}

</style>