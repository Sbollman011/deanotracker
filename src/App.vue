<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
    <h1>Deano Tracker</h1>
    </v-app-bar>

    <v-main>
      <div class = 'buttonContainer'>
      <v-btn class='buttonD' v-if = "isRightFeeding == false && isLeftFeeding == false"  @click='startLeftFeed'>Start Left Feed</v-btn>
      <v-btn class='buttonD' v-if = "isRightFeeding == false && isLeftFeeding == false" @click='startRightFeed'>Start Right Feed</v-btn>
      <v-btn class='buttonD' v-if = "isLeftFeeding == true" @click='endLeftFeed'>End Left Feed</v-btn>
      <v-btn class='buttonD' v-if = "isRightFeeding == true" @click='endRightFeed'>End Right Feed</v-btn>
      </div>
      <div class='buttonContainer' >
      <v-btn class='buttonD' @click='addPee'>Pee</v-btn>
      <v-btn class='buttonD' @click='addPoop'>Poop</v-btn>
      </div>
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
          v-for="event in events.slice().reverse()"
          :key="String(event.realTime)"
        >
          <td>{{ event.event }}</td>
          <td>{{ event.time }}</td>
          <td>{{ event.length }}</td>
          <td>{{ event.age }}</td>
          <td><v-btn color="error" @click = "deleteFromDb(event.realTime)">Remove</v-btn></td>
        </tr>
      </tbody>
    </template>
  </v-simple-table>
    </v-main>
  </v-app>
</template>

<script>

export default {
  name: 'App',

  components: {
  },

  data: () => ({
    events: [],
    startTime: {},
    isRightFeeding: false,
    isLeftFeeding: false,
    isAnyFeeding: false,
    timeTracker: 0,
    lastDay: "",
    lastWeek:"",
    birthdaDate: new Date("December 10, 2021 16:06:00")
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
      this.startTime = new Date();
      this.isLeftFeeding = true;
    },
    startRightFeed(){
      this.startTime = new Date();
      this.isRightFeeding = true;
    },
    endLeftFeed(){
     let endTime = new Date();
      let seconds = (parseInt(this.startTime - endTime) / 1000) * -1;
      let timeTracker = this.secondsToHms(seconds);
      seconds = (parseInt(endTime-this.birthdaDate) / 1000);
      let age = this.secondsToHms(seconds);
      let time = {time: endTime.toLocaleString(), event:"Left Feed", realTime: endTime.getTime(), length:timeTracker, age: age}
      this.events.push(time);
      this.isLeftFeeding = false;
      this.postToDb()


    },
    endRightFeed(){
      let endTime = new Date();
      let seconds = (parseInt(this.startTime - endTime) / 1000) * -1;
      let timeTracker = this.secondsToHms(seconds);
      seconds = (parseInt(endTime-this.birthdaDate) / 1000);
      let age = this.secondsToHms(seconds);
      let time = {time: endTime.toLocaleString(), event:"Right Feed", realTime: endTime.getTime(), length:timeTracker, age: age}
      this.events.push(time);
      this.isRightFeeding = false;
      this.postToDb()

    },
    secondsToHms(d) {
    d = Number(d);
    var w = Math.floor(d / 3600 / 24 / 7)
    var da = Math.floor(d / 3600 / 24 % 7);
    var h = Math.floor(d / 3600 % 24);
    var m = Math.floor(d % 3600 / 60);
    var s = Math.floor(d % 3600 % 60);

    var wDisplay = w > 0 ? w + (w == 1 ? " week, " : " weeks, ") : "";
    var dDisplay = da > 0 ? da + (da == 1 ? " day, " : " days, ") : "";
    var hDisplay = h > 0 ? h + (h == 1 ? " hour, " : " hours, ") : "";
    var mDisplay = m > 0 ? m + (m == 1 ? " minute, " : " minutes, ") : "";
    var sDisplay = s >= 0 ? s + (s == 1 ? " second" : " seconds") : "";
    return wDisplay + dDisplay + hDisplay + mDisplay + sDisplay; 
    },
    postToDb(){
      const data = { eventList: this.events }

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
      })
      .catch((error) => {
        console.error('Error:', error);
      }); 
    },
    getFromDb() {
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
        console.log(this.events);
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
    }
  }
};
</script>
<style>
.buttonD{
  width:20%;
  margin-left:2%;
}
.buttonContainer{
  width:98%;
  text-align:center;
  margin-top:2%;
}

</style>