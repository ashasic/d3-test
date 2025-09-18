<template>
  <div ref="chart"></div>
</template>

<script setup>
import { ref, onMounted, watch } from "vue";
import * as d3 from "d3";

const chart = ref(null);

// Example data
const dataset = [5, 10, 20, 25, 30];

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

  // Example data
const dataset = [5, 10, 25, 30, 35];

// svg.selectAll("text") //select all text in the html (none so far)
//    .data(dataset) //count and parses all data
//    .enter() //creates new, data-bound elements (placeholders) for the data values
//    .append("text") // takes the empty elements and enters text elements
//    .attr("x", 20) // x position
//    .attr("y", (d, i) => 20 + i * 20) // stack vertically
//    .text(d => "I can count up to "+ d) //text content
//    .attr("fill", d => {if (d<60) return "red"; else return "black";}); //style

  // var circles = svg.selectAll("circle")
  //   .data(dataset)
  //   .enter()
  //   .append("circle"); // now circles are appended to the end of the SVG element

  // circles.attr("cx", function(d, i) {return (i * 50) + 25;})
  //   .attr("cy", height/2)
  //   .attr("r", function(d) {return d;})
  //   .attr("fill", "yellow")
  //   .attr("stroke", "orange")
  //   .attr("stroke-width", function(d) {return d/2;});

  // svg.selectAll("text")
  //  .data(dataset)
  //  .enter()
  //  .append("text")
  //  .attr("x", 20)
  //  .attr("dy", (d, i) => 20 + i * 20)
  //  .text(d => "I can count up to "+ d)
  //  .attr("fill", d => {if (d > 10) return "red"; else return "black";});

  const x = d3.scaleBand()
    .domain(dataset.map((_, i) => i))
    .range([margin.left, width - margin.right])
    .padding(0.1);

  const y = d3.scaleLinear()
    .domain([0, d3.max(dataset)])
    .nice()
    .range([height - margin.bottom, margin.top]);

  var bars = svg.append("g")
    .attr("fill", "steelblue")
    .selectAll("rect")
    .data(dataset)
    .join("rect")
      .attr("x", (_, i) => x(i))
      .attr("y", d => y(d))
      .attr("height", d => y(0) - y(d))
      .attr("width", x.bandwidth());
  
  bars.on("mouseover", function(event, d) {
    d3.select(this).attr("fill", "orange")
  })

  bars.on("mouseout", function(event, d) {
    d3.select(this).attr("fill", "steelblue")
  })

  // X Axis
  svg.append("g")
    .attr("transform", `translate(0,${height - margin.bottom})`)
    .call(d3.axisBottom(x).tickFormat(i => i + 1).tickSizeOuter(0));

  // Y Axis
  svg.append("g")
    .attr("transform", `translate(${margin.left},0)`)
    .call(d3.axisLeft(y));

}
</script>

<style scoped>
svg {
  font-family: sans-serif;
}
</style>
