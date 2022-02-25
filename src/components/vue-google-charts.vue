<template>
  <div>
    <h2>Vue Js Google line Charts Demo</h2>
    <GChart
      type="LineChart"
      :options="options"
      :data="collectionData"
    />    
  </div>
</template>
 
<script>
import { GChart } from "vue-google-charts";
export default {
  name: "App",
  components: {
    GChart
  },
  props: {
      astroGraphdata: Array
  },
  data() {
    return {
      collectionData: this.astroGraphdata,
      options: {
        
        width: 1000,
        height: 400
      }
    };
  },
  created(){
      console.log("AstroidGraph",this.astroGraphdata);

  }
};
</script>