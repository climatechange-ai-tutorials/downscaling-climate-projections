# Tutorial Completion Guide

This guide explains what you need to do to complete the **Statistical Downscaling of
Climate Projections with Deep Learning** tutorial. Work through the notebook from top
to bottom because later cells depend on variables and functions created earlier.

## Software used

The notebook installs missing packages automatically. The main packages are:

- `numpy` and `pandas` for arrays and tables
- `xarray` and `netCDF4` for climate data
- `matplotlib` for plots
- `cartopy` for optional coastline maps
- `torch` for the deep-learning model
- `requests` for downloading the data
- `codecarbon` for estimating the energy and emissions from training

Cartopy is optional. If it cannot download coastline data, the climate analysis can
still be completed without coastlines.

## Step 1: Open and prepare the notebook

- [X] Open `Statistical_Downscaling_of_Climate_Projections_with_Deep_Learning.ipynb`.
- [X] If possible, use Google Colab and select a GPU runtime.
- [X] Run the package-installation cell.
- [X] Run the import cell.
- [X] Confirm that the notebook prints the PyTorch version, selected device, and
      whether Cartopy is available.
- [X] Run the configuration cell without changing it for your first attempt.

The default experiment uses:

- ACCESS-CM2 for training and the first projection
- EC-Earth3 as an independent second GCM
- 1961-1980 for training and validation
- 1981-2000 for independent historical testing
- 2080-2099 for future projections
- 50 training epochs

## Step 2: Download and verify the data

- [X] Run the data-download cell.
- [X] Allow the approximately 3.3 GB New Zealand subset to download from Zenodo if
      `NZ_domain` is not already present.
- [X] Wait for extraction to finish.
- [X] Confirm that the required NetCDF files are found and `DATA_ROOT` is set.
- [X] Run the path, loading, mapping, and point-sampling helper cells.

The data include 15 daily large-scale predictor fields on a coarse 16 x 16 grid and
daily high-resolution precipitation on a 128 x 128 grid.

## Step 3: Understand why downscaling is necessary

- [X] Plot the historical mean high-resolution precipitation field.
- [X] Inspect precipitation at the West Coast and Southern Alps points.
- [X] Compare the high-resolution field with the upscaled, GCM-like coarse field.
- [X] Calculate and plot the future-minus-historical precipitation change at both
      resolutions.
- [X] Examine the point comparison bar chart.

[X] Answer **Question 1** in your own words:

> Why do the two locations show similar changes in the coarse result even though the
> high-resolution simulation shows different changes? What spatial information was lost?

You should be able to explain that a roughly 200 km grid cannot resolve narrow terrain,
coastlines, and rain-shadow effects that strongly influence local precipitation.

## Step 4: Load and inspect the training data

- [ ] Load the ACCESS-CM2 training predictors and precipitation target.
- [ ] Inspect their dimensions, coordinates, variables, and units.
- [ ] Plot one day of all 15 predictor fields and its matching precipitation field.
- [ ] Confirm that predictors have shape `(time, 15, 16, 16)` after conversion.
- [ ] Confirm that each precipitation map is flattened to 16,384 output values.

Understand the terminology:

- **Predictors:** wind, humidity, temperature, and geopotential at 850, 700, and
  500 hPa.
- **Predictand:** daily precipitation in mm/day.
- **Pseudo-reality:** high-resolution RCM output used as the tutorial's truth instead
  of observations.

## Step 5: Prepare the data correctly

- [ ] Split 1961-1980 into training years 1961-1976 and validation years 1977-1980.
- [ ] Keep 1981-2000 independent for later testing.
- [ ] Calculate predictor means and standard deviations using training years only.
- [ ] Standardize the training and validation predictors.
- [ ] Convert predictors and precipitation to `float32` arrays and PyTorch tensors.
- [ ] Build the training and validation `DataLoader` objects.
- [ ] Check the printed sample counts and tensor shapes.

The important rule is to avoid information leakage: validation, test, and future data
must not contribute to the training standardization statistics.

## Step 6: Build DeepESD

- [ ] Run the cell defining the `DeepESD` PyTorch class.
- [ ] Run the model-building cell.
- [ ] Inspect the printed architecture and parameter count.
- [ ] Understand that three convolutional layers extract spatial features and the final
      fully connected layer predicts precipitation for all 128 x 128 grid cells.
- [ ] Note that the baseline uses mean squared error (MSE), which may underestimate
      rare heavy-rain events.

## Step 7: Train the model

- [ ] Run the training helper function cells.
- [ ] Train DeepESD for the configured number of epochs.
- [ ] Watch both training and validation MSE.
- [ ] Check whether validation loss improves rather than diverging from training loss.
- [ ] Plot the training and validation loss curves.
- [ ] Confirm that the trained weights are saved to `models/deepesd_pr_nz.pt`.
- [ ] Review the CodeCarbon result in `code_carbon/deepesd_training_emissions.csv` if
      an emissions estimate is available.

