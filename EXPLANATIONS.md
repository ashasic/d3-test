

## What I did

Built a scatterplot matrix for the iris dataset with histograms and interactions.

### Main Features

**Scatterplot Matrix**: Made a 4x4 grid where each cell shows the relationship between two variables. Used d3.cross to generate all combinations of the 4 iris measurements.

**Histograms**: Put histograms on the diagonal instead of self-scatterplots since those don't make sense. Added horizontal histograms on the right side. Used d3.histogram to bin the data.

**Brushing**: Added brush selection using d3.brush. When you brush an area, points outside get hidden with the "hidden" class.

**Point Selection**: Click points to select them. Keep track of selected points with a Set and highlight them across all plots.

**Histogram Interaction**: Click histogram bars to select all points in that range. 

