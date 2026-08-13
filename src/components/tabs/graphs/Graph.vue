<script>
import PrimaryToggleButton from "@/components/PrimaryToggleButton";
import SliderComponent from "@/components/SliderComponent";

export default {
  name: "Graph",
  props: {
    graphID: String,
    data: Object
  },
  components: {
    PrimaryToggleButton,
    SliderComponent
  },
  data() {
    return {
      label: "COOL",
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
      shown: true,
      prestigeColors: {
        boosts: "--color-good-dark",
        galaxies: "--color-good",
        crunches: "--color-infinity",
        rGalaxies: "--color-replicanti",
        eternities: "--color-eternity"
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
      player.graphOptions[this.graphID].predictive = newValue;
    },
    logarithmic(newValue) {
      player.graphOptions[this.graphID].logarithmic = newValue;
    },
    shown(newValue) {
      player.graphOptions[this.graphID].shown = newValue;
      this.data.shown = newValue;
    }
  },
  mounted() {
    if (!this.data.unlocked()) {
      return;
    }

    const options = player.graphOptions[this.graphID];
    this.intervalsSlider = options.intervals;
    this.trackedTimeSlider = this.trackedTimes.indexOf(options.trackedTime);
    this.predictive = options.predictive;
    this.logarithmic = options.logarithmic;
    this.shown = options.shown;
    this.data.shown = options.shown;
    
    this.cnv = this.$refs.cnvGraph;
    this.cnv.width = 3 * this.resolution;
    this.cnv.height = this.resolution;
    this.ctx = this.cnv.getContext("2d");
  },
  methods: {
    update() {
      if (this.cnv == undefined) {
        return;
      }

      this.ctx.clearRect(0, 0, 3 * this.resolution, this.resolution);
      this.ctx.lineWidth = 5;
      
      if (!this.data.unlocked() || !player.graphOptions[this.graphID].shown) {
        return;
      }

      this.data.prestiges.forEach(prestige => {
        this.ctx.strokeStyle = getComputedStyle(document.body).getPropertyValue(this.prestigeColors[prestige]);
        let toRemove = [];
        player.graphData.prestiges[prestige].forEach(ts => {
          let timeSince = Date.now() - ts;
          if (timeSince <= this.trackedTime * 1000) {
            let width = this.resolution * (this.predictive ? 2 : 3);
            let x = width - (timeSince * (width / (this.trackedTime * 1000)));

            this.ctx.beginPath();
            this.ctx.moveTo(x, 0);
            this.ctx.lineTo(x, this.resolution - 50);
            this.ctx.stroke();
          } else if (timeSince > 600 * 1000) {
            toRemove.push(ts);
          }
        });
        player.graphData.prestiges[prestige].filter(ts => !toRemove.includes(ts));
      });

      this.ctx.strokeStyle = getComputedStyle(document.body).getPropertyValue(this.data.color);
      this.ctx.beginPath();
      let graphMax = new Decimal(1e2);
      let width = ((player.graphOptions[this.graphID].predictive ? 2 : 3) * this.resolution);
      let points = [];
      for (let i = 0; i < (this.trackedTime * 1000) / player.options.updateRate; i += 100 / this.intervalsSlider) {
        let point = player.graphData.products[this.graphID][player.graphData.products[this.graphID].length - Math.floor(i + 1)] || { value: new Decimal(0) };
        let value = point.value instanceof Decimal ? point.value : new Decimal(0);
        graphMax = Decimal.max(graphMax, value);
        points.push(point);
      }
      
      let first = 1;
      points.forEach(point => {
        let value = point.value;
        if (value === true) {
          value = -10;
          first = 2;
        } else {
          if (value instanceof Decimal == false) {
            value = new Decimal(value);
          }
          if (player.graphOptions[this.graphID].logarithmic) {
            value = value.toNumber() == 0 ? new Decimal(0) : new Decimal(value.log(10));
          }
          value = Math.max(value.div(player.graphOptions[this.graphID].logarithmic ? graphMax.log(10) * 1.1 : graphMax.mul(new Decimal(1.1))).mul(new Decimal(this.resolution)).toNumber(), 0);
        }
        let timeSince = Date.now() - point.TS;
        let x = width - timeSince * (width / (this.trackedTime * 1000));
        if (first > 0) {
          this.ctx.moveTo(x, this.resolution - value - 50);
          first--;
        } else {
          this.ctx.lineTo(x, this.resolution - value - 50);
        }
      });

      this.ctx.stroke();

      if (player.graphData.challenges.filter(c => {return Math.abs(Date.now() - c.t < this.trackedTime * 1000);}).length > 0) {
        let enter = Date.now() - this.trackedTime * 1000;
        player.graphData.challenges.forEach(c => {
          if (c.y == "enter") {
            enter = c.t;
          } else if (c.y == "exit" && Date.now() - c.t < this.trackedTime * 1000) {
            this.ctx.fillStyle = getComputedStyle(document.body).getPropertyValue(`--color-${c.c}`);
            let x = width - Math.abs(Date.now() - c.t) * (width / (this.trackedTime * 1000));
            let x2 = width - Math.abs(Date.now() - enter) * (width / (this.trackedTime * 1000));
            this.ctx.fillRect(x2, this.resolution - 50, x - x2, 50);
            enter = false;
          }
        });
        if (enter) {
          if (NormalChallenge.isRunning) {
            this.ctx.fillStyle = getComputedStyle(document.body).getPropertyValue("--color-antimatter");
          } else if (InfinityChallenge.isRunning) {
            this.ctx.fillStyle = getComputedStyle(document.body).getPropertyValue("--color-infinity");
          } else if (EternityChallenge.isRunning) {
            this.ctx.fillStyle = getComputedStyle(document.body).getPropertyValue("--color-eternity");
          } else if (player.dilation.active) {
            this.ctx.fillStyle = getComputedStyle(document.body).getPropertyValue("--color-dilation");
          }
          let x = width - (Date.now() - enter) * (width / (this.trackedTime * 1000));
          this.ctx.fillRect(x, this.resolution - 50, width, this.resolution);
        }
      } else {
        if (NormalChallenge.isRunning) {
          console.log("in Normal Challenge")
          this.ctx.fillStyle = getComputedStyle(document.body).getPropertyValue("--color-antimatter");
          this.ctx.fillRect(0, this.resolution - 50, width, 50);
        } else if (InfinityChallenge.isRunning) {
          this.ctx.fillStyle = getComputedStyle(document.body).getPropertyValue("--color-infinity");
          this.ctx.fillRect(0, this.resolution - 50, width, 50);
        } else if (EternityChallenge.isRunning) {
          this.ctx.fillStyle = getComputedStyle(document.body).getPropertyValue("--color-eternity");
          this.ctx.fillRect(0, this.resolution - 50, width, 50);
        } else if (player.dilation.active) {
          this.ctx.fillStyle = getComputedStyle(document.body).getPropertyValue("--color-dilation");
          this.ctx.fillRect(0, this.resolution - 50, width, 50);
        } else {
          this.ctx.clearRect(0, this.resolution - 50, width, 50);
        }
      }
      
      this.ctx.strokeStyle = getComputedStyle(document.body).getPropertyValue("--color-good-dark");
      this.ctx.beginPath();
      this.ctx.moveTo(0, this.resolution - 49);
      this.ctx.lineTo(width, this.resolution - 49);
      this.ctx.stroke();
    },
    adjustSliderValueIntervalsSlider(value) {
      this.intervalsSlider = value;
      player.graphOptions[this.graphID].intervals = value;
    },
    adjustSliderValueTrackedTimeSlider(value) {
      this.trackedTimeSlider = value;
      this.trackedTime = this.trackedTimes[value];
      this.trackedTimeDisp = this.trackedTime >= 60 ? `${this.trackedTime / 60}m` : `${this.trackedTime}s`;
      player.graphOptions[this.graphID].trackedTime = this.trackedTimes[value];
    }
  }
};
</script>

<template>
  <div v-if="data.unlocked()" :class="{active: shown, inactive: !shown, graph: true}">
    <PrimaryToggleButton
      v-model="shown"
      class="o-primary-btn l-toggle-button l-graph-shown-btn"
      id="shown"
      on=""
      off=""
      :label="data.label"
    />
    <canvas class=graph ref="cnvGraph"></canvas>
    <div class=l-graph-buttons>
      <PrimaryToggleButton
        v-model="logarithmic"
        class="o-primary-btn l-toggle-button l-graphs-opt-button"
        id="logarithmic"
        label="Logarithmic:"
      />
      <div class="o-primary-btn o-primary-btn--option o-primary-btn--slider l-graphs-opt-button">
        <b>Resolution: {{ formatInt(intervalsSlider) }}%</b>
        <SliderComponent
          class="o-primary-btn--slider__slider"
          v-bind="sliderPropsIntervalsSlider"
          :value="intervalsSlider"
          @input="adjustSliderValueIntervalsSlider($event)"
        />
      </div>
      <div class="o-primary-btn o-primary-btn--option o-primary-btn--slider l-graphs-opt-button">
        <b>Tracked Time: {{ trackedTimeDisp }}</b>
        <SliderComponent
          class="o-primary-btn--slider__slider "
          v-bind="sliderPropsTrackedTimeSlider"
          :value="trackedTimeSlider"
          @input="adjustSliderValueTrackedTimeSlider($event)"
        />
      </div>
      <PrimaryToggleButton
        v-model="predictive"
        class="o-primary-btn l-toggle-button l-graphs-opt-button"
        id="predictive"
        label="Predictive:"
      />
    </div>
  </div>
</template>

<style scoped>
  div.graph {
    position: relative;
    border: 2px solid var(--color-good);
    border-radius: 5px;
    margin-top: 20px;
    margin-bottom: 5px;
    width: 1600px;
    height: 395px;
    padding: 0px;
    overflow: hidden;
    transition: 500ms;
  }

  div.graph.inactive {
    height: 80px;
    width: 1600px;
  }

  .l-graph-shown-btn {
    position: absolute;
    left: 0px;
    top: 0px;
    height: 100%;
    z-index: 1;
    border: none;
    border-right: 2px solid var(--color-good-dark);
    border-radius: 0px;
    transition: 500ms;
  }

  .active .l-graph-shown-btn {
    width: 113px;
    font-size: 40px;
    writing-mode: sideways-lr;
  }

  .inactive .l-graph-shown-btn {
    width: 101%;
    font-size: 50px;
  }

  .l-graph-buttons {
    position: absolute;
    right: 0px;
    top: 0px;
    height: 100%;
  }

  .l-graphs-opt-button {
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 5px;
    margin-bottom: 20px;
    font-size: 20px;
    height: 80px;
    width: 300px;
    border-color: var(--color-good-dark);
  }

  canvas.graph {
    position: absolute;
    right: 312px;
    background-color: var(--color-base);
    border-left: 3px solid var(--color-good-dark);
    border-right: 2px solid var(--color-good-dark);
    height: 391px;
    aspect-ratio: 3;
    image-rendering: crisp-edges;
  }
</style>