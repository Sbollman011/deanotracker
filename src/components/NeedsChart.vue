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
        datacollection: null,
        dayTotalArray: []
      }
    },
  props: {
    weightData:[]
  },
    created () {
      this.parseTimes();
      this.fillData()
    },
    mounted () {
      this.parseTimes();
      this.fillData();
    },
    methods: {
    getTimesForDate(date){
        let time=0;
        for(let i = 0; i< this.dayTotalArray.length; i++){
            let date1 = new Date(date);
            let date2 = this.dayTotalArray[i].time;
            if(date1.getDate() == date2){
                   let minutes = this.dayTotalArray[i].dayTotalSeconds / 60;
                   time = this.dayTotalArray[i].dayTotalMinutes + minutes;
            }
        }
        console.log(time);
        return time;

    },
      fillData () {
        let weightLabelData = [];
        let weightLabels = [];
        let tempDateHolder = [];
        for(let i = 0; i < this.weightData.length;i++){
            if(this.weightData[i].event == "Right Feed" || this.weightData[i].event == "Left Feed"){
                let date = new Date(this.weightData[i].time);
                if(!tempDateHolder.includes(date.getDate())){
                    tempDateHolder.push(date.getDate());
                    weightLabelData.push(this.getTimesForDate(this.weightData[i].time));
                   weightLabels.push(this.weightData[i].time);
                }
            }
        }
        weightLabelData.reverse();
        weightLabels.reverse();
        this.datacollection = {
          labels: weightLabels,
          datasets: [
            {
              label: 'Dean\'s Feed Tracker',
              backgroundColor: '#f87979',
              data: weightLabelData
            }
          ]
        }
      },
       parseTimes(){
        let tempDateHolder = [];
        for(let i = 0; i < this.weightData.length;i++){
            if(this.weightData[i].event == "Right Feed" || this.weightData[i].event == "Left Feed"){
                let date = new Date(this.weightData[i].time);
                if(!tempDateHolder.includes(date.getDate())){
                    tempDateHolder.push(date.getDate());
                   this.dayTotalArray.push({time:date.getDate(), dayTotalMinutes:0, dayTotalSeconds:0});
                }
            }
        }
        for(let i = 0; i < this.weightData.length; i++){
         this.parseWeight(this.weightData[i].length,this.weightData[i].time);
        }
      },
      parseWeight(weight,daytime){
        let ms;
        let ss;
        let lbEnd = 0;
        for(let i = 0; i < weight.length; i++){
          if(weight[i] == "M"){
            for(let j = 0; j < i;j++){
                let weightInt = weight[j];
              if(!isNaN(parseInt(weightInt))){
                  if(ms == undefined){
                    ms = weight[j];
                  }
                  else{
                   ms += weight[j];
                  }
                  lbEnd = j+1;
              }
            }
          }
        }
      for(let i = lbEnd; i < weight.length; i++){
          if(weight[i] == "S"){
            for(let j = lbEnd; j < i;j++){
              let weightInt = weight[j];
              if(!isNaN(parseInt(weightInt)) || weight[j] =="."){
                   if(ss == undefined){
                    ss = weight[j];
                  }
                  else{
                  ss += weight[j];
                  }
              }
            }
          }
        }
        this.convertWeightToMetric(ms,ss, daytime)
      },
      convertWeightToMetric(ms, ss,daytime){
        ms = parseInt(ms);
        ss = parseInt(ss);
        let date1 = new Date(daytime);
        for(let i = 0; i <this.weightData.length;i++ ){
            let date2 = new Date(this.weightData[i].time);
            if(date1.getDate() == date2.getDate()){
                for(let j = 0; j <this.dayTotalArray.length;j++ ){
                    let date3 = this.dayTotalArray[j].time;
                    if(date3 == date2.getDate()){
                        if(!isNaN(ms)){
                            this.dayTotalArray[j].dayTotalMinutes += ms;
                        }
                        if(!isNaN(ss)){
                            this.dayTotalArray[j].dayTotalSeconds += ss;
                        }
                    }
                }
            }
        }
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