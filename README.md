Stockpile Volume Estimation (LiDAR)

Workflow for estimating gravel stockpile volumes from LiDAR: preprocessing in CloudCompare, followed by surface differencing and volume calculation in R (lidR + terra).

⚠️ Note: Results are approximations. No ground-truth checks were performed. The piles sit on a sloped base, so interpret volumes with caution.

Preprocessing (CloudCompare)

Import the LiDAR dataset into CloudCompare (.las or .laz).

Clip the region of interest

Use the scissors or crop tool to keep only the quarry/aggregate area.

Segment piles vs. surrounding ground

Manually or semi-automatically split the dataset into:

piles.las → points belonging to aggregate piles.

surrounding.las → points representing the ground surface around piles.

Save each as a separate LAS file.

(At this point you should have two LAS files: one for piles, one for surrounding ground.)

Analysis (R script)

The R script (scripts/stockpile_volume.R) performs the following steps:

Read data (piles.las and surrounding.las).

Generate DTM (ground surface) from the surrounding points.

Generate DSM (pile surface) from the pile points.

Align rasters to ensure same grid, CRS, and resolution.

Define pile footprint

Use provided polygons (if available)

Or generate a convex hull around the pile points.

Compute Difference of Surfaces (DoD): DSM − DTM.

Mask negative values → keep only positive thickness above ground.

Integrate thickness × cell area to estimate volume in cubic meters.

Output

PNG figures (optional).

CSV summary of volumes.

Usage
Rscript scripts/stockpile_volume.R \
  --piles "/path/to/piles.las" \
  --sur "/path/to/surrounding.las" \
  --outdir "./output" \
  --res 0.5


--piles → LAS file containing pile points.

--sur → LAS file containing surrounding ground points.

--outdir → Output directory (will be created if missing).

--res → Grid resolution in meters (default: 0.5).

--footprints → (Optional) polygon file with pile footprints.

Caveats

Volumes are approximations derived from LiDAR and interpolation.

No field validation was carried out.

A sloped base under the piles may influence results.

Ensure your data is in a metric CRS (X, Y, Z in meters) for correct volumes.
