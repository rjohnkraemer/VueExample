<template>
  <div class="bg">
    <div class="col-md-12 time-header">
      <h1 class="text-center">{{ currTime(now) }}</h1>
    </div>
    <div class="curve-shape-t">
      <img src="/images/curve-top.svg" />
    </div>
    <div class="container-fluid">
      <div class="flex-row flex-wrap row d-flex justify-content-center">
        <div 
          v-for="ninja in data"
          :key="ninja.userGuid"
          class="s-card-container active-students m-4"
        >
            <student-card
              :ninja="ninja"
              :currTime="currTime(new Date(ninja.lastLoginTimestampEpoch))"
              :includeLocation="includeLocation()"
            ></student-card>
        </div>
      </div>
    </div>
    <img src='/images/white-logo.png' class="bottom-right" height="60"/>
  </div>
</template>

<script>
import StudentCard from "./StudentCard.vue";
import axios from "axios";

export default {
  name: "InTheDojo",
  components: {
    StudentCard
  },
  data() {
    return {
      data: [],
      lookup: {},
      now: new Date(),
      minutesCount: -1,
      isLoading: false,
    };
  },
  props: ['locationSlug'],
  methods: {
    includeLocation(){
      return this.locationSlug?true:false;
    },
    dataFilters(){
      this.filterDataByLocationSlug();
      this.filterHideAfter10Minutes();
    },
    filterDataByLocationSlug(){
      if(this.locationSlug){
        this.data = this.data.filter(ninja => ninja.locationSlug === this.locationSlug)
      }
    },
    filterHideAfter10Minutes(){
      this.data = this.data.filter(ninja => this.minutesLeft(ninja) > -10);
    },
    minutesLeft(ninja){
      this.now = new Date();
      if (ninja.minutesLeft === undefined) {
        ninja.minutesLeft = Math.round(
            ninja.scanInSessionLength * 3600000 -
            this.now +
            ninja.lastLoginTimestampEpoch
        ) / 60000;
      }
      return ninja.minutesLeft;
    },
    isWithinWorkingHours(now) {
      let localNow = new Date(now.toLocaleString());
      let hourMinute = localNow.getHours() * 100 + localNow.getMinutes();
      switch (localNow.getDay()) {
        case 0: return false;
        case 6:
          return 845 <= hourMinute && hourMinute <= 1215;
        default:
          return 1245 <= hourMinute && hourMinute <= 1915;
      }
    },
    updateTime() {
      this.now = new Date();
      if (!this.isWithinWorkingHours(this.now)) return;

      if (this.data) {
        this.getNinjas(false);
      }
      if (++(this.minutesCount) == 10) {
        this.minutesCount = 0;
        this.getNinjas(true); // refresh lookup table for ninjas.
      }
    },
    getNinjas(allNinjas) {
      this.isLoading = true;
      let url = `${process.env.VUE_APP_BACKEND_IP_ADDRESS}/api/ninjas`;
      if (!allNinjas) {
        url += '?today=true';
      }

      axios
        .get(url)
        .then((res) => {
          if (allNinjas) {
            this.lookup = {}
            for (let i = 0; i < res.data.length; i++) {
              this.lookup[res.data[i].studentGuid] = res.data[i];
            }
            this.isLoading = false;
            this.getNinjas(false);
          } else {
            console.log(res.data);
            for (let i = 0; i < res.data.length; i++) {
              let ninja = res.data[i];
              let refNinja = this.lookup[ninja.userGuid];
              if (refNinja) {
                ninja.programCode = this.lookup[ninja.userGuid].programCode;
                ninja.isJrWithScratch = this.lookup[ninja.userGuid].isJrWithScratch;
                ninja.locationSlug = this.lookup[ninja.userGuid].locationSlug;
                ninja.locationName = this.lookup[ninja.userGuid].locationName;
              } else {
                ninja.programCode = "CRE";
                ninja.isJrWithScratch = false;
              }

              let thisDate = new Date(ninja.dateCreated);
              ninja.lastLoginTimestamp = thisDate.toLocaleString();
              ninja.lastLoginTimestampEpoch = thisDate.getTime();
            }
            this.data = res.data;
            this.dataFilters();
            console.log(this.data);
          }
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.error(error);
          this.isLoading = false;
        });
    },
    currTime(date) {
      let amPm = 'AM';
      let hours = date.getHours();
      if (hours >= 12) {
        amPm = 'PM'
        if (hours > 12) hours -= 12;
      } else if (hours == 0) {
        hours = 12;
      }
      let hh = ("" + hours).padStart(2, '0');
      let mm = ("" + date.getMinutes()).padStart(2, '0');
      return `${hh}:${mm} ${amPm}`;
    },
  },
  mounted() {
    axios.defaults.headers.common['Authorization'] = 'Bearer ' + this.$store.state.authentication.user.token;

    setInterval(this.updateTime, 60000);
    this.updateTime();
    this.getNinjas(true);
  },
};
</script>

<style src="./InDojoStyling.css" scoped></style>