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
      <v-btn class='buttonD primary' v-if = "istummying == false"  @click='getFromtummyDb("start")'>Start tummy</v-btn>
      <v-btn class='buttonD primary' v-if = "istummying == true" @click='getFromtummyDb("end")'>End tummy</v-btn>
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
                <InputView :tummyProp=tummyCollection></InputView>
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
          v-for="event in tummyCollection"
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
        tummyCollection: [],
        time: "",
        start:{},
        end:{},
        istummying: false,
        birthdDate: new Date("December 10, 2021 16:06:00"),
        alert: false,
        deleteMessage:"",
        tempRealTime:"",

      }
    },
  props: {
  },
    created () {
      this.getFromtummyDb();

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
        starttummy(){
        if(this.istummying ==false){
            this.istummying = true;
            this.start = new Date();
            this.postTotummyDb();
       }
        else{
          alert("Someone else has already started a tummy!")
        }
        },
       endtummy(){
            if(this.istummying == true){
                this.start= new Date(this.start);
                let endTime = new Date();
                let seconds = (parseInt(this.start - endTime) / 1000) * -1;
                let timeTracker = App.methods.secondsToHms(seconds);
                seconds = (parseInt(endTime-this.birthdDate) / 1000);
                let age = App.methods.secondsToHms(seconds);
                let time = {time: this.start.toLocaleString(), event:"Tummy", realTime: endTime.getTime(), length:timeTracker, age: age}
                this.tummyCollection.push(time);
                this.istummying = false;
                this.postTotummyDb();
            }
            else{
                alert("Someone has already ended this tummy.");
            }
        },
      getTummySubmit(submission){
        this.tummyCollection = submission;
        this.postTotummyDb();
      },
    deleteFromDb(){
        for(let i = 0; i < this.tummyCollection.length;i++){
            if(this.tummyCollection[i].realTime == this.tempRealTime){
            this.tummyCollection.splice(i,1);
            this.postTotummyDb();
            break;
            }
        }
              this.toggleAlert();

    },
     postTotummyDb(){
      this.tummyCollection.sort((a, b) => (a.realTime < b.realTime) ? 1 : -1);
      console.log(this.istummying + " IStummyING")
      const tummy = { istummying: this.istummying, tummyStart: this.start,tummyEnd:this.start, tummys: this.tummyCollection}

      fetch('https://deanotracker-default-rtdb.firebaseio.com/tummy.json', {
        method: 'PUT', // or 'PUT'
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(tummy),
      })
      .then(response => response.json())
      .then(data => {
        console.log('Success:', data);
        this.getFromtummyDb();
      })
      .catch((error) => {
        console.error('Error:', error);
      }); 
    },
     getFromtummyDb(tummyEvent) {
      fetch('https://deanotracker-default-rtdb.firebaseio.com/tummy.json', {
        method: 'GET', // or 'PUT'
        headers: {
          'Content-Type': 'application/json'
        },
      })
      .then(response => response.json())
      .then(data => {
        if(data.tummys != undefined){
        this.tummyCollection = data.tummys;
        }
        if(data.istummying != undefined){
          this.istummying = data.istummying;
        }
        this.start = data.tummyStart;
        this.end = data.tummyEnd;
        if(tummyEvent == "start"){
          this.starttummy();
        }
        else if(tummyEvent == "end"){
          this.endtummy();
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