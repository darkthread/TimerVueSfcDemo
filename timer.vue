<template>
  <div class="timer" v-bind:class="[status]">
    <div class="screen">{{ mm }}:{{ ss }}</div>
    <div class="buttons">
      <button @click="start()">開始</button>
      <button @click="pauseOrResume()">
        {{ status == "P" ? "繼續" : "暫停" }}
      </button>
      <button @click="reset()">重置</button>
    </div>
  </div>
</template>
<script>
export default {
  name: "drk-timer",
  props: ["secs"],
  data() {
    return {
      //W-Waiting, R-Running, P-Paused, O-Timeout
      status: "W",
      lastTime: 0,
      totalTime: 0,
      countdownSecs: 0,
      intHandle: 0,
    };
  },
  computed: {
    remainingSecs() {
      if (!this.countdownSecs) return 0;
      return Math.max(
        0,
        this.countdownSecs - Math.round(this.totalTime / 1000)
      );
    },
    mm() {
      return Math.floor(this.remainingSecs / 60)
        .toString()
        .padStart(2, "0");
    },
    ss() {
      return (this.remainingSecs % 60).toString().padStart(2, "0");
    },
  },
  watch: {
    secs() {
      this.start();
    },
  },
  methods: {
    start() {
      this.reset();
      this.status = "R";
    },
    pauseOrResume() {
      if (this.status == "R") this.status = "P";
      else if (this.status == "P") {
        //resume
        this.lastTime = new Date().getTime();
        this.status = "R";
      }
    },
    reset() {
      this.lastTime = new Date().getTime();
      this.totalTime = 0;
      this.status = 'W';
    },
    tick() {
      if (this.status == "R") {
        const currTime = new Date().getTime();
        this.totalTime += currTime - this.lastTime;
        this.lastTime = currTime;
      }
      if (this.mm + this.ss == 0) {
        this.status = "O";
      }      
    },
  },
  mounted() {
    this.countdownSecs = parseInt(this.secs);
    if (!this.countdownSecs) this.countdownSecs = 180;
    this.intHandle = setInterval(this.tick, 500);
  },
  unmounted() {
    clearInterval(this.intHandle);
  },
};
</script>
<style scoped>
.W .screen { color: white; }
.R .screen { color: lightgreen; }
.P .screen { color: #888; }
.O .screen { color: rgb(255, 7, 7); }
.timer { 
  display: inline-block; margin: 3px;
  background-color: #ddd;
}
.buttons { padding: 6px; text-align: center; }
@font-face {
  /* https://torinak.com/font/7-segment */
  font-family: "7segments"; font-weight: 200; font-style: normal;
  src: url("data:application/font-woff;base64,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")
    format("woff");
}
.screen {
  font-family: "7segments"; font-size: 64px; width: 140px;
  margin: 6px; padding: 6px; background-color: #222;
  border: 2px solid #aaa; border-style: none none solid solid;
}
</style>