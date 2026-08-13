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
        },
        replicanti: {
          label: "Replicanti",
          color: "--color-replicanti",
          prestiges: ["crunches", "rGalaxies"],
          unlocked() {
            return PlayerProgress.replicantiUnlocked();
          }
        },
        infinityPower: {
          label: "Infinity Power",
          color: "--color-infinity",
          prestiges: ["crunches"],
          unlocked() {
            return player.dimensions.infinity[0].isUnlocked || PlayerProgress.eternityUnlocked();
          }
        }
      }
    };
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
</style>
