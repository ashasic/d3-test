<template>
  <div ref="chart"></div>
</template>

<script setup>
import { ref, onMounted, watch } from "vue";
import * as d3 from "d3";

const chart = ref(null);
const data = ref([])

// Example data
const dataset = [5, 10, 20, 25, 30];

onMounted(async() => {
  // If your CSV is inside /public folder, e.g. /public/data.csv
  const csvData = await d3.csv('/iris.csv')
  drawChart(csvData);
});

function drawChart(data) {
  const width = 500;
  const height = 300;
  const margin = { top: 20, right: 20, bottom: 30, left: 40 };

  const svg = d3.select(chart.value)
    .append("svg")
    .attr("width", width)
    .attr("height", height);

  // Example data
  const dataset = [5, 10, 25, 30, 35];
  console.log('Loaded CSV:', data)

  // Scales
  const x = d3.scaleLinear()
    .domain(d3.extent(data, d => d['sepal.length']))
    .nice()
    .range([margin.left, width - margin.right])

  const y = d3.scaleLinear()
    .domain(d3.extent(data, d => d['sepal.width']))
    .nice()
    .range([height - margin.bottom, margin.top])

  const color = d3.scaleOrdinal()
    .domain([...new Set(data.map(d => d['variety']))])
    .range(d3.schemeCategory10)

  // Axes
  svg.append("g")
    .attr("transform", `translate(0,${height - margin.bottom})`)
    .call(d3.axisBottom(x))

  svg.append("g")
    .attr("transform", `translate(${margin.left},0)`)
    .call(d3.axisLeft(y))

  // Points
  svg.append("g")
    .selectAll("circle")
    .data(data)
    .join("circle")
      .attr("cx", d => x(d['sepal.length']))
      .attr("cy", d => y(d['sepal.width']))
      .attr("r", 4)
      .attr("fill", d => color(d['variety']))
      .attr("opacity", 0.7)

  // Labels
  svg.append("text")
    .attr("x", width / 2)
    .attr("y", height - 5)
    .attr("text-anchor", "middle")
    .text("Sepal Length")

  svg.append("text")
    .attr("x", -height / 2)
    .attr("y", 15)
    .attr("transform", "rotate(-90)")
    .attr("text-anchor", "middle")
    .text("Sepal Width")

}
</script>

<style scoped>
svg {
  font-family: sans-serif;
}
</style>
