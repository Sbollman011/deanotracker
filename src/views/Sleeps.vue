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
      <v-btn class='buttonD primary' v-if = "isSleeping == false"  @click='getFromSleepDb("start")'>Start Sleep</v-btn>
      <v-btn class='buttonD primary' v-if = "isSleeping == true" @click='getFromSleepDb("end")'>End Sleep</v-btn>
      </div>
      <div class = 'buttonContainer'>
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
                <InputView :sleepProp=sleepCollection></InputView>
            </v-card>
          </v-dialog>
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
            Duration
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
          v-for="event in sleepCollection"
          :key="String(event.realTime)"
        >
          <td>{{ event.time}}</td>
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
          InputView: () => import('./Input.vue')
    },
    data () {
      return {
        sleepCollection: [],
        time: "",
        start:{},
        end:{},
        isSleeping: false,
        birthdDate: new Date("December 10, 2021 16:06:00"),
        alert: false,
        deleteMessage:"",
        tempRealTime:"",

      }
    },
  props: {
  },
    created () {
      this.getFromSleepDb();

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
        startSleep(){
        if(this.isSleeping ==false){
            this.isSleeping = true;
            this.start = new Date();
            this.postToSleepDb();
       }
        else{
          alert("Someone else has already started a sleep!")
        }
        },
       endSleep(){
            if(this.isSleeping == true){
                this.start= new Date(this.start);
                let endTime = new Date();
                let seconds = (parseInt(this.start - endTime) / 1000) * -1;
                let timeTracker = App.methods.secondsToHms(seconds);
                seconds = (parseInt(endTime-this.birthdDate) / 1000);
                let age = App.methods.secondsToHms(seconds);
                let time = {time: this.start.toLocaleString(), event:"Sleep", realTime: endTime.getTime(), length:timeTracker, age: age}
                this.sleepCollection.push(time);
                this.isSleeping = false;
                this.postToSleepDb();
            }
            else{
                alert("Someone has already ended this sleep.");
            }
        },
      getSleepSubmit(submission){
        this.sleepCollection = submission;
        this.postToSleepDb();
      },
    deleteFromDb(){
        for(let i = 0; i < this.sleepCollection.length;i++){
            if(this.sleepCollection[i].realTime == this.tempRealTime){
            this.sleepCollection.splice(i,1);
            this.postToSleepDb();
            break;
            }
        }
              this.toggleAlert();

    },
     postToSleepDb(){
      this.sleepCollection.sort((a, b) => (a.realTime < b.realTime) ? 1 : -1);
      console.log(this.isSleeping + " ISSLEEPING")
      const sleep = { isSleeping: this.isSleeping, sleepStart: this.start,sleepEnd:this.start, sleeps: this.sleepCollection}

      fetch('https://deanotracker-default-rtdb.firebaseio.com/sleep.json', {
        method: 'PUT', // or 'PUT'
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(sleep),
      })
      .then(response => response.json())
      .then(data => {
        console.log('Success:', data);
        this.getFromSleepDb();
      })
      .catch((error) => {
        console.error('Error:', error);
      }); 
    },
     getFromSleepDb(sleepEvent) {
      fetch('https://deanotracker-default-rtdb.firebaseio.com/sleep.json', {
        method: 'GET', // or 'PUT'
        headers: {
          'Content-Type': 'application/json'
        },
      })
      .then(response => response.json())
      .then(data => {
        if(data.sleeps != undefined){
        this.sleepCollection = data.sleeps;
        }
        if(data.isSleeping != undefined){
          this.isSleeping = data.isSleeping;
        }
        this.start = data.sleepStart;
        this.end = data.sleepEnd;
        if(sleepEvent == "start"){
          this.startSleep();
        }
        else if(sleepEvent == "end"){
          this.endSleep();
        }
      })
      .catch((error) => {
        console.error('Error:', error);
      });
    },
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