<template>
  <div >
    <div class="s-card">
      <img :src="beltIcon()" height="80px" />
      <div class="s-card-name">
        <span>{{ this.ninja.firstName }} {{ this.ninja.lastName.charAt(0) }}</span>
        <span class="time">{{
          this.currTime
        }}</span>
      </div>
      <div class="s-card-timer">
        <span :class="'time small text-' + variantForMinutesLeft(this.ninja)">{{
          this.minutesLeft
        }}</span>
      </div>
    </div>
    <div class="break"></div>
    <div :class="'s-card-footer bg-' + variantForMinutesLeft(this.ninja)">
        <!-- <strong class="left">{{ this.ninja.scanInSessionLength }}-hour</strong> -->
        <strong class="massive-text right" v-if="!this.includeLocation"> {{this.ninja.locationName.split(",")[0].toUpperCase()}}</strong>
    </div>
  </div>
</template>

<script>
export default{
  props:{
    ninja :{
      type: Object
    },
    currTime:{
      type:String
    },
    includeLocation:{
      type:Boolean
    }
  },
  mounted(){
    this.getMinutesLeft();
    setInterval(this.getMinutesLeft, 15000);
  },
  data(){
    return {
      now: new Date(),
      minutesLeft: this.ninja.minutesLeft,
    }
  },
  methods:{
    variantForMinutesLeft() {
      let minutesLeft = this.minutesLeft;
      let variant = "blue";
      if (minutesLeft == 0) variant = "danger";
      else if (minutesLeft <= 5) variant = "warning";
      return `${variant}`;
    },
    getMinutesLeft(){
      this.now = new Date();
      this.minutesLeft = Math.round(Math.max(0,
          this.ninja.scanInSessionLength * 3600000 -
          this.now +
          this.ninja.lastLoginTimestampEpoch
      ) / 60000);
      return this.minutesLeft;
    },
    beltIcon() {
      let beltName = this.ninja.beltName || "White";
      if (this.ninja.programCode == "JR") {
        if (this.ninja.isJrWithScratch) {
          return "/images/Custom-Icon-JR-Scratch.png";
        } else {
          return "/images/Custom-Icon-JR.png";
        }
      } else {
        return `/images/Rank-Icons-${beltName}.png`;
      }
    },
  }
};
</script>

<style src="./InDojoStyling.css" scoped />