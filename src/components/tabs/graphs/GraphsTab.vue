<script>
import PrimaryToggleButton from "@/components/PrimaryToggleButton";
import SliderComponent from "@/components/SliderComponent";
import Graph from "./Graph";

export default {
  name: "GraphsTab",
  components: {
    PrimaryToggleButton,
    SliderComponent,
    Graph
  },
  data() {
    return {
      graphs: {
        antimatter: {
          label: "Antimatter",
          color: "--color-antimatter",
          prestiges: ["boosts", "galaxies", "crunches", "eternities"],
          unlocked() {
            return true;
          }
        },
        infinityPoints: {
          label: "IP",
          color: "--color-infinity",
          prestiges: ["eternities"],
          unlocked() {
            return PlayerProgress.infinityUnlocked();
          }
        },
        replicanti: {
          label: "Replicanti",
          color: "--color-replicanti",
          prestiges: ["rGalaxies", "crunches", "eternities"],
          unlocked() {
            return PlayerProgress.replicantiUnlocked();
          }
        },
        infinityPower: {
          label: "Infinity Power",
          color: "--color-infinity",
          prestiges: ["crunches", "eternities"],
          unlocked() {
            return player.dimensions.infinity[0].isUnlocked || PlayerProgress.eternityUnlocked();
          }
        },
        eternityPoints: {
          label: "EP",
          color: "--color-eternity",
          prestiges: [],
          unlocked() {
            return PlayerProgress.eternityUnlocked();
          }
        },
        timeShards: {
          label: "Time Shards",
          color: "--color-eternity",
          prestiges: ["eternities"],
          unlocked() {
            return player.dimensions.time[0].amount.gt(0) || PlayerProgress.realityUnlocked();
          }
        },
        timeTheorems: {
          label: "Time Theorems",
          color: "--color-eternity",
          prestiges: ["eternities"],
          unlocked() {
            return player.timestudy.maxTheorem.gt(0) || PlayerProgress.realityUnlocked();
          }
        },
        tachyonParticles: {
          label: "Tach. Particles",
          color: "--color-tachyon-particle",
          prestiges: [],
          unlocked() {
            return PlayerProgress.dilationUnlocked();
          }
        },
        dilatedTime: {
          label: "Dilated Time",
          color: "--color-dilation",
          prestiges: ["eternities"],
          unlocked() {
            return PlayerProgress.dilationUnlocked();
          }
        }
      }
    };
  },
  methods: {
    update() {
      player.timeTheorems = player.timestudy.theorem;
      player.dilatedTime = player.dilation.dilatedTime;
      player.tachyonParticles = player.dilation.tachyonParticles;
    }
  }
};
</script>

<template>
  <div class="l-graphs-tab">
    <div class="l-graphs">
      <Graph
        v-for="graphID in Object.keys(graphs)"
        :graphID="graphID"
        :data="graphs[graphID]"
      />
    </div>
  </div>
</template>

<style scoped>
  div.l-graphs-tab {
    display: flex;
    justify-content: center;
  }

  div.l-graphs {
    display: grid;
    margin: 20px;
    grid-template-columns: 1fr;
    grid-row-gap: 10px;
  }
</style>
