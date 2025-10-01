<template>
  <div class="dashboard-container">
    <div class="row">
      <div class="col-lg-6 col-md-12 mb-4">
        <div class="chart-container">
          <h3 class="chart-title">Top 20 Countries by Life Expectancy</h3>
          <div ref="barChart" class="chart"></div>
        </div>
      </div>
      
      <div class="col-lg-6 col-md-12 mb-4">
        <div class="chart-container">
          <h3 class="chart-title">Life Expectancy Over Time</h3>
          <div ref="lineChart" class="chart"></div>
        </div>
      </div>
    </div>
    
    <div class="row">
      <div class="col-lg-6 col-md-12 mb-4">
        <div class="chart-container">
          <h3 class="chart-title">GDP vs Population</h3>
          <div ref="scatterPlot" class="chart"></div>
        </div>
      </div>
      
      <div class="col-lg-6 col-md-12 mb-4">
        <div class="chart-container">
          <h3 class="chart-title">Life Expectancy by Continent</h3>
          <div ref="pieChart" class="chart"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import * as d3 from "d3";

const barChart = ref(null);
const lineChart = ref(null);
const scatterPlot = ref(null);
const pieChart = ref(null);

let data = [];

onMounted(async () => {
  try {
    data = await d3.csv("/gapminder_data.csv", d => ({
      country: d.country,
      year: +d.year,
      pop: +d.pop,
      continent: d.continent,
      lifeExp: +d.lifeExp,
      gdpPercap: +d.gdpPercap
    }));
    
    drawBarChart();
    drawLineChart();
    drawScatterPlot();
    drawPieChart();
  } catch (error) {
    console.error("Error loading data:", error);
  }
});

function drawBarChart() {
  const margin = { top: 20, right: 20, bottom: 80, left: 60 };
  const width = 500 - margin.left - margin.right;
  const height = 400 - margin.bottom - margin.top;

  d3.select(barChart.value).selectAll("*").remove();

  const svg = d3.select(barChart.value)
    .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom);

  const g = svg.append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

  const latestYear = d3.max(data, d => d.year);
  const latestData = data.filter(d => d.year === latestYear)
    .sort((a, b) => b.lifeExp - a.lifeExp)
    .slice(0, 20);

  const x = d3.scaleBand()
    .domain(latestData.map(d => d.country))
    .range([0, width])
    .padding(0.1);

  const y = d3.scaleLinear()
    .domain([0, d3.max(latestData, d => d.lifeExp)])
    .range([height, 0]);

  const color = d3.scaleOrdinal()
    .domain(latestData.map(d => d.continent))
    .range(d3.schemeSet2);

  g.selectAll(".bar")
    .data(latestData)
    .enter().append("rect")
    .attr("class", "bar")
    .attr("x", d => x(d.country))
    .attr("width", x.bandwidth())
    .attr("y", d => y(d.lifeExp))
    .attr("height", d => height - y(d.lifeExp))
    .attr("fill", d => color(d.continent))
    .on("mouseover", function(event, d) {
      d3.select(this).attr("opacity", 0.7);
      
      const tooltip = d3.select("body").append("div")
        .attr("class", "tooltip")
        .style("opacity", 0);
      
      tooltip.transition()
        .duration(200)
        .style("opacity", .9);
      tooltip.html(`${d.country}<br/>Life Exp: ${d.lifeExp.toFixed(1)} years`)
        .style("left", (event.pageX + 10) + "px")
        .style("top", (event.pageY - 28) + "px");
    })
    .on("mouseout", function() {
      d3.select(this).attr("opacity", 1);
      d3.selectAll(".tooltip").remove();
    });

  g.append("g")
    .attr("transform", `translate(0,${height})`)
    .call(d3.axisBottom(x))
    .selectAll("text")
    .style("text-anchor", "end")
    .attr("dx", "-.8em")
    .attr("dy", ".15em")
    .attr("transform", "rotate(-45)");

  g.append("g")
    .call(d3.axisLeft(y));

  g.append("text")
    .attr("transform", "rotate(-90)")
    .attr("y", 0 - margin.left)
    .attr("x", 0 - (height / 2))
    .attr("dy", "1em")
    .style("text-anchor", "middle")
    .text("Life Expectancy");
}

