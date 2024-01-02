<template>
<div>
     <v-dialog
      v-model="alertPhoto"
      width="90%"
    >
       <v-alert
      prominent
      type="error"
    >
      <v-row align="center">
        <v-col class="grow">
            You are about to delete <br>
            {{tempMessagePhotoDelete}}
        </v-col>
        <v-col class="shrink">
          <v-btn @click = "deletePhoto()">Delete</v-btn>
        </v-col>
        <v-col class="shrink">
          <v-btn color="success" @click = "toggleAlertPhoto()">Cancel Deletion</v-btn>
        </v-col>
      </v-row>
    </v-alert>
 </v-dialog>
 <v-dialog
      v-model="alertMilestone"
      width="90%"
    >
       <v-alert
      prominent
      type="error"
    >
      <v-row align="center">
        <v-col class="grow">
            You are about to delete <br>
            {{tempMessageMilestoneDelete}}
        </v-col>
        <v-col class="shrink">
          <v-btn @click = "deleteMilestone()">Delete</v-btn>
        </v-col>
        <v-col class="shrink">
          <v-btn color="success" @click = "toggleAlertMilestone()">Cancel Deletion</v-btn>
        </v-col>
      </v-row>
    </v-alert>
 </v-dialog>
                        <div class="right">
                    <v-btn @click="createNewMilestone()">Create Milestone</v-btn>
                    </div>
                         <v-expansion-panels>
                        <v-expansion-panel
                            v-for="(item,i) in milestones"
                            :key="i">
                            <v-expansion-panel-header>
                                {{item.milestone}}
                            </v-expansion-panel-header>

                            <v-expansion-panel-content>
                            <div v-if=!edit>
                                <div class="right">
                                <v-btn @click="toggleEdit()">Edit</v-btn>
                                </div>
                                <v-carousel hide-delimiters>
                                <v-carousel-item
                                v-for="(photo,j) in item.photos"
                                :key="j"
                                reverse-transition="fade-transition"
                                transition="fade-transition"
                                class='center'
                                >
                                  <v-progress-circular
                                        v-if="isUploading"
                                        size= '70'
                                        width='10'
                                        indeterminate
                                        color="primary"
                                        class="down"
                                        ></v-progress-circular>
                                <v-img class = 'left' contain max-width="100%" max-height="100%" :src="photo.photo">
                                    <div class = "left">
                                      <v-btn class="error" x-small @click="toggleAlertPhoto(photo.photoRef, photo.photoRef)">X</v-btn>
                                    </div>
                                </v-img>
                            </v-carousel-item>
                            </v-carousel>

                            <v-card v-if="item==undefined">
                                <v-card-title>Growth:</v-card-title>
                               <v-card-text v-html="thegrowth"></v-card-text>
                                <v-card-title>The New:</v-card-title>
                               <v-card-text v-html="thenew"></v-card-text>
                                <v-card-title>The Good:</v-card-title>
                                <v-card-text v-html="thegood"></v-card-text>
                                <v-card-title>The Bad:</v-card-title>
                                <v-card-text v-html="thebad"> </v-card-text>

                            </v-card>
                             <v-card v-else >
                                <v-card-title>Growth:</v-card-title>
                               <v-card-text v-html="item.growth"></v-card-text>
                                <v-card-title>The New:</v-card-title>
                               <v-card-text v-html="item.new"></v-card-text>
                                <v-card-title >The Good:</v-card-title>
                                <v-card-text v-html="item.good"></v-card-text>
                                <v-card-title>The Bad:</v-card-title>
                                <v-card-text v-html="item.bad"></v-card-text>

                            </v-card>
                        </div>
                             
                <div v-if=edit>
                    <div class="right">
                    <v-btn @click="toggleEdit()">Back</v-btn>
                    </div>
                  <div v-if="item==undefined">
                      <v-file-input
                        multiple
                        v-model="photo"
                        truncate-length="15"
                        ></v-file-input>
                                 <v-col cols="12" md="6">
                                    <v-textarea v-model="themilestone" rows="2" auto-grow  solo name="input-7-4" label="Milestone"></v-textarea>
                                </v-col>
                                 <v-col cols="12" md="6">
                                    <v-textarea v-model="thegrowth" rows="2" auto-grow  solo name="input-7-4" label="Growth"></v-textarea>
                                </v-col>
                                 <v-col cols="12" md="6">
                                    <v-textarea v-model="thenew" rows="2" auto-grow  solo name="input-7-4" label="The New"></v-textarea>
                                </v-col>
                                 <v-col cols="12" md="6">
                                    <v-textarea v-model="thegood" rows="2" auto-grow  solo name="input-7-4" label="The Good"></v-textarea>
                                </v-col>                               
                                <v-col cols="12" md="6">
                                    <v-textarea v-model="thebad" rows="2" auto-grow  solo name="input-7-4" label="The Bad"></v-textarea>
                                </v-col>
                    <div class="left">
                         <v-btn class="error" @click="toggleAlertMilestone(i, themilestone)">Delete</v-btn>
                    </div>
                    <div class="right">
                    <v-btn @click="makeMilestone(i)" class="success">Submit</v-btn>
                    </div>
                     </div>
                    <div v-else>
                      <v-file-input
                        multiple
                        v-model="photo"
                        truncate-length="15"
                        ></v-file-input>
                    <v-col cols="12" md="6">
                        <v-textarea v-model="item.milestone" rows="2" auto-grow  solo name="input-7-4" label="Milestone"></v-textarea>
                    </v-col>
                     <v-col cols="12" md="6">
                        <v-textarea v-model="item.growth" rows="2" auto-grow  solo name="input-7-4" label="Growth"></v-textarea>
                    </v-col>
                        <v-col cols="12" md="6">
                        <v-textarea v-model="item.new" rows="2" auto-grow  solo name="input-7-4" label="The New"></v-textarea>
                    </v-col>
                        <v-col cols="12" md="6">
                        <v-textarea v-model="item.good" rows="2" auto-grow  solo name="input-7-4" label="The Good"></v-textarea>
                    </v-col>                               
                    <v-col cols="12" md="6">
                        <v-textarea v-model="item.bad" rows="2" auto-grow  solo name="input-7-4" label="the Bad"></v-textarea>
                    </v-col>
                   <div class="left">
                         <v-btn class="error" @click="toggleAlertMilestone(i, item.milestone)">Delete</v-btn>
                    </div>
                    <div class="right">
                    <v-btn @click="makeMilestone(i)" class="success">Submit</v-btn>
                    </div>
                     </div>
                 </div>

                            </v-expansion-panel-content>
                        </v-expansion-panel>
                    </v-expansion-panels>

            </div>
