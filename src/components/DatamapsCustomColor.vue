<template>
    <div style="width:80%; margin:auto">
        <vue-datamaps
            :fills="fills"
            :data="mapData"
            :geographyConfig="geographyConfig"
            :localData="world"
            popupTemplate
            @custom:popup="popupTemplate"
        >
          <div slot="hoverinfo" class="hoverinfo" style="white-space: pre-line;">
            {{ popupData }}
          </div>
        </vue-datamaps>
    </div>
</template>

<script>
import { VueDatamaps } from '../../../src'
import { world } from '../../../src/data/index'

export default {
  components: {
    VueDatamaps
  },
  props: {
    fills: Object,
    mapData: Object,
    feature: String
  },
  data: () => ({
    world: world,
    geographyConfig: {
      responsive: true,
      popupOnHover: true,
      highlightOnHover: true,
      borderWidth: 0,
      highlightFillColor: function(data) {
        if (data.fillKey) {
            return data.fillKey;
        }
        return 'rgb(50,50,50)';
      },
    },
    popupData: ''
  }),
  methods: {
    popupTemplate ({ geography, datum }) {
      if (typeof(datum) == 'undefined') {
        this.popupData = `${geography.properties.name}\n ${this.feature} : No Data`
      } else {
        this.popupData = `${geography.properties.name}\n ${this.feature} : ${Math.round(datum.value * 100) / 100}`
      }
    }
  }
}
</script>

<style>

</style>