Do not judge the model only by its training loss. Validation behaviour shows whether
the learned relationship generalises to unseen years.

## Step 8: Evaluate the historical predictions

- [ ] Run the `downscale` and time-alignment helper cells.
- [ ] Load the independent 1981-2000 perfect predictors and pseudo-reality target.
- [ ] Produce DeepESD predictions for the full historical test period.
- [ ] Calculate the evaluation summary table.
- [ ] Examine RMSE and the biases of mean precipitation, SDII, P98, and RX1day.
- [ ] Compare pseudo-reality and DeepESD maps using the same colour scales.
- [ ] Examine the spatial RMSE and bias maps.

Know what the metrics measure:

- **RMSE:** average day-to-day prediction error
- **Mean:** long-term average precipitation
- **SDII:** average intensity on wet days of at least 1 mm/day
- **P98:** the 98th percentile of daily precipitation
- **RX1day:** average annual maximum one-day precipitation

Answer **Question 2**:

> Which aspects of precipitation does DeepESD reproduce well, which does it struggle
> with, and why is underestimating extremes dangerous for flood planning?

Answer **Question 3**:

> Why should a downscaling model be evaluated with several diagnostics instead of a
> single RMSE value?

You should identify that the baseline generally captures the mean spatial pattern better
than the upper tail and annual extremes. Similar RMSE values can hide very different
errors in wet-day intensity and extreme precipitation.

## Step 9: Examine distribution shift

- [ ] Load the imperfect historical ACCESS-CM2 GCM predictors.
- [ ] Compare their distribution with the perfect predictors used for training.
- [ ] Identify why a GCM's biased inputs may differ from the model's training inputs.
- [ ] Read the note on bias correction or bias adjustment.

The notebook introduces this issue but does not implement bias adjustment. You still need
to understand that applying a model to different input distributions can reduce its skill.

## Step 10: Produce the ACCESS-CM2 projection

- [ ] Downscale historical ACCESS-CM2 GCM predictors.
- [ ] Downscale future ACCESS-CM2 predictors for 2080-2099.
- [ ] Calculate the high-resolution climate-change signal:
      future climatology minus historical climatology.
- [ ] Compare the DeepESD change with the coarse GCM-like change and pseudo-reality.
- [ ] Inspect the maps and the two-location bar chart.
- [ ] Explain how downscaling restores different local changes across nearby locations.

## Step 11: Test transfer to EC-Earth3

- [ ] Apply the same trained DeepESD model to EC-Earth3 without retraining it.
- [ ] Downscale the historical and future EC-Earth3 predictors.
- [ ] Calculate the EC-Earth3 future-minus-historical change signal.
- [ ] Compare its maps and point values with ACCESS-CM2 and pseudo-reality.
- [ ] Identify where the two GCM projections agree and differ.
- [ ] Explain why using more than one GCM is important for representing uncertainty.

Do not interpret success on one second GCM as proof of universal transferability. A new
GCM may contain circulation patterns or biases outside the training distribution.

## Step 12: Complete the final reflection

- [ ] State what statistical downscaling adds to a coarse climate projection.
- [ ] Explain why fine spatial detail does not eliminate climate uncertainty.
- [ ] List the main limitations of this tutorial:
  - pseudo-reality rather than observations
  - MSE underestimation of precipitation extremes
  - GCM predictor bias and distribution shift
  - the stationarity assumption under future warming
  - results from only one region and a small number of GCMs
- [ ] Explain why the results are educational and not ready-made engineering or policy
      values.

## Optional improvement challenge

The following experiments are encouraged but are not required for a first complete run:

- [ ] Increase `NUM_EPOCHS` or tune `LEARNING_RATE`.
- [ ] Increase `FILTERS_LAST_CONV` or modify the model architecture.
- [ ] Train precipitation in `log1p` space or replace MSE with another loss.
- [ ] Add or remove predictor variables or pressure levels.
- [ ] Replace DeepESD with the supplied U-Net example.
- [ ] Try different New Zealand points or another CORDEX ML-Bench region.

If you attempt an improvement, predict whether it will improve the mean, the extremes,
or both before running it. Then repeat the same evaluation and compare the evidence with
your prediction.

## When the tutorial is complete

You have completed the core tutorial when you can check all of the following:

- [ ] The notebook runs from setup through both GCM projections without unresolved errors.
- [ ] DeepESD has been trained and its loss curves inspected.
- [ ] The independent historical evaluation table and maps have been produced.
- [ ] ACCESS-CM2 and EC-Earth3 climate-change signals have been downscaled and compared.
- [ ] Questions 1, 2, and 3 have been answered in your own words.
- [ ] You can explain why downscaling is useful and why RMSE alone is insufficient.
- [ ] You can describe distribution shift, stationarity, GCM uncertainty, and the
      limitations of pseudo-reality.
- [ ] You understand that high-resolution output is not automatically more certain or
      suitable for direct real-world decisions.