function drawLineChart() {
  const margin = { top: 20, right: 80, bottom: 50, left: 60 };
  const width = 500 - margin.left - margin.right;
  const height = 400 - margin.bottom - margin.top;

  d3.select(lineChart.value).selectAll("*").remove();

  const svg = d3.select(lineChart.value)
    .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom);

  const g = svg.append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

  const continentData = d3.group(data, d => d.continent);
  const aggregatedData = [];

  continentData.forEach((values, continent) => {
    const yearData = d3.rollup(values, 
      v => d3.mean(v, d => d.lifeExp), 
      d => d.year
    );
    
    yearData.forEach((lifeExp, year) => {
      aggregatedData.push({ continent, year, lifeExp });
    });
  });

  const x = d3.scaleLinear()
    .domain(d3.extent(data, d => d.year))
    .range([0, width]);

  const y = d3.scaleLinear()
    .domain(d3.extent(aggregatedData, d => d.lifeExp))
    .range([height, 0]);

  const color = d3.scaleOrdinal()
    .domain([...continentData.keys()])
    .range(d3.schemeSet1);

  const line = d3.line()
    .x(d => x(d.year))
    .y(d => y(d.lifeExp));

  const nestedData = d3.group(aggregatedData, d => d.continent);

  nestedData.forEach((values, continent) => {
    g.append("path")
      .datum(values.sort((a, b) => a.year - b.year))
      .attr("fill", "none")
      .attr("stroke", color(continent))
      .attr("stroke-width", 2)
      .attr("d", line);

    g.selectAll(`.dot-${continent.replace(/\s+/g, '')}`)
      .data(values)
      .enter().append("circle")
      .attr("cx", d => x(d.year))
      .attr("cy", d => y(d.lifeExp))
      .attr("r", 3)
      .attr("fill", color(continent));
  });

  g.append("g")
    .attr("transform", `translate(0,${height})`)
    .call(d3.axisBottom(x).tickFormat(d3.format("d")));

  g.append("g")
    .call(d3.axisLeft(y));

  g.append("text")
    .attr("transform", `translate(${width / 2}, ${height + 40})`)
    .style("text-anchor", "middle")
    .text("Year");

  g.append("text")
    .attr("transform", "rotate(-90)")
    .attr("y", 0 - margin.left)
    .attr("x", 0 - (height / 2))
    .attr("dy", "1em")
    .style("text-anchor", "middle")
    .text("Life Expectancy");

  const legend = g.selectAll(".legend")
    .data([...continentData.keys()])
    .enter().append("g")
    .attr("class", "legend")
    .attr("transform", (d, i) => `translate(${width + 10},${i * 20})`);

  legend.append("rect")
    .attr("x", 0)
    .attr("width", 18)
    .attr("height", 18)
    .style("fill", color);

  legend.append("text")
    .attr("x", 25)
    .attr("y", 9)
    .attr("dy", ".35em")
    .style("text-anchor", "start")
    .text(d => d);
}

function drawScatterPlot() {
  const margin = { top: 20, right: 20, bottom: 60, left: 80 };
  const width = 500 - margin.left - margin.right;
  const height = 400 - margin.bottom - margin.top;

  d3.select(scatterPlot.value).selectAll("*").remove();

  const svg = d3.select(scatterPlot.value)
    .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom);

  const g = svg.append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

  const latestYear = d3.max(data, d => d.year);
  const latestData = data.filter(d => d.year === latestYear);

  const x = d3.scaleLog()
    .domain(d3.extent(latestData, d => d.gdpPercap))
    .range([0, width]);

  const y = d3.scaleLog()
    .domain(d3.extent(latestData, d => d.pop))
    .range([height, 0]);

  const color = d3.scaleOrdinal()
    .domain([...new Set(latestData.map(d => d.continent))])
    .range(d3.schemeSet2);

  const radius = d3.scaleLinear()
    .domain(d3.extent(latestData, d => d.lifeExp))
    .range([3, 15]);

  g.selectAll(".dot")
    .data(latestData)
    .enter().append("circle")
    .attr("class", "dot")
    .attr("r", d => radius(d.lifeExp))
    .attr("cx", d => x(d.gdpPercap))
    .attr("cy", d => y(d.pop))
    .style("fill", d => color(d.continent))
    .style("opacity", 0.7)
    .on("mouseover", function(event, d) {
      d3.select(this).style("opacity", 1).attr("stroke", "#000").attr("stroke-width", 2);
      
      const tooltip = d3.select("body").append("div")
        .attr("class", "tooltip")
        .style("opacity", 0);
      
      tooltip.transition()
        .duration(200)
        .style("opacity", .9);
      tooltip.html(`${d.country}<br/>GDP: $${d.gdpPercap.toLocaleString()}<br/>Population: ${d.pop.toLocaleString()}`)
        .style("left", (event.pageX + 10) + "px")
        .style("top", (event.pageY - 28) + "px");
    })
    .on("mouseout", function() {
      d3.select(this).style("opacity", 0.7).attr("stroke", "none");
      d3.selectAll(".tooltip").remove();
    });

  g.append("g")
    .attr("transform", `translate(0,${height})`)
    .call(d3.axisBottom(x).tickFormat(d => `$${d.toLocaleString()}`));

  g.append("g")
    .call(d3.axisLeft(y).tickFormat(d => d.toExponential()));

  g.append("text")
    .attr("transform", `translate(${width / 2}, ${height + 50})`)
    .style("text-anchor", "middle")
    .text("GDP per Capita");

  g.append("text")
    .attr("transform", "rotate(-90)")
    .attr("y", 0 - margin.left)
    .attr("x", 0 - (height / 2))
    .attr("dy", "1em")
    .style("text-anchor", "middle")
    .text("Population");
}

