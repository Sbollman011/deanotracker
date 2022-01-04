<template>
<div class="chartSize">
  <div class="small">
    <line-chart :chart-data="datacollection"></line-chart>
        <button @click="fillData()">Update</button>

  </div>
</div>
</template>

<script>
  import LineChart from './LineChart.js'

  export default {
    components: {
      LineChart
    },
    data () {
      return {
        datacollection: null
      }
    },
  props: {
    weightData:[]
  },
    created () {
      this.parseWeights();
      this.fillData()
    },
    mounted () {
      this.parseWeights();
      this.fillData();
    },
    methods: {
      fillData () {
        let weightLabelData = [];
        let weightLabels = [];
        for(let i = 0; i < this.weightData.length;i++){
          weightLabelData.push(this.weightData[i].grams)
          weightLabels.push(this.weightData[i].time);
        }
        weightLabelData.reverse();
        weightLabels.reverse();
        this.datacollection = {
          labels: weightLabels,
          datasets: [
            {
              label: 'Dean\'s Weight Tracker',
              backgroundColor: '#f87979',
              data: weightLabelData
            }
          ]
        }
      },
      parseWeights(){
        for(let i = 0; i < this.weightData.length; i++){
         this.parseWeight(this.weightData[i].length,i);
        }
      },
      parseWeight(weight,index){
        let lbs;
        let oz;
        let lbEnd =0;
        for(let i = 0; i < weight.length; i++){
          if(weight[i] == "l"){
            for(let j = 0; j < i;j++){
                let weightInt = weight[j];
              if(!isNaN(parseInt(weightInt))){
                  if(lbs == undefined){
                    lbs = weight[j];
                  }
                  else{
                  lbs += weight[j];
                  }
                  lbEnd = j+1;
              }
            }
          }
        }
      for(let i = lbEnd; i < weight.length; i++){
          if(weight[i] == "o"){
            for(let j = lbEnd; j < i;j++){
              let weightInt = weight[j];
              if(!isNaN(parseInt(weightInt)) || weight[j] =="."){
                   if(oz == undefined){
                    oz = weight[j];
                  }
                  else{
                  oz += weight[j];
                  }
              }
            }
          }
        }
        this.convertWeightToMetric(lbs,oz, index)
      },
      convertWeightToMetric(lbs, oz, index){
        lbs = parseInt(lbs) * 16;
        oz = parseInt(oz) + lbs;
        let grams = oz/0.035274;
        this.weightData[index].grams = grams;
      },
      getRandomInt () {
        return Math.floor(Math.random() * (50 - 5 + 1)) + 5
      }
    }
  }
</script>

<style>
  .small {
    max-width: 600px;
    text-align:center;
  }
  .chartSize {
    width:95%;
    margin-left:2.5%;
    text-align:center;
  }
</style>