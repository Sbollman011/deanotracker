<template>
          <v-card flat>
          <v-form>
    <v-container>
      <v-row>
        <v-col
          cols="12"
          md="4"
        >
          <v-text-field
            v-model="formEvent"
            label="Event"
            placeholder="Weight, Poop, Pee, Right Feed, or Left Feed"
            required
          ></v-text-field>
        </v-col>

        <v-col
          cols="12"
          md="4"
        >
          <v-text-field
            v-model="formTime"
            label="Time"
            placeholder="December 10, 2021 16:06:00"
            required
          ></v-text-field>
        </v-col>

        <v-col
          cols="12"
          md="4"
        >
          <v-text-field
            v-model="formLength"
            label="Length/Weight"
            placeholder="Minutes/Lbs"
          ></v-text-field>
        </v-col>
      </v-row>
      <v-btn @click="submit()" class="success">Submit </v-btn>
    </v-container>
  </v-form>
        </v-card>
</template>

<script>
import App from '../App.vue'
import SleepView from './Sleeps.vue'
import WeightView from './Weight.vue'
import NeedView from './Needs.vue'
import TummyView from './Tummy.vue'

  export default {
    name: 'InputView',
    components: {
    },
    data () {
      return {
        formEvent:"",
        formTime:"",
        formLength:"",
        birthDate: new Date("December 10, 2021 16:06:00"),

      }
    },
  props: {
      weightsProp: null,
      sleepProp: null,
      needsProp: null,
      tummyProp:null,
  },
    created () {

    },
    mounted () {

    },
    methods: {
        submit(){
        let formEvent = this.formEvent;
        let weight = this.formLength;
        let time = new Date(this.formTime);
        
        let duration = parseInt(this.formLength * 60);
        let length = App.methods.secondsToHms(duration);
        let seconds = (parseInt(time-this.birthDate) / 1000);
        let age = App.methods.secondsToHms(seconds);
        if(formEvent == "Weight"){
            let submission = {time: time.toLocaleString(), event:formEvent, realTime: time.getTime(), length:weight, age: age, grams: this.grams};
            this.weightsProp.push(submission);
            WeightView.methods.getWeightSubmit(this.weightsProp);
        }
        else if(formEvent == "Sleep"){
            let submission = {time: time.toLocaleString(), event:formEvent, realTime: time.getTime(), length:length, age: age};
            this.sleepProp.push(submission);
            SleepView.methods.getSleepSubmit(this.sleepProp);
        }
        else if(formEvent == "Tummy"){
            let submission = {time: time.toLocaleString(), event:formEvent, realTime: time.getTime(), length:length, age: age};
            this.tummyProp.push(submission);
            TummyView.methods.getTummySubmit(this.tummyProp);
        }
        else{
            let submission = {time: time.toLocaleString(), event:formEvent, realTime: time.getTime(), length:length, age: age, dayTotal: this.dayTotal};
           this.needsProp.push(submission);
           NeedView.methods.getNeedSubmit(this.needsProp);
        }
        this.formEvent= null;
        this.formTime=null;
        this.formLength=null;
        }
    }
}
</script>

<style>
.buttonD{
  width:40%;
  margin-left:2%;
  background-color:aquamarine;
}
.buttonContainer{
  width:98%;
  text-align:center;
  margin-top:2%;
  margin-bottom:2%;
}
</style>
       
       
