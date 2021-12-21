<template>
  <v-app>
   <v-card>
    <v-toolbar
      color="cyan"
      dark
      flat
    >
      <v-toolbar-title>Deano Tracker</v-toolbar-title>

      <template v-slot:extension>
        <v-tabs
          v-model="tab"
          align-with-title
        >
          <v-tabs-slider color="yellow"></v-tabs-slider>

          <v-tab >
            Tracker
          </v-tab>
          <v-tab >
            Graph
          </v-tab>
          <v-tab >
            Input
          </v-tab>
        </v-tabs>
      </template>
    </v-toolbar>

    <v-tabs-items v-model="tab">
      <v-tab-item
      >
      <v-card flat>
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
            <v-btn x-small color="error" @click = "deleteFromDb(event.realTime)">Remove</v-btn>
          <v-btn x-small color="success" @click = "edit(event.realTime)">Edit</v-btn></td>
        </tr>
      </tbody>
    </template>
  </v-simple-table>
  </v-card>
  </div>
      </v-card>
      </v-tab-item>



       <v-tab-item>
        <v-card flat>
          <v-card-text >Test</v-card-text>
        </v-card>
      </v-tab-item>



      <v-tab-item>
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
            placeholder="Poop, Pee, Right Feed, or Left Feed"
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
            label="Length"
            placeholder="Minutes"
          ></v-text-field>
        </v-col>
      </v-row>
      <v-btn @click="submit" class="success">Submit </v-btn>
    </v-container>
  </v-form>
        </v-card>
      </v-tab-item>



    </v-tabs-items>

    

  </v-card>

   
  </v-app>
</template>

<script>

export default {
  name: 'App',

  components: {
  },

  data: () => ({
    tab:null,
    events: [],
    startTime: {},
    isRightFeeding: false,
    isLeftFeeding: false,
    isAnyFeeding: false,
    timeTracker: 0,
    lastDay: "",
    lastWeek:"",
    birthdaDate: new Date("December 10, 2021 16:06:00"),
    formEvent:"",
    formTime:"",
    formLength:"",
  }),
  created() {
    this.getFromDb();
  },
  methods : {
    addPoop(){
      let d = new Date();
      let seconds = (parseInt(d-this.birthdaDate ) / 1000);
      let age = this.secondsToHms(seconds);
      let poop = {time: d.toLocaleString(), event:"Poop", realTime: d.getTime(), length: "", age: age}
      this.events.push(poop);
      this.postToDb()

    },    
    addPee(){
      let d = new Date();
       let seconds = (parseInt(d-this.birthdaDate) / 1000);
      let age = this.secondsToHms(seconds);
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
    if(this.isLeftFeeding == true){
      this.startTime=new Date(intime)
      let endTime = new Date();
      let seconds = (parseInt(this.startTime - endTime) / 1000) * -1;
      let timeTracker = this.secondsToHms(seconds);
      seconds = (parseInt(endTime-this.birthdaDate) / 1000);
      let age = this.secondsToHms(seconds);
      console.log(timeTracker)
      let time = {time: this.startTime.toLocaleString(), event:"Left Feed", realTime: endTime.getTime(), length:timeTracker, age: age}
      this.events.push(time);
      this.isLeftFeeding = false;
      this.postToDb();
     }
      else{
        alert("Someone has already ended this feeding.");
      }

    },
    endRightFeed(intime){
      if(this.isRightFeeding == true){
        this.startTime= new Date(intime);
        let endTime = new Date();
        let seconds = (parseInt(this.startTime - endTime) / 1000) * -1;
        let timeTracker = this.secondsToHms(seconds);
        seconds = (parseInt(endTime-this.birthdaDate) / 1000);
        let age = this.secondsToHms(seconds);
        let time = {time: this.startTime.toLocaleString(), event:"Right Feed", realTime: endTime.getTime(), length:timeTracker, age: age}
        this.events.push(time);
        this.isRightFeeding = false;
        this.postToDb()
      }
      else{
        alert("Someone has already ended this feeding.");
      }

    },
    secondsToHms(d) {
    d = Number(d);
    var w = Math.floor(d / 3600 / 24 / 7)
    var da = Math.floor(d / 3600 / 24 % 7);
    var h = Math.floor(d / 3600 % 24);
    var m = Math.floor(d % 3600 / 60);
    var s = Math.floor(d % 3600 % 60);

    var wDisplay = w > 0 ? w + (w == 1 ? "Week " : "Weeks ") : "";
    var dDisplay = da > 0 ? da + (da == 1 ? "Day " : "Days ") : "";
    var hDisplay = h > 0 ? h + (h == 1 ? "H " : "H ") : "";
    var mDisplay = m > 0 ? m + (m == 1 ? "M " : "M, ") : "";
    var sDisplay = s >= 0 ? s + (s == 1 ? "S" : "S") : "";
    return wDisplay + dDisplay + hDisplay + mDisplay + sDisplay
    },
    postToDb(){
      this.events.sort((a, b) => (a.realTime < b.realTime) ? 1 : -1);
      const data = { eventList: this.events, isLeftFeeding: this.isLeftFeeding, isRightFeeding:this.isRightFeeding, startTime:this.startTime.toLocaleString()}

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
    deleteFromDb(key){
      for(let i = 0; i < this.events.length;i++){
        if(this.events[i].realTime == key){
          this.events.splice(i,1);
          this.postToDb();
          break;
        }
      }
    },
    filter(key){
        for(let i = 0; i < this.events.length;i++){
        if(this.events[i].realTime == key){
          //this.events.splice(i,1);
          //this.postToDb();
          break;
        }
      }
    },
    submit(){
      let formEvent = this.formEvent;
      let time = new Date(this.formTime);
      let duration = parseInt(this.formLength * 60);
      console.log(duration);
      let length = this.secondsToHms(duration);
      console.log(length);
      let seconds = (parseInt(time-this.birthdaDate) / 1000);
      let age = this.secondsToHms(seconds);
      let submission = {time: time.toLocaleString(), event:formEvent, realTime: time.getTime(), length:length, age: age};
      this.events.push(submission);
      this.postToDb()
      this.formEvent= null;
      this.formTime=null;
      this.formLength=null;
    }
  }
};
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
.deantable{
text-align:left;
margin-left:5%;
}
.theme--light.v-data-table > .v-data-table__wrapper > table > thead > tr:last-child > th {
    font-size: 60% !important;

 }
 .v-data-table > .v-data-table__wrapper > table > tbody > tr > td, .v-data-table > .v-data-table__wrapper > table > thead > tr > td, .v-data-table > .v-data-table__wrapper > table > tfoot > tr > td {
    font-size: 50% !important;
}
.v-btn__content{
  font-size:60%;
}
.v-btn:not(.v-btn--round).v-size--x-small{
  margin-top:2% !important;
  margin-bottom: 2% !important;
}
.center{
  text-align:center;
}
</style>