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
          <v-btn color="success" @click = "toggleAlert">Cancel Deletion</v-btn>
        </v-col>
      </v-row>
    </v-alert>
 </v-dialog>
      <div class = 'buttonContainer'>
      <v-btn class='buttonD primary' v-if = "isRightFeeding == false && isLeftFeeding == false"  @click='startLeftFeed'>Start Left Feed</v-btn>
      <v-btn class='buttonD primary' v-if = "isRightFeeding == false && isLeftFeeding == false" @click='startRightFeed'>Start Right Feed</v-btn>
      <v-btn class='buttonD primary' v-if = "isLeftFeeding == true" @click='getFromDb("EL")'>End Left Feed</v-btn>
      <v-btn class='buttonD primary' v-if = "isRightFeeding == true" @click='getFromDb("ER")'>End Right Feed</v-btn>
      </div>
      <div class='buttonContainer' >
      <v-btn class='buttonD primary' @click='addPee'>Pee</v-btn>
      <v-btn class='buttonD primary' @click='addPoop'>Poop</v-btn>
      </div>
      <div class='buttonContainer' >
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
                <InputView :needsProp=events></InputView>
            </v-card>
          </v-dialog>
          </div>
          <div class="graphContainer">
           <v-expansion-panels>
                <v-expansion-panel>
                <v-expansion-panel-header>
                    Graph
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                    <NeedsChart :weightData=events></NeedsChart>
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
            Event
          </th>
          <th class="text-left">
            Time
          </th>
          <th class="text-left">
            Length
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
          v-for="event in events"
          :key="String(event.realTime)"
        >
          <td>{{ event.event }}</td>
          <td>{{ event.time}}
          <td>{{ event.length }}</td>
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
        NeedsChart: () => import('../components/NeedsChart.vue')
    },
    data () {
      return {
        events: [],
        startTime: {},
        isRightFeeding: false,
        isLeftFeeding: false,
        isAnyFeeding: false,
        timeTracker: 0,
        alert: false,
        deleteMessage:"",
        tempRealTime:"",
        birthDate: new Date("December 10, 2021 16:06:00"),

      }
    },
  props: {
  },
    created () {
      this.getFromDb();

    },
    mounted () {

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
    getNeedSubmit(submission) {
        this.events = submission;
        this.postToDb();
    },
    addPoop(){
      let d = new Date();
      let seconds = (parseInt(d-this.birthDate ) / 1000);
      let age = App.methods.secondsToHms(seconds);
      let poop = {time: d.toLocaleString(), event:"Poop", realTime: d.getTime(), length: "", age: age}
      this.events.push(poop);
      this.postToDb()

    },    
    addPee(){
      let d = new Date();
       let seconds = (parseInt(d-this.birthDate) / 1000);
      let age = App.methods.secondsToHms(seconds);
      let poop = {time: d.toLocaleString(), event:"Pee", realTime: d.getTime(),length:"", age: age}
      this.events.push(poop);
      this.postToDb()

    },
    startLeftFeed(){
     this.getFromDb("left");
    },
    startRightFeed(){
       this.getFromDb("right");
    },
    endLeftFeed(intime){
      this.getFromDb();
    if(this.isLeftFeeding == true){
      this.startTime=new Date(intime)
      console.log("here");
      let endTime = new Date();
      let seconds = (parseInt(this.startTime - endTime) / 1000) * -1;
      let timeTracker = App.methods.secondsToHms(seconds);
      seconds = (parseInt(endTime-this.birthDate) / 1000);
      let age = App.methods.secondsToHms(seconds);
      console.log(timeTracker)
      let time = {time: this.startTime.toLocaleString(), event:"Left Feed", realTime: endTime.getTime(), length:timeTracker, age: age, dayTotal:this.dayTotal}
      this.events.push(time);
      this.isLeftFeeding = false;
      this.postToDb();
     }
      else{
        alert("Someone has already ended this feeding.");
      }

    },
    endRightFeed(intime){
      this.getFromDb();
      if(this.isRightFeeding == true){
        this.startTime= new Date(intime);
        let endTime = new Date();
        let seconds = (parseInt(this.startTime - endTime) / 1000) * -1;
        let timeTracker = App.methods.secondsToHms(seconds);
        seconds = (parseInt(endTime-this.birthDate) / 1000);
        let age = App.methods.secondsToHms(seconds);
        let time = {time: this.startTime.toLocaleString(), event:"Right Feed", realTime: endTime.getTime(), length:timeTracker, age: age, dayTotal:this.dayTotal}
        this.events.push(time);
        this.isRightFeeding = false;
        this.postToDb()
      }
      else{
        alert("Someone has already ended this feeding.");
      }

    },
    postToDb(){
      this.events.sort((a, b) => (a.realTime < b.realTime) ? 1 : -1);
      const data = {eventList: this.events, isLeftFeeding: this.isLeftFeeding, isRightFeeding:this.isRightFeeding, startTime:this.startTime.toLocaleString()}

      fetch('https://deanotracker-default-rtdb.firebaseio.com/data.json', {
        method: 'PUT', // or 'PUT'
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(data),
      })
      .then(response => response.json())
      .then(data => {
        console.log('Success:', data);
         this.getFromDb();
      })
      .catch((error) => {
        console.error('Error:', error);
      }); 
     
    },
    getFromDb(feed) {
      fetch('https://deanotracker-default-rtdb.firebaseio.com/data.json', {
        method: 'GET', // or 'PUT'
        headers: {
          'Content-Type': 'application/json'
        },
      })
      .then(response => response.json())
      .then(data => {
        console.log('Success:', data);
        this.events = data.eventList;
        if(data.isRightFeeding != undefined){
          this.isRightFeeding = data.isRightFeeding;
        }
        if(data.isLeftFeeding != undefined){
          this.isLeftFeeding = data.isLeftFeeding;
        }
        if(data.startTime != undefined){
          this.startTime = data.startTime;
        }

        for(let i = 0; i < this.events.length;i++){
          if(this.events[i] === null){
            this.events[i].splice(i,1);
          }
        }
        if(feed == "left"){
          if(this.isLeftFeeding ==false && this.isRightFeeding == false){
          this.startTime = new Date();
          this.isLeftFeeding = true;
          this.postToDb()
       }
        else{
          alert("Someone else has already started a feeding!")
        }
      }
      if(feed == "right"){
        if(this.isRightFeeding ==false && this.isLeftFeeding == false){
          this.startTime = new Date();
          this.isRightFeeding = true;
          this.postToDb()
       }
        else{
          alert("Someone else has already started a feeding!")
        }
      }
      if(feed == "ER"){
        this.endRightFeed(data.startTime);
      }
      if(feed == "EL"){
        this.endLeftFeed(data.startTime);
      }
      })
      .catch((error) => {
        console.error('Error:', error);
      });
    },
    deleteFromDb(){
      for(let i = 0; i < this.events.length;i++){
        if(this.events[i].realTime == this.tempRealTime){
          this.events.splice(i,1);
          this.postToDb();
          break;
        }
      }
      this.toggleAlert();
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

</style>