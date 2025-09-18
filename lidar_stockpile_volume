# Stockpile Volume Estimation (LiDAR / lidR + terra + sf)
# -------------------------------------------------------
# - Build DTM from surrounding-only ground points
# - Build DSM from pile points
# - Align grids, clip to footprint, DoD = DSM - DTM
# - Sum positive DoD to estimate volume (m³)
# Notes: Approximation only; no field verification; piles on sloped base → interpret cautiously.

library(lidR)
library(terra)
library(sf)

# ---- 0) (Optional) output folder & figure switch ----
outdir <- "output"; if (!dir.exists(outdir)) dir.create(outdir, recursive = TRUE)
save_figs <- FALSE  # set TRUE to save PNGs + CSV

# ---- 1) Read data ----
piles <- readLAS("file_path/Aggregate.las")
sur   <- readLAS("file_path/Aggregate_surrounding.las")

# Quick plots (optional)
plot(piles)  # opens 3D viewer
plot(sur)    # surrounding ground-only points

# ---- 2) Ground from surrounding-only (DTM). TIN is robust for terrain. ----
dtm <- rasterize_terrain(sur, res = 0.5, algorithm = tin())
dtm <- terra::rast(dtm)  # ensure terra raster
plot(dtm, main = "Digital Terrain Model (Ground Surface)")

# ---- 3) Pile surface (DSM) from pile points ----
# dsmtin() hugs pile slopes well; p2r() also works if you prefer.
dsm <- rasterize_canopy(piles, res = 0.5, algorithm = dsmtin())
dsm <- terra::rast(dsm)
plot(dsm, main = "Digital Surface Model (Pile Surface)")

# ---- 4) Align grids (same CRS, extent, and resolution) ----
if (!identical(crs(dtm), crs(dsm))) dsm <- project(dsm, crs(dtm))
dsm <- resample(dsm, dtm, method = "bilinear")

# ---- 5) Footprint polygon (convex hull of pile points, quick approximation) ----
xy  <- cbind(piles@data$X, piles@data$Y)
h   <- chull(xy)
hxy <- rbind(xy[h, ], xy[h[1], ])  # close polygon

# Turn into sf polygon (CRS inherited from rasters)
footprint <- st_polygon(list(hxy)) |> 
  st_sfc(crs = st_crs(dtm)) |> 
  st_as_sf()

plot(footprint, main = "Pile Footprint Polygon")

# Clip/mask to footprint to avoid edge artifacts
dtm_fp <- mask(crop(dtm, footprint), footprint)
dsm_fp <- mask(crop(dsm, footprint), footprint)

# ---- 6) Difference of Digital surfaces (DoD): height above ground ----
dod <- dsm_fp - dtm_fp
plot(dod, main = "Difference of Surfaces (DSM - DTM)")

# Keep only positive heights (pile above ground)
dod_pos <- classify(dod, rbind(c(-Inf, 0, NA)))  # set <=0 to NA

# ---- 7) Volume: sum(height) * cell_area (m³) ----
cell_area <- prod(res(dod_pos))  # m² per cell (ensure CRS in meters)
vol_m3 <- global(dod_pos, "sum", na.rm = TRUE)[1,1] * cell_area
cat(sprintf("Estimated pile volume: %.1f m³\n", vol_m3))

# ---- (Optional) Save figures + CSV ----
if (save_figs) {
  png(file.path(outdir, "piles_points_2D.png"), 1600, 1200, res=160)
  plot(xy, pch=".", main="Pile Point Cloud (LiDAR)", xlab="X", ylab="Y"); dev.off()

  png(file.path(outdir, "sur_points.png"), 1600, 1200, res=160)
  plot(sur, main="Surrounding Ground Point Cloud (LiDAR)"); dev.off()

  png(file.path(outdir, "dtm.png"), 1600, 1200, res=160); plot(dtm, main="DTM (Ground)"); dev.off()
  png(file.path(outdir, "dsm.png"), 1600, 1200, res=160); plot(dsm, main="DSM (Pile)");   dev.off()
  png(file.path(outdir, "footprint.png"), 1600, 1200, res=160); plot(footprint, main="Pile Footprint"); dev.off()
  png(file.path(outdir, "dod.png"), 1600, 1200, res=160); plot(dod, main="DoD (DSM - DTM)"); dev.off()
  png(file.path(outdir, "dod_positive.png"), 1600, 1200, res=160); plot(dod_pos, main="Positive DoD (Thickness)"); dev.off()

  write.csv(data.frame(volume_m3 = vol_m3), file.path(outdir, "volume_summary.csv"), row.names = FALSE)
}
