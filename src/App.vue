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
          center-active
          show-arrows
        >
          <v-tabs-slider color="yellow"></v-tabs-slider>

          <v-tab >
            Needs
          </v-tab>
          <v-tab >
            Weight
          </v-tab>
          <v-tab >
            Sleep
          </v-tab>
          <v-tab >
            Tummy
          </v-tab>
          <v-tab >
            Milestones
          </v-tab>
        </v-tabs>
      </template>
    </v-toolbar>

    <v-tabs-items v-model="tab">
        <v-tab-item>
          <NeedView></NeedView>
        </v-tab-item>
        <v-tab-item>
          <WeightView></WeightView>
        </v-tab-item>
        <v-tab-item>
          <SleepView></SleepView>
        </v-tab-item>
          <v-tab-item>
          <TummyView></TummyView>
        </v-tab-item>
        <v-tab-item>
          <MilestonesView></MilestonesView>
        </v-tab-item>
      </v-tabs-items>
    </v-card>
  </v-app>
</template>

<script>
import WeightView from './views/Weight.vue'
import SleepView from './views/Sleeps.vue'
import NeedView from './views/Needs.vue'
import TummyView from './views/Tummy.vue'
import MilestonesView from './views/Milestones.vue'

export default {
  name: 'App',

  components: {
    SleepView,
    WeightView,
    NeedView,
    MilestonesView,
    TummyView
  },

  data: () => ({
    tab:null,
    grams:"",
  }),
  created() {
    this.getFromDb();
  },
  methods : {
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
.buffer{
  width:80%;
  margin-left:10%;
}
</style>