<template>
  <div class="dashboard-container">
    <div class="row">
      <div class="col-12">
        <div class="chart-container">
          <h3 class="chart-title">Iris Dataset Scatterplot Matrix</h3>
          <div ref="scatterMatrix" class="chart"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import * as d3 from "d3";

const scatterMatrix = ref(null);
let data = [];

onMounted(async () => {
  try {
    data = await d3.csv("/src/assets/iris.csv", d => ({
      "sepal.length": +d["sepal.length"],
      "sepal.width": +d["sepal.width"],
      "petal.length": +d["petal.length"],
      "petal.width": +d["petal.width"],
      variety: d.variety
    }));
    
    drawScatterMatrix();
  } catch (error) {
    console.error("Error loading data:", error);
  }
});

function drawScatterMatrix() {
  d3.select(scatterMatrix.value).selectAll("*").remove();

  const width = 1200;
  const height = width;
  const padding = 60;
  const columns = ["sepal.length", "sepal.width", "petal.length", "petal.width"];
  const size = (width - (columns.length + 1) * padding) / columns.length + padding;

  // Create popup
  const popup = d3.select("body").selectAll(".popup")
    .data([1])
    .join("div")
    .attr("class", "popup")
    .style("display", "none");

  popup.append("div")
    .attr("class", "popup-close")
    .text("×")
    .on("click", () => popup.style("display", "none"));

  popup.append("div")
    .attr("class", "popup-content");

  const x = columns.map(c => d3.scaleLinear()
      .domain(d3.extent(data, d => d[c]))
      .rangeRound([padding / 2, size - padding / 2]));

  const y = x.map(x => x.copy().range([size - padding / 2, padding / 2]));

  const color = d3.scaleOrdinal()
      .domain(data.map(d => d.variety))
      .range(d3.schemeCategory10);

  const svg = d3.select(scatterMatrix.value)
      .append("svg")
      .attr("width", width)
      .attr("height", height)
      .attr("viewBox", [-padding, 0, width, height]);

  // Functions (defined after svg is created)
  const showPointPopup = (d, event) => {
    console.log("showPointPopup called with:", d);
    const content = popup.select(".popup-content");
    content.html(`
      <h3>${d.variety}</h3>
      <div class="popup-details">
        <div><strong>Sepal Length:</strong> ${d["sepal.length"]} cm</div>
        <div><strong>Sepal Width:</strong> ${d["sepal.width"]} cm</div>
        <div><strong>Petal Length:</strong> ${d["petal.length"]} cm</div>
        <div><strong>Petal Width:</strong> ${d["petal.width"]} cm</div>
      </div>
    `);
    
    popup
      .style("display", "block")
      .style("left", (event.pageX + 15) + "px")
      .style("top", (event.pageY + 15) + "px");
  };

  const updateHighlights = () => {
    svg.selectAll("circle").each(function(d) {
      const pointId = `${d["sepal.length"]}-${d["sepal.width"]}-${d["petal.length"]}-${d["petal.width"]}`;
      if (selectedPoints.has(pointId)) {
        d3.select(this).attr("r", 6).attr("stroke-width", 3).attr("stroke", "#333").attr("fill-opacity", 1);
      } else {
        d3.select(this).attr("r", 4).attr("stroke-width", 0.5).attr("stroke", "#fff").attr("fill-opacity", 0.8);
      }
    });
    
    svg.selectAll("rect.histogram-bar").each(function() {
      const bar = d3.select(this);
      if (bar.classed("selected")) {
        bar.attr("stroke", "#333").attr("stroke-width", 3);
      } else {
        bar.attr("stroke", "#fff").attr("stroke-width", 1);
      }
    });
  };

  svg.append("style")
      .text(`
        circle.hidden { fill: #000; fill-opacity: 0.1; r: 2px; }
        circle.highlighted { stroke: #000; stroke-width: 3; r: 6; }
        .brush .selection { fill: rgba(70, 130, 180, 0.3); stroke: steelblue; stroke-width: 2; }
        .histogram-bar:hover { opacity: 0.7; cursor: pointer; }
        .histogram-bar.selected { stroke: #333; stroke-width: 2; }
        .tooltip { position: absolute; background: rgba(0,0,0,0.9); color: white; padding: 15px; border-radius: 8px; pointer-events: none; font-size: 14px; box-shadow: 0 4px 12px rgba(0,0,0,0.3); }
        .popup { position: absolute; background: white; border: 2px solid #3498db; border-radius: 8px; padding: 15px; box-shadow: 0 8px 16px rgba(0,0,0,0.3); z-index: 1001; max-width: 300px; }
        .popup-close { position: absolute; top: 5px; right: 10px; cursor: pointer; font-size: 18px; font-weight: bold; color: #666; }
        .popup-close:hover { color: #000; }
      `);

  const axisx = d3.axisBottom()
      .ticks(5)
      .tickSize(size * columns.length);
  const xAxis = g => g.selectAll("g").data(x).join("g")
      .attr("transform", (d, i) => `translate(${i * size},0)`)
      .each(function(d) { return d3.select(this).call(axisx.scale(d)); })
      .call(g => g.select(".domain").remove())
      .call(g => g.selectAll(".tick line").attr("stroke", "#ddd"))
      .call(g => g.selectAll(".tick text").style("font-size", "12px"));

  const axisy = d3.axisLeft()
      .ticks(5)
      .tickSize(-size * columns.length);
  const yAxis = g => g.selectAll("g").data(y).join("g")
      .attr("transform", (d, i) => `translate(0,${i * size})`)
      .each(function(d) { return d3.select(this).call(axisy.scale(d)); })
      .call(g => g.select(".domain").remove())
      .call(g => g.selectAll(".tick line").attr("stroke", "#ddd"))
      .call(g => g.selectAll(".tick text").style("font-size", "12px"));

  svg.append("g").call(xAxis);
  svg.append("g").call(yAxis);

  svg.append("g")
    .selectAll("text")
    .data(columns)
    .join("text")
    .attr("transform", (d, i) => `translate(${i * size + size/2}, ${-25})`)
    .style("text-anchor", "middle")
    .style("font", "bold 16px sans-serif")
    .style("fill", "#2c3e50")
    .text(d => d.replace(".", " ").replace(/\b\w/g, l => l.toUpperCase()));

  svg.append("g")
    .selectAll("text")
    .data(columns)
    .join("text")
    .attr("transform", (d, i) => `translate(${-35}, ${i * size + size/2}) rotate(-90)`)
    .style("text-anchor", "middle")
    .style("font", "bold 16px sans-serif")
    .style("fill", "#2c3e50")
    .text(d => d.replace(".", " ").replace(/\b\w/g, l => l.toUpperCase()));

  const cell = svg.append("g")
    .selectAll("g")
    .data(d3.cross(d3.range(columns.length), d3.range(columns.length)))
    .join("g")
      .attr("transform", ([i, j]) => `translate(${i * size},${j * size})`);

  cell.append("rect")
      .attr("fill", "none")
      .attr("stroke", "#aaa")
      .attr("stroke-width", 1.5)
      .attr("x", padding / 2 + 0.5)
      .attr("y", padding / 2 + 0.5)
      .attr("width", size - padding)
      .attr("height", size - padding);

  const tooltip = d3.select("body").append("div")
    .attr("class", "tooltip")
    .style("opacity", 0);

  let selectedPoints = new Set();
  let selectedHistogramBars = new Set();

  cell.each(function([i, j]) {
    const cellData = data.filter(d => !isNaN(d[columns[i]]) && !isNaN(d[columns[j]]));
    
    if (i === j) {
      const histogramData = d3.histogram()
        .domain(x[i].domain())
        .thresholds(12)
        (data.map(d => d[columns[i]]));

      const yHist = d3.scaleLinear()
        .domain([0, d3.max(histogramData, d => d.length)])
        .range([size - padding / 2, padding / 2]);

      d3.select(this).selectAll("rect.histogram-bar")
        .data(histogramData)
        .join("rect")
        .attr("class", "histogram-bar")
        .attr("x", d => x[i](d.x0))
        .attr("y", d => yHist(d.length))
        .attr("width", d => Math.max(0, x[i](d.x1) - x[i](d.x0) - 2))
        .attr("height", d => yHist(0) - yHist(d.length))
        .attr("fill", "steelblue")
        .attr("opacity", 0.8)
        .attr("stroke", "#fff")
        .attr("stroke-width", 1)
        .style("cursor", "pointer")
        .on("mouseover", function(event, d) {
          d3.select(this).attr("opacity", 1);
          tooltip.transition().duration(200).style("opacity", .9);
          tooltip.html(`Range: ${d.x0.toFixed(1)} - ${d.x1.toFixed(1)}<br/>Count: ${d.length} flowers`)
            .style("left", (event.pageX + 10) + "px")
            .style("top", (event.pageY - 28) + "px");
        })
        .on("mouseout", function() {
          d3.select(this).attr("opacity", 0.8);
          tooltip.transition().duration(500).style("opacity", 0);
        })
        .on("click", function(event, d) {
          event.stopPropagation();
          const barId = `${columns[i]}-${d.x0}-${d.x1}`;
          const pointsInBin = data.filter(point => point[columns[i]] >= d.x0 && point[columns[i]] < d.x1);
          
          if (selectedHistogramBars.has(barId)) {
            selectedHistogramBars.delete(barId);
            d3.select(this).classed("selected", false);
            pointsInBin.forEach(point => {
              const pointId = `${point["sepal.length"]}-${point["sepal.width"]}-${point["petal.length"]}-${point["petal.width"]}`;
              selectedPoints.delete(pointId);
            });
          } else {
            selectedHistogramBars.add(barId);
            d3.select(this).classed("selected", true);
            pointsInBin.forEach(point => {
              const pointId = `${point["sepal.length"]}-${point["sepal.width"]}-${point["petal.length"]}-${point["petal.width"]}`;
              selectedPoints.add(pointId);
            });
          }
          updateHighlights();
        });
    } else {
      d3.select(this).selectAll("circle")
        .data(cellData)
        .join("circle")
        .attr("cx", d => x[i](d[columns[i]]))
        .attr("cy", d => y[j](d[columns[j]]))
        .attr("r", 4)
        .attr("fill-opacity", 0.8)
        .attr("fill", d => color(d.variety))
        .attr("stroke", "#fff")
        .attr("stroke-width", 0.5)
        .style("cursor", "pointer")
        .on("mouseover", function(event, d) {
          d3.select(this).attr("r", 6).attr("stroke-width", 2).attr("stroke", "#333");
        })
        .on("mouseout", function(event, d) {
          const pointId = `${d["sepal.length"]}-${d["sepal.width"]}-${d["petal.length"]}-${d["petal.width"]}`;
          if (!selectedPoints.has(pointId)) {
            d3.select(this).attr("r", 4).attr("stroke-width", 0.5).attr("stroke", "#fff");
          }
        })
        .on("click", function(event, d) {
          event.stopPropagation();
          console.log("Circle clicked:", d);
          
          // Show popup immediately (primary functionality)
          showPointPopup(d, event);
          
          // Handle selection/highlighting (secondary)
          const pointId = `${d["sepal.length"]}-${d["sepal.width"]}-${d["petal.length"]}-${d["petal.width"]}`;
          if (selectedPoints.has(pointId)) {
            selectedPoints.delete(pointId);
          } else {
            selectedPoints.add(pointId);
          }
          updateHighlights();
        });
    }
  });

  const brush = d3.brush()
    .extent([[padding / 2, padding / 2], [size - padding / 2, size - padding / 2]])
    .on("start brush end", function(event, [i, j]) {
      if (i === j) return;
      
      const selection = event.selection;
      if (selection) {
        const [[x0, y0], [x1, y1]] = selection;
        svg.selectAll("circle").classed("hidden", function(d) {
          const cx = x[i](d[columns[i]]);
          const cy = y[j](d[columns[j]]);
          return cx < x0 || cx > x1 || cy < y0 || cy > y1;
        });
      } else {
        svg.selectAll("circle").classed("hidden", false);
      }
    });

  cell.filter(([i, j]) => i !== j).call(brush);

  const histogramCells = svg.append("g")
    .selectAll("g")
    .data(columns)
    .join("g")
    .attr("transform", (d, i) => `translate(${(columns.length) * size},${i * size})`);

  histogramCells.each(function(column, i) {
    const histogramData = d3.histogram()
      .domain(y[i].domain())
      .thresholds(12)
      (data.map(d => d[column]));

    const xHist = d3.scaleLinear()
      .domain([0, d3.max(histogramData, d => d.length)])
      .range([padding / 2, size - padding / 2]);

    d3.select(this).selectAll("rect")
      .data(histogramData)
      .join("rect")
      .attr("class", "histogram-bar")
      .attr("x", padding / 2)
      .attr("y", d => y[i](d.x1))
      .attr("width", d => xHist(d.length) - xHist(0))
      .attr("height", d => Math.max(0, y[i](d.x0) - y[i](d.x1) - 2))
      .attr("fill", "steelblue")
      .attr("opacity", 0.8)
      .attr("stroke", "#fff")
      .attr("stroke-width", 1)
      .style("cursor", "pointer")
      .on("mouseover", function(event, d) {
        d3.select(this).attr("opacity", 1);
        tooltip.transition().duration(200).style("opacity", .9);
        tooltip.html(`Range: ${d.x0.toFixed(1)} - ${d.x1.toFixed(1)}<br/>Count: ${d.length} flowers`)
          .style("left", (event.pageX + 10) + "px")
          .style("top", (event.pageY - 28) + "px");
      })
      .on("mouseout", function() {
        d3.select(this).attr("opacity", 0.8);
        tooltip.transition().duration(500).style("opacity", 0);
      })
      .on("click", function(event, d) {
        event.stopPropagation();
        const barId = `horizontal-${column}-${d.x0}-${d.x1}`;
        const pointsInBin = data.filter(point => point[column] >= d.x0 && point[column] < d.x1);
        
        if (selectedHistogramBars.has(barId)) {
          selectedHistogramBars.delete(barId);
          d3.select(this).classed("selected", false);
          pointsInBin.forEach(point => {
            const pointId = `${point["sepal.length"]}-${point["sepal.width"]}-${point["petal.length"]}-${point["petal.width"]}`;
            selectedPoints.delete(pointId);
          });
        } else {
          selectedHistogramBars.add(barId);
          d3.select(this).classed("selected", true);
          pointsInBin.forEach(point => {
            const pointId = `${point["sepal.length"]}-${point["sepal.width"]}-${point["petal.length"]}-${point["petal.width"]}`;
            selectedPoints.add(pointId);
          });
        }
        updateHighlights();
      });
  });

  const legendData = [...new Set(data.map(d => d.variety))];
  const legend = svg.append("g")
    .attr("transform", `translate(${width - 150}, 30)`)
    .selectAll("g")
    .data(legendData)
    .join("g")
    .attr("transform", (d, i) => `translate(0, ${i * 25})`);

  legend.append("circle")
    .attr("r", 6)
    .attr("fill", d => color(d))
    .attr("stroke", "#fff")
    .attr("stroke-width", 1);

  legend.append("text")
    .attr("x", 15)
    .attr("dy", "0.35em")
    .style("font-size", "14px")
    .style("font-weight", "bold")
    .text(d => d);
}
</script>

<style scoped>
.dashboard-container {
  padding: 30px;
  max-width: 100%;
  overflow-x: auto;
}

.chart-container {
  background: #ffffff;
  border-radius: 12px;
  padding: 30px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
  height: 100%;
  min-height: 800px;
}

.chart-title {
  text-align: center;
  margin-bottom: 30px;
  color: #2c3e50;
  font-size: 1.5rem;
  font-weight: 700;
  border-bottom: 3px solid #3498db;
  padding-bottom: 15px;
}

.chart {
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: visible;
  min-height: 700px;
}

:global(.tooltip) {
  position: absolute;
  text-align: left;
  padding: 15px;
  font-size: 14px;
  background: rgba(44, 62, 80, 0.95);
  color: white;
  border-radius: 8px;
  pointer-events: none;
  z-index: 1000;
  box-shadow: 0 4px 12px rgba(0,0,0,0.3);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255,255,255,0.1);
}

:global(.tooltip strong) {
  color: #3498db;
  font-size: 16px;
}

:global(svg) {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

:global(.brush .selection) {
  stroke-dasharray: 5,5;
  animation: dash 1s linear infinite;
}

@keyframes dash {
  to {
    stroke-dashoffset: -10;
  }
}
</style>