function drawPieChart() {
  const width = 400;
  const height = 400;
  const radius = Math.min(width, height) / 2 - 20;

  d3.select(pieChart.value).selectAll("*").remove();

  const svg = d3.select(pieChart.value)
    .append("svg")
    .attr("width", width)
    .attr("height", height);

  const g = svg.append("g")
    .attr("transform", `translate(${width / 2},${height / 2})`);

  const latestYear = d3.max(data, d => d.year);
  const latestData = data.filter(d => d.year === latestYear);
  
  const continentAvgs = d3.rollup(latestData, 
    v => d3.mean(v, d => d.lifeExp), 
    d => d.continent
  );

  const pieData = Array.from(continentAvgs, ([continent, lifeExp]) => ({
    continent,
    lifeExp
  }));

  const color = d3.scaleOrdinal()
    .domain(pieData.map(d => d.continent))
    .range(d3.schemeSet3);

  const pie = d3.pie()
    .value(d => d.lifeExp)
    .sort(null);

  const arc = d3.arc()
    .innerRadius(0)
    .outerRadius(radius);

  const labelArc = d3.arc()
    .outerRadius(radius + 30)
    .innerRadius(radius + 30);

  const arcs = g.selectAll(".arc")
    .data(pie(pieData))
    .enter().append("g")
    .attr("class", "arc");

  arcs.append("path")
    .attr("d", arc)
    .attr("fill", d => color(d.data.continent))
    .attr("stroke", "#fff")
    .attr("stroke-width", 2)
    .on("mouseover", function(event, d) {
      d3.select(this).attr("opacity", 0.7);
      
      const tooltip = d3.select("body").append("div")
        .attr("class", "tooltip")
        .style("opacity", 0);
      
      tooltip.transition()
        .duration(200)
        .style("opacity", .9);
      tooltip.html(`${d.data.continent}<br/>Avg Life Exp: ${d.data.lifeExp.toFixed(1)} years`)
        .style("left", (event.pageX + 10) + "px")
        .style("top", (event.pageY - 28) + "px");
    })
    .on("mouseout", function() {
      d3.select(this).attr("opacity", 1);
      d3.selectAll(".tooltip").remove();
    });

  arcs.append("text")
    .attr("transform", d => `translate(${labelArc.centroid(d)})`)
    .attr("dy", ".35em")
    .style("text-anchor", "middle")
    .style("font-size", "11px")
    .style("font-weight", "bold")
    .text(d => {
      const percent = ((d.endAngle - d.startAngle) / (2 * Math.PI) * 100).toFixed(1);
      return `${d.data.continent} (${percent}%)`;
    });
}
</script>

<style scoped>
.dashboard-container {
  padding: 20px;
}

.chart-container {
  background: #f8f9fa;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  height: 100%;
}

.chart-title {
  text-align: center;
  margin-bottom: 20px;
  color: #333;
  font-size: 1.1rem;
  font-weight: 600;
}

.chart {
  display: flex;
  justify-content: center;
  align-items: center;
}

:global(.tooltip) {
  position: absolute;
  text-align: center;
  padding: 8px;
  font-size: 12px;
  background: rgba(0, 0, 0, 0.8);
  color: white;
  border-radius: 4px;
  pointer-events: none;
}
</style>
