## What I did with the data

### Bar Chart
Basically just took the 2007 data, sorted countries by life expectancy and picked the top 20. Used different colors for continents.

### Line Chart  
Grouped all the data by continent and calculated average life expectancy for each year. Then just drew lines connecting the points over time.

### Scatter Plot
Used 2007 data again. Put GDP on x-axis and population on y-axis. Made circle size based on life expectancy. Used log scales because the numbers were all over the place.

### Pie Chart
Calculated average life expectancy for each continent in 2007. Made pie slices proportional to those averages. Added percentages to make it clearer.

## Why I made these choices

- Used consistent colors across charts so continents always have same color
- Added tooltips because it's easier to see exact numbers when you hover
- Made titles simple and clear
- Used D3 because that's what the assignment asked for

## Technical stuff

- Loaded CSV data once and reused it for all charts
- Used Vue.js because the template was already set up
- Made it responsive with Bootstrap grid
- Added some basic styling to make it look decent

The charts show different aspects of the same data - current state, trends over time, relationships between variables, and continental comparisons.