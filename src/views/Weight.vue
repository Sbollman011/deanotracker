<template>
  <v-card flat>
      <v-dialog
      v-model="alert"
      width="90%"
    >
       <v-alert
      prominent
      type="error"
    >
      <v-row align="center">
        <v-col class="grow">
            You are about to delete <br>
            {{deleteMessage}}
        </v-col>
        <v-col class="shrink">
          <v-btn @click = "deleteFromDb()">Delete</v-btn>
        </v-col>
        <v-col class="shrink">
          <v-btn color="success" @click = "toggleAlert()">Cancel Deletion</v-btn>
        </v-col>
      </v-row>
    </v-alert>
 </v-dialog>
    <div class="deantable2">
        <v-card width="90%" font-size="100%">
            <div class="buffer">
                <br>
                <v-text-field
                    padding=5%;
                    v-model="weight"
                    label="Weight"
                    placeholder="Lbs, oz"
                ></v-text-field>
                <v-btn @click="submitWeight()" class="success">Submit </v-btn>
                <br><br>


                <v-dialog
                    v-model="dialog"
                    width="500"
                    >
                <template v-slot:activator="{ on, attrs }">
                    <v-btn
                    color="primary"
                    dark
                    v-bind="attrs"
                    v-on="on"
                    >
                    Manual Entry
                    </v-btn>
                </template>

            <v-card>
                <v-card-title class="text-h5 grey lighten-2">
                Manual Entry
                </v-card-title>        
                <InputView :weightsProp=weights></InputView>
            </v-card>
            </v-dialog>

            <br><br>
            </div>
        </v-card>
         </div> 
    
        <div class="graphContainer">
              <v-expansion-panels>
                <v-expansion-panel>
                <v-expansion-panel-header>
                    Graph
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                    <WeightChart :weightData=weights></WeightChart>
                </v-expansion-panel-content>
                </v-expansion-panel>
             </v-expansion-panels>
        </div>
    

      <div class="deantable">
        <v-card width="95%" font-size="70%">
        <v-simple-table>
    <template v-slot:default>
      <thead>
        <tr>

          <th class="text-left">
            Time
          </th>
          <th class="text-left">
            Weight
          </th>
          <th class="text-left">
            Increase
          </th>
          <th class="text-left">
            Age          
          </th>
          <th class="text-left">
            Actions        
          </th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="event in weights"
          :key="String(event.realTime)"
        >
          <td>{{ event.time}}</td>
          <td>{{ event.length }}</td>
          <td>{{ event.increase }}</td>
          <td>{{ event.age }}</td>
          <td class="center">
            <v-btn x-small color="error" @click = "toggleAlert(event.realTime, event.event + ' ' + event.time + ' ' + event.length + ' '+ event.age)">
                Remove</v-btn>
          </td>
        </tr>
      </tbody>
    </template>
  </v-simple-table>
        </v-card>
      </div>
        </v-card>
</template>

<script>
import App from '../App.vue'

  export default {
    components: {
          InputView: () => import('./Input.vue'),
          WeightChart: () => import('../components/RandomChart.vue')


    },
    data () {
      return {
        weights: [],
        weight:"",
        birthDate: new Date("December 10, 2021 16:06:00"),
        alert: false,
        deleteMessage:"",
        tempRealTime:"",


      }
    },
  props: {

  },
    created () {
      this.getFromWeightDb();
    },
    methods: {
     toggleAlert(rtime,message){
        if(this.alert == true){
            this.alert = false;
        }
        else{
            this.alert = true;
        }
        this.deleteMessage = message;
        this.tempRealTime = rtime;
    },
      getWeightSubmit(submission){
        this.weights = submission;
        this.postToWeightDb();
      },
    deleteFromDb(){
        for(let i = 0; i < this.weights.length;i++){
            if(this.weights[i].realTime == this.tempRealTime){
            this.weights.splice(i,1);
            this.postToWeightDb();
            break;
            }
        }
              this.toggleAlert();
    },
     postToWeightDb(){
      this.weights.sort((a, b) => (a.realTime < b.realTime) ? 1 : -1);
      const weight = { weights:this.weights}

      fetch('https://deanotracker-default-rtdb.firebaseio.com/weights.json', {
        method: 'PUT', // or 'PUT'
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(weight),
      })
      .then(response => response.json())
      .then(data => {
        console.log('Success:', data);
        this.getFromWeightDb();
      })
      .catch((error) => {
        console.error('Error:', error);
      }); 

    },
     getFromWeightDb() {
      fetch('https://deanotracker-default-rtdb.firebaseio.com/weights.json', {
        method: 'GET', // or 'PUT'
        headers: {
          'Content-Type': 'application/json'
        },
      })
      .then(response => response.json())
      .then(data => {
        if(data.weights != undefined){
            this.weights = data.weights;
        }
      })
      .catch((error) => {
        console.error('Error:', error);
      });
    },
    submitWeight(){
        let formEvent = "Weight";
        let weight = this.weight;
        this.weight = "";
        let time= new Date();
        let seconds = (parseInt(time-this.birthDate) / 1000);
        let age = App.methods.secondsToHms(seconds);
        let submission = {time: time.toLocaleString(), event:formEvent, realTime: time.getTime(), length:weight, age: age, grams: this.grams};
        this.weights.push(submission);
        this.postToWeightDb()

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
.graphContainer{
    width:90%;
    margin-left:5%;
    text-align:center;
}
.deantable2{
 text-align:center;
 margin-left:5%;
 padding:5%;
}
</style>