</template>

<script>
import deano from '../assets/logo.png';
// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";
import { deleteObject, getDownloadURL, getStorage, ref, uploadBytes } from "firebase/storage";

// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
const firebaseConfig = {
  apiKey: "AIzaSyDXVU0aRIQ5gpcogFtJIIYwQh899TYzrJU",
  authDomain: "deanotracker.firebaseapp.com",
  databaseURL: "https://deanotracker-default-rtdb.firebaseio.com",
  projectId: "deanotracker",
  storageBucket: "deanotracker.appspot.com",
  messagingSenderId: "947488263114",
  appId: "1:947488263114:web:7679bfe78852d10bdbdd33"
};

// Initialize Firebase
const firebaseApp = initializeApp(firebaseConfig);

// Get a reference to the storage service, which is used to create references in your storage bucket
const storage = getStorage(firebaseApp);



  export default {
    components: {
    },
    data () {
      return {
          items: [],
          edit: false,
          milestones: [],
          thegrowth:"",
          thegood:"",
          thebad:"",
          thenew:"",
          themilestone:"New Milestone",
          photo:[],
          image:deano,
          slowPhoto: "",
          isUploading: false,
          alertPhoto: false,
          tempPhotoDelete:"",
          tempMessagePhotoDelete:"",
          alertMilestone: false,
          tempMilestoneDelete:-1,
          tempMessageMilestoneDelete:"",


      }
    },
    props: {
    },
    created () {
        this.getFromMilestoneDb();
    },
    methods: {
        async uploadPhotos(thephoto){
            let tempRef = ref(storage, thephoto.name);
          await  uploadBytes(tempRef, thephoto).then((snapshot) => {
                console.log(snapshot);
            });
        },
       async downloadPhotos(thephoto){
          await  getDownloadURL(ref(storage, thephoto))
            .then((url) => {
                console.log(url);
                this.slowPhoto= url;
            })
            .catch((error) => {
                console.log(error);
                // Handle any errors
            });

        },
        async deletePhoto(){
            let thephoto = this.tempPhotoDelete;
            let tempRef = ref(storage, thephoto);
            for(let i = 0; i < this.milestones.length; i++){
                for(let j =0; j < this.milestones[i].photos.length; j++){
                    if(this.milestones[i].photos[j].photoRef == thephoto){
                        this.milestones[i].photos.splice(j,1);

                    }
                }
            }
            this.tempPhotoDelete = "";
            this.tempMessagePhotoDelete ="";
            this.toggleAlertPhoto();
            this.postToMilestoneDb();
            // Delete the file
           await deleteObject(tempRef).then(() => {
            // File deleted successfully
            }).catch((error) => {
                console.log(error);
            // Uh-oh, an error occurred!
            });
        },
        toggleEdit(){
            if(this.edit == true){
                this.edit = false;
            }
            else{
                this.edit = true;
            }
        },
        toggleAlertPhoto(item, msg){
            if(this.alertPhoto == true){
                this.alertPhoto = false;
                this.tempMessagePhotoDelete = "";
                this.tempPhotoDelete = "";
            }
            else{
                this.alertPhoto = true;
                this.tempMessagePhotoDelete = msg;
                this.tempPhotoDelete = item;
            }
            
        },
        toggleAlertMilestone(item,msg){
            if(this.alertMilestone == true){
                this.alertMilestone = false;
                this.tempMessageMilestoneDelete = "";
                this.tempMilestoneDelete = -1;
            }
            else{
               this.tempMessageMilestoneDelete = msg;
                this.tempMilestoneDelete = item;
                this.alertMilestone = true;
            }
        },
        deleteMilestone(){
            let milestone = this.tempMilestoneDelete;
            for(let i = 0; i < this.milestones.length;i++){
                if(i == milestone){
                    this.milestones.splice(i,1);
                }
            }
            this.tempMilestoneDelete = "";
            this.tempMessageMilestoneDelete ="";
            this.alertMilestone = false;
            this.postToMilestoneDb();
        },
        createNewMilestone() {
            let anarray =[];
            anarray.push(null);
            this.milestones.push({milestone:this.themilestone, growth:this.thegrowth,
                     new:this.thenew, good:this.thegood, bad:this.thebad, photos:anarray, photoRef:""});
                     console.log(this.milestones);
            this.postToMilestoneDb();
        },
       async makeMilestone(index){
            if(this.photo.length > 0){
                for(let i =0; i < this.milestones.length; i++){
                    if(i == index){
                        console.log(i);
                        console.log(this.milestones);
                        if(this.milestones[i].photos == undefined){
                            this.milestones[i].photos = [];
                        }
                        this.toggleEdit();
                        for(let j = 0; j < this.photo.length;j++){
                            this.isUploading = true;
                            if(this.milestones[i].photos[0] == null){
                              await  this.uploadPhotos(this.photo[j]);
                              await this.downloadPhotos(this.photo[j].name);
                                this.milestones[i].photos[0]={photo: this.slowPhoto, photoRef:this.photo[j].name};
                                this.slowPhoto = "";
                            }
                            else{
                                await this.uploadPhotos(this.photo[j]);
                                await this.downloadPhotos(this.photo[j].name);
                                this.milestones[i].photos.push({photo: this.slowPhoto, photoRef:this.photo[j].name});
                                this.slowPhoto = "";


                            }
                        }
                        this.isUploading =false;
                    }
                }

            }
            else{
              this.toggleEdit();

            }
          this.thegrowth = "";
          this.thenew = "";
          this.thegood = "";
          this.thebad = "";
          this.themilestone = "New Milestone";
          this.photo = [];
          this.postToMilestoneDb();

        },
    postToMilestoneDb(){

      fetch('https://deanotracker-default-rtdb.firebaseio.com/milestones.json', {
        method: 'PUT', // or 'PUT'
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(this.milestones),
      })
      .then(response => response.json())
      .then(data => {
        console.log('Success:', data);
        this.getFromMilestoneDb();
      })
      .catch((error) => {
        console.error('Error:', error);
      }); 
    },
     getFromMilestoneDb() {
      fetch('https://deanotracker-default-rtdb.firebaseio.com/milestones.json', {
        method: 'GET', // or 'PUT'
        headers: {
          'Content-Type': 'application/json'
        },
      })
      .then(response => response.json())
      .then(data => {
        if(data != undefined){
        this.milestones = data;
        }
      })
      .catch((error) => {
        console.error('Error:', error);
      });
    }
    }
}
</script>

<style>
.right{
    text-align:right;
    width:100%;
}
.left{
    text-align:left;
    width:100%;
}
.down{
    margin-top:20%;
    position:absolute;
    z-index:2;
}

</style>