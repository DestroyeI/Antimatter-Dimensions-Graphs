<script>
import PrimaryToggleButton from "@/components/PrimaryToggleButton";
import SliderComponent from "@/components/SliderComponent";
import TachyonParticles from "../time-dilation/TachyonParticles.vue";

export default {
  name: "GraphsTab",
  components: {
    PrimaryToggleButton,
    SliderComponent
  },
  data() {
    return {
      boot: 0,
      cnv: undefined,
      ctx: undefined,
      resolution: 1000,
      intervalsSlider: 40,
      trackedTimeSlider: 1,
      trackedTimes: [1, 10, 30, 60, 120, 300, 600],
      trackedTime: 10,
      trackedTimeDisp: "10s",
      predictive: false,
      logarithmic: true,
      shown: {
        antimatter: true,
        infinities: true
      },
      graphs: {
        antimatter: {
          label: "Antimatter",
          color: "--color-antimatter",
          prestiges: ["boosts", "galaxies", "crunches"],
          unlocked() {
            return true;
          }
        },
        infinityPoints: {
          label: "IP",
          color: "--color-infinity",
          prestiges: [],
          unlocked() {
            return PlayerProgress.infinityUnlocked();
          }
        }
      },
      prestigeColors: {
        boosts: "--color-good-dark",
        galaxies: "--color-good",
        crunches: "--color-infinity"
      }
    };
  },
  computed: {
    sliderPropsIntervalsSlider() {
      return {
        min: 10,
        max: 100,
        interval: 5,
        width: "100%",
        tooltip: false
      };
    },
    sliderPropsTrackedTimeSlider() {
      return {
        min: 0,
        max: 6,
        interval: 1,
        width: "100%",
        tooltip: false
      };
    }
  },
  watch: {
    predictive(newValue) {
      player.graphOptions.predictive = newValue;
    },
    logarithmic(newValue) {
      player.graphOptions.logarithmic = newValue;
    }
  },
  mounted() {
    const options = player.graphOptions;
    this.intervalsSlider = options.intervals;
    this.trackedTimeSlider = this.trackedTimes.indexOf(options.trackedTime);
    this.predictive = options.predictive;
    this.logarithmic = options.logarithmic;
    this.shown = { ...options.shown };
    
    this.cnv = this.$refs.cnvGraph;
    this.cnv.width = 3 * this.resolution;
    this.cnv.height = this.resolution;
    this.ctx = this.cnv.getContext("2d");
  },
  methods: {
    update() {
      if (this.cnv == undefined) {
        console.error("Graph canvas not initialized; Update too fast");
        return;
      }

      this.ctx.clearRect(0, 0, 3 * this.resolution, this.resolution);
      this.ctx.lineWidth = 5;
      
      let intervalTime = ((player.graphOptions.trackedTime * 1000) / player.graphOptions.intervals);
      let tOff = (Date.now() - player.records.gameCreatedTime) % intervalTime;
      Object.keys(this.graphs).forEach((graph) => {
        if (!this.graphs[graph].unlocked() || !player.graphOptions.shown[graph]) {
          return;
        }

        this.graphs[graph].prestiges.forEach(prestige => {
          this.ctx.strokeStyle = getComputedStyle(document.body).getPropertyValue(this.prestigeColors[prestige]);
          let toRemove = [];
          player.graphPrestigesData[prestige].forEach(ts => {
            let timeSince = Date.now() - ts;
            if (timeSince <= this.trackedTime * 1000) {
              let width = this.resolution * (this.predictive ? 2 : 3);
              let x = width - (timeSince * (width / (this.trackedTime * 1000)));

              this.ctx.beginPath();
              this.ctx.moveTo(x, 0);
              this.ctx.lineTo(x, this.resolution);
              this.ctx.stroke();
            } else if (timeSince > 600 * 1000) {
              toRemove.push(ts);
            }
          });
          player.graphPrestigesData[prestige].filter(ts => !toRemove.includes(ts));
        });

        this.ctx.strokeStyle = getComputedStyle(document.body).getPropertyValue(this.graphs[graph].color);
        this.ctx.beginPath();
        let graphMax = new Decimal(1e2);
        let values = [];
        for (let i = tOff / intervalTime; i <= player.graphOptions.intervals; i++) {
          let time = Math.floor(i * (intervalTime / player.options.updateRate));
          let point = player.graphData[graph][player.graphData[graph].length - (time + 1)] || { value: new Decimal(0) };
          let value = point.value instanceof Decimal ? point.value : new Decimal(0);
          graphMax = Decimal.max(graphMax, value);
          values.push(value);
        }
        
        let first = 1;
        values.forEach((value, i) => {
          if (value === true) {
            value = -10;
            first = 2;
          } else {
            if (value instanceof Decimal == false) {
              value = new Decimal(value);
            }
            if (player.graphOptions.logarithmic) {
              value = value.toNumber() == 0 ? new Decimal(0) : new Decimal(value.log(10));
            }
            value = Math.max(value.div(player.graphOptions.logarithmic ? graphMax.log(10) * 1.1 : graphMax.mul(new Decimal(1.1))).mul(new Decimal(this.resolution)).toNumber(), 0);
          }
          let width = ((player.graphOptions.predictive ? 2 : 3) * this.resolution);
          if (first > 0) {
            this.ctx.moveTo(width - ((i + tOff / intervalTime) / player.graphOptions.intervals) * width, this.resolution - value);
            first--;
          } else {
            this.ctx.lineTo(width - ((i + tOff / intervalTime) / player.graphOptions.intervals) * width, this.resolution - value);
          }
        });

        this.ctx.stroke();
      });
    }
  }
};
</script>

<template>
    <canvas class=graph ref="cnvGraph"></canvas>
</template>

<style scoped>
  .graph {
    margin-top: 10px;
    background-color: var(--color-primary);
    border: 2px solid var(--color-good);
    width: 80%;
    aspect-ratio: 3;
    image-rendering: crisp-edges;
  }
</style>