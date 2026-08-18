# Key Learnings: Statistical Downscaling of Climate Projections

## 1. Why downscaling is needed

Global Climate Models (GCMs) simulate the large-scale climate system, but their
typical grid spacing of roughly 100-200 km is too coarse to represent mountains,
coastlines, valleys, rain shadows, and other local features. Nearby places can
therefore have very different real climates while occupying the same GCM grid cell.
Downscaling converts coarse climate information into the finer-scale information
needed for local decisions in areas such as flood planning, water management,
agriculture, and infrastructure.

## 2. Statistical and dynamical downscaling are different approaches

Dynamical downscaling runs a high-resolution Regional Climate Model (RCM). It is
physically detailed but computationally expensive. Statistical downscaling learns
an empirical relationship between large-scale atmospheric conditions and a local
climate variable. Once trained, a statistical model can process many GCMs and
scenarios quickly, but its results depend on the training data and assumptions.

## 3. What this tutorial predicts

The tutorial uses large-scale atmospheric predictors such as wind, humidity,
temperature, and geopotential at several pressure levels to predict daily
precipitation on a finer grid over New Zealand. The input fields are approximately
200 km resolution, while the precipitation target is approximately 10 km resolution.
The model therefore learns how large-scale weather patterns relate to local rainfall.

## 4. The tutorial uses a perfect-prognosis workflow

The model learns a predictor-predictand relationship during a historical period.
It is then driven by historical or future GCM predictors to produce high-resolution
precipitation. The target used for teaching is RCM output, described as
"pseudo-reality," rather than observations. This makes the experiment controlled,
but it is not the same as building an operational product from observed data.

## 5. Data preparation must prevent information leakage

Training, validation, and testing periods have different purposes. Training data
fit the model, validation data monitor generalisation and overfitting, and an
independent test period estimates performance on unseen conditions. Predictor
standardisation must use statistics calculated from the training data only. The
same training statistics must also be used when downscaling new historical or
future GCM inputs.

## 6. DeepESD learns spatial relationships

DeepESD uses convolutional layers to extract spatial features from coarse predictor
maps and a final fully connected layer to produce precipitation at every fine-grid
location. The raw number of parameters is not the whole story: the convolutional
layers learn the transferable spatial representation, while much of the large final
layer acts as local calibration for individual grid cells.

## 7. Training loss does not tell the whole story

The model is trained using mean squared error (MSE) and the Adam optimiser. Training
and validation losses should be compared to detect overfitting. More epochs or a
larger network may improve results, but they increase computation and emissions and
do not guarantee better performance on important climate statistics.

## 8. A single evaluation metric is not enough

RMSE measures average day-to-day error, but it can hide errors that matter for
climate impacts. The tutorial also evaluates:

- Mean precipitation: the long-term average rainfall pattern.
- SDII: rainfall intensity on wet days.
- P98: the upper tail, represented by the 98th percentile.
- RX1day: annual maximum one-day precipitation.
- Spatial bias maps: where the model overpredicts or underpredicts each statistic.

Metrics should be inspected both as domain summaries and as maps. Spatial averaging
can allow positive and negative errors to cancel, so mean absolute bias is useful
when aggregating grid-cell biases.

## 9. Good mean performance does not imply good extremes

DeepESD can reproduce the broad rainfall climatology and New Zealand's west-east
contrast while still underestimating heavy precipitation. MSE is dominated by
common, moderate events and tends to smooth rare extremes. This is especially
important because extreme rainfall, rather than average rainfall, often drives
flood risk and infrastructure design.

## 10. Climate projections focus on change, not only absolute values

A climate-change signal is calculated as a future multi-decadal climatology minus
a historical climatology. Downscaling the historical and future periods separately
and then taking their difference produces a local-scale change signal. The finer
projection can distinguish changes between nearby locations that a coarse GCM treats
as nearly identical.

## 11. Multiple GCMs are needed to explore uncertainty

A trained statistical model can be applied cheaply to several GCMs. Different GCMs
can produce different regional change signals, so agreement from one model should
not be treated as certainty. An ensemble of driving models and scenarios gives a
more useful picture of plausible futures.

## 12. Distribution shift is a major practical problem

Training uses near-perfect RCM predictors, while real projections use imperfect GCM
predictors that contain model-specific biases. Their statistical distributions may
differ from the training inputs. Bias adjustment may be needed before operational
use, although it is not implemented in this tutorial. Transfer to an unseen GCM is
encouraging when it works, but must be evaluated rather than assumed.

## 13. Future use relies on a stationarity assumption

Statistical downscaling assumes that the historical relationship between large-scale
atmospheric predictors and local precipitation remains valid in a warmer future.
Climate change may alter physical relationships or create conditions outside the
training range. Downscaled detail should therefore not be mistaken for certainty or
new physical information guaranteed by the data.

## 14. Responsible use requires more than running the model

Before downscaled results support real decisions, they need evaluation against
observations, testing of extremes and wet-day behaviour, treatment of GCM bias,
multi-model uncertainty analysis, climate-science review, and clear communication of
limitations. A tutorial projection should not be used directly as a flood-design,
insurance, or policy number.

## 15. Useful next experiments

- Train for longer or tune the learning rate and compare validation performance.
- Change model capacity and inspect whether the mean and extremes improve differently.
- Transform precipitation with log1p or try a loss that better represents extremes.
- Add or remove predictor variables and pressure levels.
- Replace DeepESD with a compatible U-Net or another architecture.
- Evaluate on another region or GCM to test transferability.
- Compare results before and after bias adjustment.
- Track whether additional computation delivers meaningful scientific improvement.

## Overall takeaway

Statistical downscaling is a fast and useful bridge between global climate models
and local climate-risk information. Its value comes from learning spatially detailed
relationships and applying them across projections. Its credibility comes from
careful data separation, evaluation of distributions and extremes, testing across
GCMs, and honest treatment of bias, non-stationarity, and uncertainty.
