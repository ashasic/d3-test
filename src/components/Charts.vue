<template>
  <div ref="chart"></div>
</template>

<script setup>
import { ref, onMounted, watch } from "vue";
import * as d3 from "d3";

const chart = ref(null);

// Example data
const dataset = [30, 80, 45, 60, 20, 90, 50];

onMounted(() => {
  drawChart();
});

function drawChart() {
  const width = 500;
  const height = 300;
  const margin = { top: 20, right: 20, bottom: 30, left: 40 };

  const svg = d3.select(chart.value)
    .append("svg")
    .attr("width", width)
    .attr("height", height);

  svg.selectAll("text")
    .data(dataset)
    .enter()
    .append("text")
    .attr("x", 20) // x position
    .attr("dy", (d, i) => 20 + i * 20) // stack vertically
    .text("A new text")
    .attr("fill", "black");
}
</script>

<style scoped>
svg {
  font-family: sans-serif;
}
</style>
