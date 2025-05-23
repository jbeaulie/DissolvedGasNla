Sensitivity to measurement error in dissolved gas measurements
================
Roy Martin, Jake Beaulieu, Michael McManus
2025-03-05

- [1 Background](#1-background)
- [2 Dissolved concentration](#2-dissolved-concentration)
  - [2.1 Calculate $\text{N}_2\text{O}$](#21-calculate-textn_2texto)
  - [2.2 Empricial variation](#22-empricial-variation)
  - [2.3 Simulate measurment error](#23-simulate-measurment-error)
    - [2.3.1 Equilibrated headspace](#231-equilibrated-headspace)
    - [2.3.2 Ambient air](#232-ambient-air)
    - [2.3.3 Water temperature](#233-water-temperature)
    - [2.3.4 Barometric pressure](#234-barometric-pressure)
    - [2.3.5 Water volume](#235-water-volume)
    - [2.3.6 Mixing ratio](#236-mixing-ratio)
      - [2.3.6.1 Ambient air](#2361-ambient-air)
        - [2.3.6.1.1 $R_{M(low)}$](#23611-r_mlow)
        - [2.3.6.1.2 $R_{M(equal)}$](#23612-r_mequal)
        - [2.3.6.1.3 $R_{M(high)}$](#23613-r_mhigh)
      - [2.3.6.2 Pure gas](#2362-pure-gas)
        - [2.3.6.2.1 $R_{M(low)}$](#23621-r_mlow)
        - [2.3.6.2.2 $R_{M(equal)}$](#23622-r_mequal)
        - [2.3.6.2.3 $R_{M(high)}$](#23623-r_mhigh)
    - [2.3.7 All observables](#237-all-observables)
  - [2.4 Summary of contributions to
    error](#24-summary-of-contributions-to-error)
- [3 Equilibrium concentration](#3-equilibrium-concentration)
  - [3.1 Simulate equilibrium
    $\text{N}_2\text{O}$](#31-simulate-equilibrium-textn_2texto)
  - [3.2 Simulate measurement error](#32-simulate-measurement-error)
    - [3.2.1 Ambient air](#321-ambient-air)
    - [3.2.2 Water temperature](#322-water-temperature)
    - [3.2.3 Barometric pressure](#323-barometric-pressure)
    - [3.2.4 All observables](#324-all-observables)
  - [3.3 Summary of contributions to
    error](#33-summary-of-contributions-to-error)
- [4 Saturation ratio](#4-saturation-ratio)
  - [4.1 Standard GC](#41-standard-gc)
    - [4.1.1 Ambient air, standard
      thermometer](#411-ambient-air-standard-thermometer)
      - [4.1.1.1 Map source-sink status](#4111-map-source-sink-status)
    - [4.1.2 Pure gas, standard
      thermometer](#412-pure-gas-standard-thermometer)
      - [4.1.2.1 Source-sink status](#4121-source-sink-status)
    - [4.1.3 Ambient air, high precision
      thermometer](#413-ambient-air-high-precision-thermometer)
      - [4.1.3.1 Source-sink status](#4131-source-sink-status)
    - [4.1.4 Pure gas, high precision
      thermometer](#414-pure-gas-high-precision-thermometer)
      - [4.1.4.1 Source-sink status](#4141-source-sink-status)
  - [4.2 High precision GC](#42-high-precision-gc)
    - [4.2.1 Pure gas, standard
      thermometer](#421-pure-gas-standard-thermometer)
      - [4.2.1.1 Source-sink status](#4211-source-sink-status)
    - [4.2.2 Pure gas, high precision
      thermometer](#422-pure-gas-high-precision-thermometer)
      - [4.2.2.1 Source-sink status](#4221-source-sink-status)
- [5 MIMS](#5-mims)
  - [5.1 Dissolved concentration](#51-dissolved-concentration)
    - [5.1.1 Standard GC](#511-standard-gc)
    - [5.1.2 High precision GC](#512-high-precision-gc)
  - [5.2 Saturation ratio](#52-saturation-ratio)
    - [5.2.1 Standard GC](#521-standard-gc)
      - [5.2.1.1 Standard field
        temperature](#5211-standard-field-temperature)
        - [5.2.1.1.1 Source-sink status](#52111-source-sink-status)
      - [5.2.1.2 High precision
        temperature](#5212-high-precision-temperature)
        - [5.2.1.2.1 Source-sink status](#52121-source-sink-status)
    - [5.2.2 High precision GC](#522-high-precision-gc)
      - [5.2.2.1 Standard temperature](#5221-standard-temperature)
        - [5.2.2.1.1 Source-sink status](#52211-source-sink-status)
      - [5.2.2.2 High precision
        temperature](#5222-high-precision-temperature)
        - [5.2.2.2.1 Source-sink status](#52221-source-sink-status)
- [6 Summary of ratio results](#6-summary-of-ratio-results)
- [7 Update data with uncertainty
  info](#7-update-data-with-uncertainty-info)
- [8 Session Info](#8-session-info)

``` r
library(knitr)
library(rmarkdown)
library(bayesplot)
library(kableExtra)
library(ggpubr)
library(tidyverse)
library(tidybayes)
library(ggplot2)
library(ggExtra)
library(gridExtra)
library(readxl)
library(janitor)
library(sf)
library(usmap)

# Identify local path for each user
localPath <- Sys.getenv("USERPROFILE")

# Define helper functions
# standardized formatting for column names
toEPA <- function(X1){
  names(X1) = tolower(names(X1))
  names(X1) = gsub(pattern = c("\\(| |#|)|/|-|\\+|:|_"), replacement = ".", x = names(X1))
  X1
}

options(max.print = 2000)
```

# 1 Background

In this document we explore a range of scenarios wherein measurement
errors could arise in methods for determining the concentration of
$\text{N}_2\text{O}$ from field and laboratory measurements observed as
part of the 2017 National Lakes Assessment (NLA).

# 2 Dissolved concentration

The equation for estimating the concentration (nM) of
$\text{N}_2\text{O}$ gas dissolved in water is:

$$C = B^{-6} \cdot \left(\frac{V_g(M_h - M_r)}{(8.3144598 \cdot K \cdot V_w) + (H^{\theta} \cdot e^{2700 \cdot ( \frac{1}{K} - \frac{1}{298.15})} \cdot M_h)} \right)$$

where:

- $C$ = concentration (nM) dissolved in water
- $B$ = barometric pressure (kPa)
- $V_g$ = volume of reference gas in headspace (mL)
- $M_h$ = gas measured in the equilibrated headspace
- $M_r$ = gas measured in the reference
- $V_w$ = volume of water below the headspace (mL)
- $K$ = temperature (K) (T_c + 273 below)
- $H^{\theta}$ = Henry’s law constant ($H_{N_2O}$ or $H_{Ar}$ below)

and:

$$\begin{split}
 V & = V_g + V_w \\
  V_g & = V \cdot R_M \\
  V_w & = V \cdot (1 - R_M)
  \end{split}$$

where:

- $V$ = Total volume of the mixing vessel (e.g. 140mL syringe)
- $R_M$ = Gas mixing proportion (gas volume / mixing vessel volume)
- $V_g$ = Volume of gas (mL) in the mixing vessel
- $V_w$ = Volume of water (mL) in the mixing vessel

## 2.1 Calculate $\text{N}_2\text{O}$

Below is a summary of the empirical measurements of N2O (ppm) in the
equilibrated headspace.

``` r
load("./../inputData/dg.RData")

# summary of measured N2O ppm in equilibrated headspace gas
dg %>%
  group_by(sample.source) %>%
  summarize(mean_n2O = mean(n2o.ppm.gc, na.rm = TRUE),
          median_n2o = median(n2o.ppm.gc, na.rm = TRUE)) #mean = 0.36, median = 0.31
```

    ## # A tibble: 2 × 3
    ##   sample.source mean_n2O median_n2o
    ##   <chr>            <dbl>      <dbl>
    ## 1 AIR              0.309      0.310
    ## 2 DG               0.362      0.307

Calculate a single concentration without measurement error to represent
a “true” value for dissolved $\text{N}_2\text{O}$. The median of the
empirical measurements was used for values of $M_h$ and $M_r$ (M_a_n2o).

``` r
# Set volume of syringe/vessel
V <- 140 # 140mL syringe

# Set ratio of gas volume to water volume in the mixing vessel
R_M <- 0.25

B <- 99 # barometric pressure (kPa)
V_g <- V * R_M # volume of reference gas in headspace
V_w <- V * (1 - R_M) # volume of water below headspace
T_c <- 23 # temperature ( C )
M_h <- 0.307 # gas measured in headspace; Changes as function of R_M at constant C
M_a_n2o <- 0.310 # gas measured in reference (air)
H_n2o <- 0.00024 #mol m-3 Pa, range: 0.00018 - 0.00025; Henry's law constant
H_ar <- 0.000014 #mol m-3 Pa, range: 0.000013 - 0.000016; Henry's law constant

C <- 1e-6 * B * (V_g * (M_h - M_a_n2o) / (8.3144598 * (T_c + 273.15) * V_w) + H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15)) * M_h) 

C <- C * 1e9 #convert mol to nmol

round(C, 1) 
```

    ## [1] 7.7

``` r
#### Re-arrange for Mh
M_h <- (((C / 1e9) * 8.3144598 * (T_c + 273.15) * V_w) + (1e-6 * B * V_g * M_a_n2o)) / (1e-6 * B * (V_g + 8.3144598 * (T_c + 273.15) * V_w * H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15))))

M_h
```

    ## [1] 0.307

## 2.2 Empricial variation

Measurement error for this analysis is assumed to be the standard
deviation of repeated analyses of the analytical standards throughout
the period of time that the NLA2017 samples were analyzed, which was
$\sigma$ = 0.0078725.

## 2.3 Simulate measurment error

Below, potential errors in GC measurements for are simulated and
assessed s eparately for measurements of $\text{N}_2\text{O}$ in:

- 1)  the equilibrated headspace,
- 2)  the air, and
- 3)  the headspace + air

Error is then simulated and assessed separately for measurements of:

- 4)  water temperature,
- 5)  barometric pressure, and
- 6)  water volume

Finally, all sources of error (1, 2, 4, 5, and 6) were simulated
simultaneously and assessed as an estimate of the total variation in
$\text{N}_2\text{O}$ measurements. The estimated absolute error was
added to each of the empirical point measurements recorded in this
study, which resulted in an estimate of uncertainty due to measurement
error for each empirical observation. The errors were also carried
through for estimates of the saturation ratio and a categorical “source”
or “sink” status was assigned for each site. This status was assigned
such that any observations where the central 95th percentile of the
measurement plus simulated error included 1.0 were deemed to be of
“undeterminded” status. The source/sink/undetermined status for each
site was then mapped.

Set the number of simulations for estimating error:

``` r
nsim <- 1e5
```

### 2.3.1 Equilibrated headspace

Simulate measurement error in dissolved $\text{N}_2\text{O}$ due solely
to variation in GC measurements of gas in the equlibrated headspace
($M_h$ above).

``` r
# Create empty vector to fill below with simulated values for N2O observed with 
# error due to variation in the GC measurments of gas in the equilibrated 
# headspace. The subscript, "me", denotes an observed or measured value, which 
# is contrasted with the known value, C, set above.
C_h_me <- rep(NA, nsim)  

# The true value for the concentration of equilibrated gas in the headspace, 
# M_h, was set above. This step simulates additive error in this measurement 
# that is centered over the true value. The observed concentration of
# equilibrated gas, M_h_me, is simulated as normally distributed variable, with a mean
# M_h, and a standard deviation specified as the observed sd of replicate GC 
# measurements in our lab. 
M_h_me <- rnorm(nsim, M_h, n2o_sd_epa) 

for(i in 1:nsim){
  C_h_me[i] <- 1e-6 * B * (V_g * (M_h_me[i] - M_a_n2o) / (8.3144598 * (T_c + 273.15) * V_w) + H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15)) * M_h_me[i])
}

# combine into dataframe for ggplot. 
C_h_df <- tibble(C_h_me = C_h_me, M_h_me = M_h_me)
# save absolute error for plot
C_h_me_abs <- (C_h_me * 1e9) - C
```

The results of the simulations are summarized using density histograms
below.
<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_C_h_me-1.png" style="display: block; margin: auto;" />

### 2.3.2 Ambient air

Error in the GC measurements of $\text{N}_2\text{O}$ in the ambient air
($M_r$ above) reference is simulated below. The result of the simulation
is plotted as a histogram to illustrate the variation in the
measurement. The resulting effect on the uncertainty in the simulated
dissolved N2O observation is plotted along with the absolute error in
the simulated measurement.

``` r
C_a_me <- rep(NA, nsim)

M_a_n2o_me <- rnorm(nsim, M_a_n2o, n2o_sd_epa) # m.e. measured gas in air

for(i in 1:nsim){
  C_a_me[i] <- 1e-6 * B * (V_g * (M_h - M_a_n2o_me[i]) / (8.3144598 * (T_c + 273.15) * V_w) + H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15)) * M_h)
}

# combine into dataframe for ggplot. qplot, which accepts vectors,
# was deprecated in ggplot 3.4.0
C_a_df <- tibble(C_a_me = C_a_me, M_a_n2o_me = M_a_n2o_me)
# save absolute error for plot
C_a_me_abs<- (C_a_me * 1e9) - C
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_Ca-1.png" style="display: block; margin: auto;" />

### 2.3.3 Water temperature

Next, error in water temperature measurements is simulated and its
potential effect on the final dissolved N2O concentration assessed.

``` r
C_tc_me <- rep(NA, nsim)

# How to get standard deviation of temperature measurement?
# calibration certificate reports an uncertainty of +/-0.058C
# take standard deviation of simulated measurements with specified uncertainty
T_c_sd <- sd(runif(n = nsim, min = T_c - 0.058, max = T_c + 0.058)) # calibration certificate reports an uncertainty of +/-0.058C
T_c_me <- rnorm(nsim, T_c, T_c_sd) #m.e. in measured water temp

for(i in 1:nsim){
  C_tc_me[i] <- 1e-6 * B * (V_g * (M_h - M_a_n2o) / (8.3144598 * (T_c_me[i] + 273.15) * V_w) + H_n2o * exp(2700 * (1 / (T_c_me[i] + 273.15) - 1 / 298.15)) * M_h)
}

# combine into dataframe for ggplot. qplot, which accepts vectors,
# was deprecated in ggplot 3.4.0
C_tc_df <- tibble(C_tc_me = C_tc_me, T_c_me = T_c_me)
# save absolute error for plot
C_tc_me_abs <- (C_tc_me * 1e9) - C
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_T_c-1.png" style="display: block; margin: auto;" />

### 2.3.4 Barometric pressure

Next, error in barometric pressure measurements is simulated and its
potential effect on the final dissolved N2O concentration assessed.

``` r
C_b_me <- rep(NA, nsim)

# How to get standard deviation of barometric pressure?
# The manual reports an "accuracy" of +/-3 mm Hg (+/-0.4kPa), but EPA calibration
# checks performed in the AWBERC metrology lab demonstrate much greater precision.
# below we calculat sd of replicate measurements made with 15 different barometers.
# we then take the mean of the 15 sd values.

#2023 barometer verification in AWBERC metrology lab
B_sd <- mean(
  sd(c(999,998.6)*0.1), # YSI ProDSS
  sd(c(1000.848, 1000.848)*0.1), # EXO 21G104022
  sd(c(1000.982, 1000.982)*0.1), # EXO 18H111572
  sd(c(1000.982, 1000.982)*0.1), # EXO 22F105975
  sd(c(1001.382, 1001.382)*0.1), # EXO 21K100162
  sd(c(1000.98, 1000.85)*0.1), # EXO 21F103579
  sd(c(1000.31, 1000.44)*0.1), # MDS 16D100327
  sd(c(1001.25, 1001.25)*0.1), # MDS 12F102618
  sd(c(1000, 1000)*0.1), # MDS 16D100328
  sd(c(1002, 1002)*0.1), # MDS 02K599 AF
  sd(c(1003,1003)*0.1), # MDS 12J0793 AF
  sd(c(1000.85,1000.85)*0.1), # MDS 16E100831
  sd(c(1000.32, 1000.32)*0.1), # MDS 12F102619
  sd(c(1002, 1002)*0.1)) # MDS 15F104725

B_me <- rnorm(nsim, B, B_sd) #m.e. in measured barometric pressure

for(i in 1:nsim){
  C_b_me[i] <- 1e-6 * B_me[i] * (V_g * (M_h - M_a_n2o) / (8.3144598 * (T_c + 273.15) * V_w) + H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15)) * M_h)
}

# combine into dataframe for ggplot. qplot, which accepts vectors,
# was deprecated in ggplot 3.4.0
C_b_df <- tibble(C_b_me = C_b_me, B_me = B_me)
# save absolute error for plot
C_b_me_abs <- (C_b_me * 1e9) - C
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_B-1.png" style="display: block; margin: auto;" />

### 2.3.5 Water volume

Next, error in the volume of water (relative to volume of gas) in the
sample vessel is simulated and its potential effect on the final
dissolved N2O concentration assessed.

``` r
C_w_me <- rep(NA, nsim)

V_w_me <- rnorm(nsim, V_w, 1) #m.e. in measured water volume
V_g_me <- 140 - V_w_me

for(i in 1:nsim){
  C_w_me[i] <- 1e-6 * B * (V_g_me[i] * (M_h - M_a_n2o) / (8.3144598 * (T_c + 273.15) * V_w_me[i] ) + H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15)) * M_h)
}

# combine into dataframe for ggplot. qplot, which accepts vectors,
# was deprecated in ggplot 3.4.0
C_w_df <- tibble(C_w_me = C_w_me, V_w_me = V_w_me)
# save absolute error for plot
C_w_me_abs <- (C_w_me * 1e9) - C
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_V_w-1.png" style="display: block; margin: auto;" />

### 2.3.6 Mixing ratio

To check the effect of different mixing ratios, we re-arrange the
equation to solve for $M_h$, given a known $C$ from above where $C =$
7.7. For both ambient air and a pure gas (e.g., helium) as the headspace
gas, we evaluate three scenarios for the mixing ratio, $R_M$. The $R_M$
value indicates the proportion of gas relative to water in the headspace
mixture. In the simulations below, we set $R_M$ to be (1) very low
($R_{M(low)} = 0.1$), (2) at equal mixing proportions
($R_{M(equal)} = 0.5$), and (3) very high ($R_{M(high)} = 0.9$). For
reference, in the simulations above, and in the empirical field
measurments, $R_M$ was set at 0.25. Also note that the total volume of
the mixing vessel in the field study was 140mL, so the volume of gas in
the mixing vessel was intended to be $R_M \cdot V_g =$ 35mL and the
volume of water was $R_M \cdot V_w =$ 105mL. Finally, note that varying
the total volume of the mixing vessel has no consequence for $C$, only
the ratio of gas to water.

#### 2.3.6.1 Ambient air

For the same fixed concentration, $C =$ 7.7, using a low proportion of
gas relative to water ($R_{M(low)} = 0.1$) in the equilibrated headspace
results in a smaller value of $M_h$ relative to the simulations above
with $R_M =$ 0.25. This simulation assumes ambient air is used as the
headspace gas.

``` r
# Set volume of syringe/vessel
V <- 140 # 140mL syringe

# Set ratio of gas volume to water volume in the mixing vessel to something
# smaller than the scenario above
R_M_low <- 0.1

C <- C # known N2O concentration in sample from simulation above
B <- 99 # barometric pressure (kPa)
V_g_low <- V * R_M_low # volume of reference gas in headspace
V_w_low <- V * (1 - R_M_low) # volume of water below headspace
T_c <- 23 # temperature ( C )
M_a_n2o <- 0.310 # gas measured in reference (air)
H_n2o <- 0.00024 #mol m-3 Pa, range: 0.00018 - 0.00025; Henry's law constant

#### Re-arranged previous equation to solve for Mh (with low proportion of gas to water)
M_h_low <- (((C / 1e9) * 8.3144598 * (T_c + 273.15) * V_w_low) + (1e-6 * B * V_g_low * M_a_n2o)) / (1e-6 * B * (V_g_low + 8.3144598 * (T_c + 273.15) * V_w_low * H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15))))

round(M_h_low, 3)
```

    ## [1] 0.306

Next, we calculate $M_h$ for equal proportions of gas and water in the
headspace, such that $R_{M(equal)} = 0.5$. In this case we get a
slightly higher value for $M_h$ relative to the low mixing proportion
above.

``` r
# Set volume of syringe/vessel
V <- 140 # 140mL syringe

# Set ratio of gas volume to water volume in the mixing vessel to something
# smaller than the scenario above
R_M_equal <- 0.5

C <- C # known N2O concentration in sample from simulation above
B <- 99 # barometric pressure (kPa)
V_g_equal <- V * R_M_equal # volume of reference gas in headspace
V_w_equal <- V * (1 - R_M_equal) # volume of water below headspace
T_c <- 23 # temperature ( C )
M_a_n2o <- 0.310 # gas measured in reference (air)
H_n2o <- 0.00024 #mol m-3 Pa, range: 0.00018 - 0.00025; Henry's law constant

#### Re-arranged previous equation to solve for Mh
M_h_equal <- (((C / 1e9) * 8.3144598 * (T_c + 273.15) * V_w_equal) + (1e-6 * B * V_g_equal * M_a_n2o)) / (1e-6 * B * (V_g_equal + 8.3144598 * (T_c + 273.15) * V_w_equal * H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15))))

round(M_h_equal, 3)
```

    ## [1] 0.308

Next, we calculate $M_h$ for a higher proportion of gas relative to
water in the headspace, such that $R_{M(high)} = 0.9$. In this case we
again get a higher value for $M_h$.

``` r
# Set volume of syringe/vessel
V <- 140 # 140mL syringe

# Set ratio of gas volume to water volume in the mixing vessel to something
# smaller than the scenario above
R_M_high <- 0.9

C <- C # known N2O concentration in sample from simulation above
B <- 99 # barometric pressure (kPa)
V_g_high <- V * R_M_high # volume of reference gas in headspace
V_w_high <- V * (1 - R_M_high) # volume of water below headspace
T_c <- 23 # temperature ( C )
M_a_n2o <- 0.310 # gas measured in reference (air)
H_n2o <- 0.00024 #mol m-3 Pa, range: 0.00018 - 0.00025; Henry's law constant

#### Re-arranged previous equation to solve for Mh
M_h_high <- (((C / 1e9) * 8.3144598 * (T_c + 273.15) * V_w_high) + (1e-6 * B * V_g_high * M_a_n2o)) / (1e-6 * B * (V_g_high + 8.3144598 * (T_c + 273.15) * V_w_high * H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15))))

round(M_h_high, 3)
```

    ## [1] 0.31

Finally, we simulate $C$ again with measurment error on $M_h$, but using
the values of $M_h$ obtained above using the 3 different values of
$R_M$.

##### 2.3.6.1.1 $R_{M(low)}$

``` r
C_h_me_low <- rep(NA, nsim)  

M_h_me_low <- rnorm(nsim, M_h_low, n2o_sd_epa) 

for(i in 1:nsim){
  C_h_me_low[i] <- 1e-6 * B * (V_g_low * (M_h_me_low[i] - M_a_n2o) / (8.3144598 * (T_c + 273.15) * V_w_low) + H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15)) * M_h_me_low[i])
}

# combine into dataframe for ggplot. 
C_h_low_df <- tibble(C_h_me_low = C_h_me_low, M_h_me_low = M_h_me_low)
# save absolute error for plot
C_h_me_low_abs <- (C_h_me_low * 1e9) - C
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_C_h_me_low-1.png" style="display: block; margin: auto;" />

##### 2.3.6.1.2 $R_{M(equal)}$

``` r
C_h_me_equal <- rep(NA, nsim)  

M_h_me_equal <- rnorm(nsim, M_h_equal, n2o_sd_epa) 

for(i in 1:nsim){
  C_h_me_equal[i] <- 1e-6 * B * (V_g_equal * (M_h_me_equal[i] - M_a_n2o) / (8.3144598 * (T_c + 273.15) * V_w_equal) + H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15)) * M_h_me_equal[i])
}

# combine into dataframe for ggplot. 
C_h_equal_df <- tibble(C_h_me_equal = C_h_me_equal, M_h_me_equal = M_h_me_equal)
# save absolute error for plot
C_h_me_equal_abs <- (C_h_me_equal * 1e9) - C
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_C_h_me_equal-1.png" style="display: block; margin: auto;" />

##### 2.3.6.1.3 $R_{M(high)}$

``` r
C_h_me_high <- rep(NA, nsim)  

M_h_me_high <- rnorm(nsim, M_h_high, n2o_sd_epa) 

for(i in 1:nsim){
  C_h_me_high[i] <- 1e-6 * B * (V_g_high * (M_h_me_high[i] - M_a_n2o) / (8.3144598 * (T_c + 273.15) * V_w_high) + H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15)) * M_h_me_high[i])
}

# combine into dataframe for ggplot. 
C_h_high_df <- tibble(C_h_me_high = C_h_me_high, M_h_me_high = M_h_me_high)
# save absolute error for plot
C_h_me_high_abs <- (C_h_me_high * 1e9) - C
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_C_h_me_high-1.png" style="display: block; margin: auto;" />

#### 2.3.6.2 Pure gas

To check the effect of different mixing ratios using a pure gas such as
helium as the headspace, we again re-arrange the equation to solve for
$M_h$, given known $C$. In this case with a pure gas, we set $M_a = 0$.

For the same fixed concentration, $C$, using a low proportion of pure
gas relative to water ($R_{M(low)} = 0.1$) in the equilibrated headspace
results in a much smaller value of $M_h$ relative to the original
simulations (and field method) where $R_M =$ 0.25. This is because the
mixing ratio is very low and also because there is no need to account
for ambient air in the headspace.

``` r
# Set volume of syringe/vessel
V <- 140 # 140mL syringe

# Set ratio of gas volume to water volume in the mixing vessel to something
# smaller than the scenario above
R_M_low <- 0.1

C <- C # known N2O concentration in sample from simulation above
B <- 99 # barometric pressure (kPa)
V_g_low <- V * R_M_low # volume of reference gas in headspace
V_w_low <- V * (1 - R_M_low) # volume of water below headspace
T_c <- 23 # temperature ( C )
H_n2o <- 0.00024 #mol m-3 Pa, range: 0.00018 - 0.00025; Henry's law constant

#### Re-arranged previous equation to solve for Mh
M_h_pg_low <- (((C / 1e9) * 8.3144598 * (T_c + 273.15) * V_w_low) + (1e-6 * B * V_g_low * 0)) / (1e-6 * B * (V_g_low + 8.3144598 * (T_c + 273.15) * V_w_low * H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15))))

round(M_h_pg_low, 3)
```

    ## [1] 0.26

Next, we calculate $M_h$ for equal proportions of pure gas and water in
the headspace, such that $R_{M(equal)} = 0.5$. In this case we get a
substantially lower value for $M_h$ relative to the low mixing
proportion above.

``` r
# Set volume of syringe/vessel
V <- 140 # 140mL syringe

# Set ratio of gas volume to water volume in the mixing vessel to something
# smaller than the scenario above
R_M_equal <- 0.5

C <- C # known N2O concentration in sample from simulation above
B <- 99 # barometric pressure (kPa)
V_g_equal <- V * R_M_equal # volume of reference gas in headspace
V_w_equal <- V * (1 - R_M_equal) # volume of water below headspace
T_c <- 23 # temperature ( C )
H_n2o <- 0.00024 #mol m-3 Pa, range: 0.00018 - 0.00025; Henry's law constant

#### Re-arranged previous equation to solve for Mh
M_h_pg_equal <- (((C / 1e9) * 8.3144598 * (T_c + 273.15) * V_w_equal) + (1e-6 * B * V_g_equal * 0)) / (1e-6 * B * (V_g_equal + 8.3144598 * (T_c + 273.15) * V_w_equal * H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15))))

round(M_h_pg_equal, 3)
```

    ## [1] 0.118

Next, we calculate $M_h$ for a higher proportion of pure gas relative to
water in the headspace, such that $R_{M(high)} = 0.9$. In this case we
again get a very low value for $M_h$.

``` r
# Set volume of syringe/vessel
V <- 140 # 140mL syringe

# Set ratio of gas volume to water volume in the mixing vessel to something
# smaller than the scenario above
R_M_high <- 0.9

C <- C # known N2O concentration in sample from simulation above
B <- 99 # barometric pressure (kPa)
V_g_high <- V * R_M_high # volume of reference gas in headspace
V_w_high <- V * (1 - R_M_high) # volume of water below headspace
T_c <- 23 # temperature ( C )
H_n2o <- 0.00024 #mol m-3 Pa, range: 0.00018 - 0.00025; Henry's law constant

#### Re-arranged previous equation to solve for Mh
M_h_pg_high <- (((C / 1e9) * 8.3144598 * (T_c + 273.15) * V_w_high) + (1e-6 * B * V_g_high * 0)) / (1e-6 * B * (V_g_high + 8.3144598 * (T_c + 273.15) * V_w_high * H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15))))

round(M_h_pg_high, 3)
```

    ## [1] 0.02

Below, we again simulate $C$ with measurment error on $M_h$, using the
values of $M_h$ obtained above using the 3 different values of $R_M$
with the hypothetical pure gas headspace mixture.

##### 2.3.6.2.1 $R_{M(low)}$

``` r
C_h_pg_me_low <- rep(NA, nsim)  

M_h_pg_me_low <- rnorm(nsim, M_h_pg_low, n2o_sd_epa) 

for(i in 1:nsim){
  C_h_pg_me_low[i] <- 1e-6 * B * (V_g_low * (M_h_pg_me_low[i] - 0) / (8.3144598 * (T_c + 273.15) * V_w_low) + H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15)) * M_h_pg_me_low[i])
}

# combine into dataframe for ggplot. 
C_h_pg_low_df <- tibble(C_h_pg_me_low = C_h_pg_me_low, M_h_pg_me_low = M_h_pg_me_low)
# save absolute error for plot
C_h_pg_me_low_abs <- (C_h_pg_me_low * 1e9) - C
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_C_h_me_pg_low-1.png" style="display: block; margin: auto;" />

##### 2.3.6.2.2 $R_{M(equal)}$

``` r
C_h_pg_me_equal <- rep(NA, nsim)  

M_h_pg_me_equal <- rnorm(nsim, M_h_pg_equal, n2o_sd_epa) 

for(i in 1:nsim){
  C_h_pg_me_equal[i] <- 1e-6 * B * (V_g_equal * (M_h_pg_me_equal[i] - 0) / (8.3144598 * (T_c + 273.15) * V_w_equal) + H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15)) * M_h_pg_me_equal[i])
}

# combine into dataframe for ggplot. 
C_h_pg_equal_df <- tibble(C_h_pg_me_equal = C_h_pg_me_equal, M_h_pg_me_equal = M_h_pg_me_equal)
# save absolute error for plot
C_h_pg_me_equal_abs <- (C_h_pg_me_equal * 1e9) - C
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_C_h_me_pg_equal-1.png" style="display: block; margin: auto;" />

##### 2.3.6.2.3 $R_{M(high)}$

``` r
C_h_pg_me_high <- rep(NA, nsim)  

M_h_pg_me_high <- rnorm(nsim, M_h_pg_high, n2o_sd_epa) 

for(i in 1:nsim){
  C_h_pg_me_high[i] <- 1e-6 * B * (V_g_high * (M_h_pg_me_high[i] - 0) / (8.3144598 * (T_c + 273.15) * V_w_high) + H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15)) * M_h_pg_me_high[i])
}

# combine into dataframe for ggplot. 
C_h_pg_high_df <- tibble(C_h_pg_me_high = C_h_pg_me_high, M_h_pg_me_high = M_h_pg_me_high)
# save absolute error for plot
C_h_pg_me_high_abs <- (C_h_pg_me_high * 1e9) - C
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_C_h_me_pg_high-1.png" style="display: block; margin: auto;" />

### 2.3.7 All observables

In order to get a sense of the total error in a measurement resulting
from the sources above, we simulate measurement error for all of the
observables (*i.e.*, GC measure of gas in equilibrated headspace, GC
measure of gas in air, water temperature, and barometric pressure) and
assess the resulting effects on the dissolved N2O concentration measure.
In this scenario, it is assumed that, as with the empirical field
measurements, air is used as the headspace gas and the mixing ratio is
\$R_M = \$ 0.25.

``` r
C_all_me <- rep(NA, nsim)

for(i in 1:nsim){
  C_all_me[i] <- 1e-6 * B_me[i] * (V_g_me[i] * (M_h_me[i] - M_a_n2o_me[i] ) / (8.3144598 * (T_c_me[i] + 273.15) * V_w_me[i]) + H_n2o * exp(2700 * (1 / (T_c_me[i] + 273.15) - 1 / 298.15)) * M_h_me[i])
}

# combine into dataframe for ggplot. qplot, which accepts vectors,
# was deprecated in ggplot 3.4.0
C_all_df <- tibble(C_all_me = C_all_me)
# save absolute error for plot
C_all_me_abs <- (C_all_me * 1e9) - C
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_all-1.png" style="display: block; margin: auto;" />

The estimated standard deviations of dissolved N2O measurements and the
resulting 95% credible interval for error in measurements is printed
below.

``` r
# standard deviation of a measurement is:
sd(C_all_me* 1e9) # 0.20 nmol L-1
```

    ## [1] 0.3230261

``` r
# 2 standard deviations
sd(C_all_me * 1e9) * 1.96 # +/- 0.395 nmol N2O
```

    ## [1] 0.6331311

Alternatively, calculate the central 95th percentile of the simulated
measures.

``` r
# standard deviation of a measurement is:
round(sd(C_all_me_abs), 3) # 0.20 nmol L-1
```

    ## [1] 0.323

``` r
# the 95% CI for an observation with measurement error can be computed 
# approximately as the value +/- 2 standard deviations; or the central 95th 
# percentile of the absolute error distribution.
round(quantile((C + C_all_me_abs), probs = c(0.025, 0.975)), 3) # +/- nmol N2O
```

    ##  2.5% 97.5% 
    ## 7.080 8.347

## 2.4 Summary of contributions to error

Below, we compare the total error estimated for all observables in the
last simulation, to the error estimated from individual sources.

``` r
results_dissolved_N2O <- tibble(Eh_me= C_h_me_abs,
                                Air_me = C_a_me_abs,
                                Tc_me = C_tc_me_abs,
                                B_me = C_b_me_abs,
                                W_me = C_w_me_abs,
                                All_me = C_all_me_abs) %>%
  pivot_longer(cols = ends_with("me"), names_to = "Source", values_to = "Error")


results_dissolved_N2O %>%
  mutate(Source = factor(Source)) %>%
  mutate(Source = fct_relevel(Source, c("B_me",
                                        "W_me",
                                        "Tc_me",
                                        "Air_me",
                                        "Eh_me",
                                        "All_me"
                                        ))) %>%
  ggplot(aes(x = Error, y = Source)) +
  stat_pointinterval() +
  scale_y_discrete(labels=c("Eh_me" = "Equilibrated headspace",
                            "Air_me" = "Ambient air",
                            "Tc_me" = "Water temperature",
                            "B_me" = "Barometric pressure",
                            "W_me" = "Water volume",
                            "All_me" = "All sources")) +
  xlab("Absolute error in dissolved \nN2O estimate (measured - true) \n") +
  ylab("") 
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/final_figure_N2O_diss-1.png" style="display: block; margin: auto;" />

# 3 Equilibrium concentration

The equation for estimating the concentration (nM) of
($\text{N}_2\text{O}$) gas dissolved in water that is in equilibrium
with the atmosphere is:

$$C_{eq} = 1^{-6} \cdot B \cdot C_a \cdot H^{\theta} \cdot e^{2700 \cdot \frac{1}{Tc} -  \frac{1}{298.15}}$$
where:

- $C_{eq}$ = The equilibrium concentration in nM
- $B$ = barometric pressure (kPa)
- $C_a$ = The air concentration in nM
- $H^{\theta}$ = Henry’s law constant ($H_{N_2O}$ below)
- $T_c$ = temperature (K) = T_c (Celsius) + 273.15

## 3.1 Simulate equilibrium $\text{N}_2\text{O}$

A “known” concentration is calculated:

``` r
C_eq <- M_a_n2o * B * 1e-6 * (H_n2o  * exp(2700*(1 / (T_c + 273.15) - 1 / 298.15))) 

C_eq <- C_eq * 1e9 # convert mol to nmol
```

## 3.2 Simulate measurement error

### 3.2.1 Ambient air

Error in the GC measurements of $\text{N}_2\text{O}$ in the ambient air
($M_a$) is simulated below. The resulting effects on the uncertainty of
the equilibrium N2O measurement is plotted along with the absolute error
in N2O as a result of this measurement.

``` r
C_a_eq_me <- rep( NA, nsim )

M_a_n2o_me <- rnorm( nsim, M_a_n2o, n2o_sd_epa ) #m.e. in measured gas in air


for(i in 1:nsim){
    C_a_eq_me[i] <- M_a_n2o_me[i] * B * 1e-6 * (H_n2o  * exp(2700*(1 / (T_c + 273.15) - 1 / 298.15)))
}

C_a_eq_df <- tibble(C_a_eq_me = C_a_eq_me, M_a_n2o_me = M_a_n2o_me)
# save absolute error for plot
C_a_eq_me_abs <- (C_a_eq_me * 1e9) - C_eq
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_sat_C_a_n2o-1.png" style="display: block; margin: auto;" />

### 3.2.2 Water temperature

``` r
C_tc_eq_me <- rep(NA, nsim)

for(i in 1:nsim){
    C_tc_eq_me[i] <- M_a_n2o * B * 1e-6 * (H_n2o  * exp(2700 * (1 / (T_c_me[i] + 273.15) - 1/298.15)))
}

# combine into dataframe for ggplot. qplot, which accepts vectors,
# was deprecated in ggplot 3.4.0
C_tc_eq_df <- tibble(C_tc_eq_me = C_tc_eq_me, T_c_me = T_c_me)
# save absolute error for plot
C_tc_eq_me_abs <- (C_tc_eq_me * 1e9) - C_eq
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_sat_T_c-1.png" style="display: block; margin: auto;" />

### 3.2.3 Barometric pressure

``` r
C_b_eq_me <- rep(NA, nsim)

for(i in 1:nsim){
    C_b_eq_me[i] <- M_a_n2o * B_me[i] * 1e-6 * (H_n2o  * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15)))
}

# combine into dataframe for ggplot. qplot, which accepts vectors,
# was deprecated in ggplot 3.4.0
C_b_eq_df <- tibble(C_b_eq_me = C_b_eq_me, B_me = B_me)
# save absolute error for plot
C_b_eq_me_abs <- (C_b_eq_me * 1e9) - C_eq
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_sat_B-1.png" style="display: block; margin: auto;" />

### 3.2.4 All observables

Simulate measurement error for all observables (*i.e.*, air, water
temperature, and barometric pressure) and the resulting effects on
equilibrium N2O observations.

``` r
C_all_eq_me <- rep( NA, nsim )

for(i in 1:nsim){
    C_all_eq_me[i] <- M_a_n2o_me[i] * B_me[i] * 1e-6 * (H_n2o  * exp(2700 * (1 / (T_c_me[i] + 273.15) - 1 / 298.15)))
}

# combine into dataframe for ggplot. qplot, which accepts vectors,
# was deprecated in ggplot 3.4.0
C_all_eq_df <- tibble(C_all_eq_me = C_all_eq_me)
# save absolute error for plot
C_all_eq_me_abs <- (C_all_eq_me * 1e9) - C_eq
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_sat_all-1.png" style="display: block; margin: auto;" />

``` r
# standard deviation of a measurement is:
sd(C_all_eq_me* 1e9) # 0.20 nmol L-1
```

    ## [1] 0.1986766

``` r
# 2 standard deviations
sd(C_all_eq_me * 1e9) * 1.96 # +/- 0.395 nmol N2O
```

    ## [1] 0.3894061

``` r
# standard deviation of a measurement is:
round(sd(C_all_eq_me_abs), 3) # 0.20 nmol L-1
```

    ## [1] 0.199

``` r
# the 95% CI for an observation with measurement error can be computed 
# approximately as the value +/- 2 standard deviations; or the central 95th 
# percentile of the absolute error distribution.
round(quantile((C_eq + C_all_eq_me_abs), probs = c(0.025, 0.975)), 3) # +/- nmol N2O
```

    ##  2.5% 97.5% 
    ## 7.443 8.220

## 3.3 Summary of contributions to error

``` r
results_equilibrium_N2O <- tibble(Air_me = C_a_eq_me_abs,
                                Tc_me = C_tc_eq_me_abs,
                                B_me = C_b_eq_me_abs,
                                All_me = C_all_eq_me_abs) %>%
  pivot_longer(cols = ends_with("me"), names_to = "Source", values_to = "Error")


results_equilibrium_N2O %>%
  mutate(Source = factor(Source)) %>%
  mutate(Source = fct_relevel(Source, c("B_me",
                                        "Tc_me",
                                        "Air_me",
                                        "All_me"
                                        ))) %>%
  ggplot(aes(x = Error, y = Source)) +
  stat_pointinterval() +
  scale_y_discrete(labels=c("Air_me" = "Ambient air",
                            "Tc_me" = "Water temperature",
                            "B_me" = "Barometric pressure",
                            "All_me" = "All sources")) +
  xlab("Absolute error in equilibrium \nN2O estimate (measured - true) \n") +
  ylab("") 
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/final_figure_N2O_eq-1.png" style="display: block; margin: auto;" />

# 4 Saturation ratio

The equation for estimating the saturation ratio for ($N_20$) gas in
water is:

$$S = \left(\frac{(1^{-6} \cdot B ) \biggl( \frac{V_a (Mr_h - Mr_r)}{(8.3144598 \cdot K \cdot V_w) + H^{\theta} \cdot e^{2700 \cdot ( \frac{1}{K} - \frac{1}{298.15})} \cdot Mr_h} \biggr)}{1^{-6} \cdot B \cdot C_a \cdot e^{2700 \cdot \frac{1}{Tc} -  \frac{1}{298.15}}} \right)$$

where:

- $S$ = Saturatio ratio
- $B$ = barometric pressure (kPa)
- $C_a$ = the air concentration in nM
- $V_a$ = volume of reference gas in air
- $M_h$ = gas measured in the headspace
- $M_r$ = gas measured in the reference (air)
- $V_w$ = volume of water below the headspace
- $T_c$ = temperature celsius
- $H^{\theta}$ = Henry’s law constant ($H_{N_2O}$ below)

Simulate a “true” saturation ratio.

``` r
S <- (1e-6 * B * (V_g * (M_h - M_a_n2o) / (8.3144598 * (T_c + 273.15) * V_w) + H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15)) * M_h)) /
  (M_a_n2o * B * 1e-6 * (H_n2o  * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15)))) 
S
```

    ## [1] 0.9851878

Uncertainty in the saturation ratio, $S$, is simulated below for both a
standard GC and a high precision GC. The GC used for the current study
is optimized to quantify several analytes across a broad range of
concentrations but greater precision can be achieved by optimizing the
instrument for a single analyte across a narrower range of
concentrations. For example, laboratories participating in an
international analysis of N2O standards used for atmsopheric monitoring
reported a mean standard deviation of 7.3^{-4}, which is 10-fold lower
than the standard deviation of N2O measurements in this study
(0.0078725). Utilizing more precise N2O measurements for the
equilibrated headspace and/or air will reduce uncertainty in the
estimated dissolved N2O concentration, dissolved N2O equilibrium value,
and N2O saturation ratio.

In addition to the two different GC setups, uncertainty in $S$ is also
simulated for under additional scenarios using combinations of ambient
air vs a pure gas as the headspace gas along with a standard *vs.* high
precision thermometer for field temperature (C) measurements.

All of these simulations use the mixing ratio used for the empirical
field measurements, where $R_M =$ 0.25.

## 4.1 Standard GC

### 4.1.1 Ambient air, standard thermometer

Simulate uncertainty in the saturation ratio due to measurement error
associated with the standard GC, using ambient air as the headspace gas,
and a standard field thermometer for temperature. This scenario
represents the approach used for the empirical observations used in this
study.

``` r
S_a_me <- rep( NA, nsim )

for(i in 1:nsim){
S_a_me[i] <- (1e-6 * B_me[i] * (V_g_me[i] * (M_h_me[i] - M_a_n2o_me[i]) / (8.3144598 * (T_c_me[i] + 273.15) * V_w_me[i]) + H_n2o * exp(2700 * (1 / (T_c_me[i] + 273.15) - 1 / 298.15)) * M_h_me[i])) /
  (M_a_n2o_me[i] * B_me[i] * 1e-6 * (H_n2o  * exp(2700*(1/(T_c_me[i] + 273.15) - 1/298.15)))) 
}

# combine into dataframe for ggplot. qplot, which accepts vectors,
# was deprecated in ggplot 3.4.0
S_a_df <- tibble(S_a_me = S_a_me)
# save absolute error for plot
S_a_me_abs <- S_a_me - S
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_ratio_all_air-1.png" style="display: block; margin: auto;" />

``` r
# standard deviation of a saturation ratio value s:
sd(S_a_me) # 0.026
```

    ## [1] 0.05481428

``` r
# the 95% CI is therefore the computed value +/- 2 standard deviations
sd(S_a_me) * 1.96 # +/- 0.05258214
```

    ## [1] 0.107436

``` r
# so any ratio within 0.05 of 1.0 cannot be discerned as a source or sink?
```

``` r
# standard deviation of a measurement is:
round(sd(S_a_me_abs), 3) # 0.20 nmol L-1
```

    ## [1] 0.055

``` r
# central 95th percentile of absolute error
round(quantile(S_a_me_abs, probs = c(0.025, 0.975)), 3) # approx +/- 0.11
```

    ##   2.5%  97.5% 
    ## -0.103  0.111

``` r
# the 95% CI for an observation with measurement error can be computed 
# approximately as the value +/- 2 standard deviations; or the central 95th 
# percentile of the absolute error distribution.
round(quantile((S + S_a_me_abs), probs = c(0.025, 0.975)), 3) # +/- nmol N2O
```

    ##  2.5% 97.5% 
    ## 0.882 1.096

#### 4.1.1.1 Map source-sink status

Below, the source-sink status of the empirical measurments used for this
study are estimated, assuming the measurement errors estimated above.
The proportion of observations assumed to be sinks or sources of $N_2O$
is also estimated along with the estimate of the proportion of the lakes
sampled for which the status was assumed to be “undetermined” due to
uncertainty arising from the estimated measurement error. A map of each
of the sampled lakes’ estimated status is also shown.

``` r
# two standard deviations = 0.3019449
# 95% CI is therefore x+/-0.3019449

#dg <- dg %>%
#  mutate(n2o.src.snk.with.all.error = case_when(
#    abs(n2o.sat.ratio - 1) < sd(C_me_ratio) ~ "undetermined",
#    TRUE ~ n2o.src.snk)) 
dg <- dg %>%
  mutate(n2o.src.snk.with.all.error = case_when(
    abs(n2o.sat.ratio - 1) < quantile(S_a_me_abs, probs = c(0.975)) ~ "undetermined",
    TRUE ~ n2o.src.snk)) 

dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {table(.$n2o.src.snk.with.all.error)} %>%
  kable(digits = 2, caption = "Number of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         |  378 |
| source       |  184 |
| undetermined |  422 |

Number of lakes classified as source/sink

``` r
dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {prop.table(table(.$n2o.src.snk.with.all.error))} %>%
  kable(digits = 2, caption = "Proportion of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         | 0.38 |
| source       | 0.19 |
| undetermined | 0.43 |

Proportion of lakes classified as source/sink

``` r
# sf for plotting
dg.sf <- st_as_sf(dg, coords = c("map.lon.dd", "map.lat.dd"), 
                  crs = 4269) %>% # standard for lat/lon
  st_transform(5070) # project to CONUS ALBERS for plotting
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_map_ratio_all_air-1.png" style="display: block; margin: auto;" />

### 4.1.2 Pure gas, standard thermometer

Simulate uncertainty in the saturation ratio due to measurement error
associated with the standard GC, using a pure gas (helium) as the
headspace gas, and the standard field thermometer.

``` r
# set M_h to a lower value to generate a saturation ratio close to that used above
# M_h will be lower due to dilution with pure headspace gas (e.g. helium, nitrogen)
M_h_pg <- 0.2

# this simulates measurement error on GC.
# assume TRUE value is M_h and error is simulated with with sd of replicate GC measurements
M_h_pg_me <- rnorm(nsim, M_h_pg, n2o_sd_epa) # me in measured gas in headspace.
                                   # add random error to measured concentration

# under this scenario we don't need to subtract M_a_n2o (N2O concentration in air) from M_h (N2O concentration in equilibrated headspace)
S_pg <- (1e-6 * B * (V_g * (M_h_pg - 0) / (8.3144598 * (T_c + 273.15) * V_w) + H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15) ) * M_h_pg)) /
  (M_a_n2o * B * 1e-6 * (H_n2o  * exp(2700*(1/(T_c + 273.15) - 1/298.15))))

S_pg_me <- rep(NA, nsim)

for(i in 1:nsim){
S_pg_me[i] <- (1e-6 * B_me[i] * (V_g_me[i] * (M_h_pg_me[i] - 0) / (8.3144598 * (T_c_me[i] + 273.15) * V_w_me[i]) + H_n2o * exp(2700 * (1 / (T_c_me[i] + 273.15) - 1 / 298.15)) * M_h_pg_me[i])) /
  (M_a_n2o_me[i] * B_me[i] * 1e-6 * (H_n2o  * exp(2700*(1/(T_c_me[i] + 273.15) - 1/298.15)))) 
}

# combine into dataframe for ggplot. qplot, which accepts vectors,
# was deprecated in ggplot 3.4.0
S_pg_df <- tibble(S_pg_me = S_pg_me)
# save absolute error for plot
S_pg_me_abs <- S_pg_me - S_pg
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_ratio_pg_all-1.png" style="display: block; margin: auto;" />

``` r
# standard deviation of a saturation ratio value s:
sd(S_pg_me) # 0.048
```

    ## [1] 0.04814038

``` r
# the 95% CI is therefore the computed value +/- 2 standard deviations
sd(S_pg_me) * 1.96 # +/- 0.094
```

    ## [1] 0.09435515

``` r
# so any ratio within 0.094 of 1.0 cannot be discerned as a source or sink
```

``` r
# standard deviation of a measurement is:
round(sd(S_pg_me_abs), 3) # 0.20 nmol L-1
```

    ## [1] 0.048

``` r
# central 95th percentile of absolute error
round(quantile(S_pg_me_abs, probs = c(0.025, 0.975)), 3) # approx +/- 0.11
```

    ##   2.5%  97.5% 
    ## -0.091  0.097

``` r
# the 95% CI for an observation with measurement error can be computed 
# approximately as the value +/- 2 standard deviations; or the central 95th 
# percentile of the absolute error distribution.
round(quantile((S_pg + S_pg_me_abs), probs = c(0.025, 0.975)), 3) # +/- nmol N2O
```

    ##  2.5% 97.5% 
    ## 0.896 1.084

#### 4.1.2.1 Source-sink status

``` r
# two standard deviations = 0.3019449
# 95% CI is therefore x+/-0.3019449

#dg <- dg %>%
#  mutate(n2o.src.snk.with.all.error = case_when(
#    abs(n2o.sat.ratio - 1) < sd(C_me_ratio) ~ "undetermined",
#    TRUE ~ n2o.src.snk)) 
dg <- dg %>%
  mutate(n2o.src.snk.with.all.error = case_when(
    abs(n2o.sat.ratio - 1) < quantile(S_pg_me_abs, probs = c(0.975)) ~ "undetermined",
    TRUE ~ n2o.src.snk)) 

dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {table(.$n2o.src.snk.with.all.error)} %>%
  kable(digits = 2, caption = "Number of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         |  409 |
| source       |  194 |
| undetermined |  381 |

Number of lakes classified as source/sink

``` r
dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {prop.table(table(.$n2o.src.snk.with.all.error))} %>%
  kable(digits = 2, caption = "Proportion of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         | 0.42 |
| source       | 0.20 |
| undetermined | 0.39 |

Proportion of lakes classified as source/sink

### 4.1.3 Ambient air, high precision thermometer

Simulate uncertainty in the saturation ratio due to measurement error
associated with the standard GC and ambient air used as the headspace
gas, and a high precision thermometer (e.g., MIMS) used for temperature
measurements.

``` r
# High precision water batch used to control temperature of standards
T_c_mims_me <- rnorm(nsim, T_c, 0.005) # thermo stated bath precision is +/-0.01 Celsius
# hist(T_c_mims_me) # sd of 0.005 gives approx distribution for the specified precision

T_c_df <- tibble(T_c_me = T_c_me, 
                 T_c_me_abs = T_c_me - T_c,  
                 T_c_mims_me = T_c_mims_me,
                 T_c_mims_me_abs = T_c_mims_me - T_c)
```

The precision gain from the MIMS thermometer relative to the standard
field thermometer is roughly one order of magnitude.
<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_T_c_vs_T_c_MIMS-1.png" style="display: block; margin: auto;" />

Simulate uncertainty in the saturation ratio when using ambient air in
the headspace and high precision temperature measurements

``` r
S_a_hpt_me <- rep(NA, nsim)

for(i in 1:nsim){
S_a_hpt_me[i] <- (1e-6 * B_me[i] * (V_g_me[i] * (M_h_me[i] - M_a_n2o_me[i]) / (8.3144598 * (T_c_mims_me[i] + 273.15) * V_w_me[i]) + H_n2o * exp(2700 * (1 / (T_c_mims_me[i] + 273.15) - 1 / 298.15)) * M_h_me[i])) /
  (M_a_n2o_me[i] * B_me[i] * 1e-6 * (H_n2o  * exp(2700*(1/(T_c_mims_me[i] + 273.15) - 1/298.15)))) 
}

# combine into dataframe for ggplot. qplot, which accepts vectors,
# was deprecated in ggplot 3.4.0
S_a_hpt_df <- tibble(S_a_hpt_me = S_a_hpt_me)
# save absolute error for plot
S_a_hpt_me_abs <- S_a_hpt_me - S
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_ratio_all_air_hpt-1.png" style="display: block; margin: auto;" />

``` r
# standard deviation of a saturation ratio value s:
sd(S_a_hpt_me) # 0.026
```

    ## [1] 0.05481451

``` r
# the 95% CI is therefore the computed value +/- 2 standard deviations
sd(S_a_hpt_me) * 1.96 # +/- 0.05258214
```

    ## [1] 0.1074364

``` r
# so any ratio within 0.05 of 1.0 cannot be discerned as a source or sink?
```

``` r
# standard deviation of a measurement is:
round(sd(S_a_hpt_me_abs), 3) # 0.20 nmol L-1
```

    ## [1] 0.055

``` r
# central 95th percentile of absolute error
round(quantile(S_a_hpt_me_abs, probs = c(0.025, 0.975)), 3) # approx +/- 0.11
```

    ##   2.5%  97.5% 
    ## -0.103  0.111

``` r
# the 95% CI for an observation with measurement error can be computed 
# approximately as the value +/- 2 standard deviations; or the central 95th 
# percentile of the absolute error distribution.
round(quantile((S + S_a_hpt_me_abs), probs = c(0.025, 0.975)), 3) # +/- nmol N2O
```

    ##  2.5% 97.5% 
    ## 0.882 1.096

#### 4.1.3.1 Source-sink status

``` r
# two standard deviations = 0.3019449
# 95% CI is therefore x+/-0.3019449

#dg <- dg %>%
#  mutate(n2o.src.snk.with.all.error = case_when(
#    abs(n2o.sat.ratio - 1) < sd(C_me_ratio) ~ "undetermined",
#    TRUE ~ n2o.src.snk)) 
dg <- dg %>%
  mutate(n2o.src.snk.with.all.error = case_when(
    abs(n2o.sat.ratio - 1) < quantile(S_a_hpt_me_abs, probs = c(0.975)) ~ "undetermined",
    TRUE ~ n2o.src.snk)) 

dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {table(.$n2o.src.snk.with.all.error)} %>%
  kable(digits = 2, caption = "Number of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         |  378 |
| source       |  184 |
| undetermined |  422 |

Number of lakes classified as source/sink

``` r
dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {prop.table(table(.$n2o.src.snk.with.all.error))} %>%
  kable(digits = 2, caption = "Proportion of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         | 0.38 |
| source       | 0.19 |
| undetermined | 0.43 |

Proportion of lakes classified as source/sink

### 4.1.4 Pure gas, high precision thermometer

Simulate uncertainty in the saturation ratio due to measurement error
associated with the standard GC, a pure gas (helium) as the headspace
gas, and using a high precision thermometer (e.g., MIMS).

``` r
S_pg_hpt_me <- rep(NA, nsim)

for(i in 1:nsim){
S_pg_hpt_me[i] <- (1e-6 * B_me[i] * (V_g_me[i] * (M_h_pg_me[i] - 0) / (8.3144598 * (T_c_mims_me[i] + 273.15) * V_w_me[i]) + H_n2o * exp(2700 * (1 / (T_c_mims_me[i] + 273.15) - 1 / 298.15)) * M_h_pg_me[i])) /
  (M_a_n2o_me[i] * B_me[i] * 1e-6 * (H_n2o  * exp(2700*(1/(T_c_mims_me[i] + 273.15) - 1/298.15)))) 
}

# combine into dataframe for ggplot. qplot, which accepts vectors,
# was deprecated in ggplot 3.4.0
S_pg_hpt_df <- tibble(S_pg_hpt_me = S_pg_hpt_me)
# save absolute error for plot
S_pg_hpt_me_abs <- S_pg_hpt_me - S_pg
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_ratio_all_pg_hpt-1.png" style="display: block; margin: auto;" />

``` r
# standard deviation of a saturation ratio value s:
sd(S_pg_hpt_me) # 0.026
```

    ## [1] 0.04814114

``` r
# the 95% CI is therefore the computed value +/- 2 standard deviations
sd(S_pg_hpt_me) * 1.96 # +/- 0.05258214
```

    ## [1] 0.09435663

``` r
# so any ratio within 0.05 of 1.0 cannot be discerned as a source or sink?
```

``` r
# standard deviation of a measurement is:
round(sd(S_pg_hpt_me_abs), 3) # 0.20 nmol L-1
```

    ## [1] 0.048

``` r
# central 95th percentile of absolute error
round(quantile(S_pg_hpt_me_abs, probs = c(0.025, 0.975)), 3) # approx +/- 0.11
```

    ##   2.5%  97.5% 
    ## -0.091  0.097

``` r
# the 95% CI for an observation with measurement error can be computed 
# approximately as the value +/- 2 standard deviations; or the central 95th 
# percentile of the absolute error distribution.
round(quantile((S_pg + S_pg_hpt_me_abs), probs = c(0.025, 0.975)), 3) # +/- nmol N2O
```

    ##  2.5% 97.5% 
    ## 0.896 1.084

#### 4.1.4.1 Source-sink status

``` r
# two standard deviations = 0.3019449
# 95% CI is therefore x+/-0.3019449

#dg <- dg %>%
#  mutate(n2o.src.snk.with.all.error = case_when(
#    abs(n2o.sat.ratio - 1) < sd(C_me_ratio) ~ "undetermined",
#    TRUE ~ n2o.src.snk)) 
dg <- dg %>%
  mutate(n2o.src.snk.with.all.error = case_when(
    abs(n2o.sat.ratio - 1) < quantile(S_pg_hpt_me_abs, probs = c(0.975)) ~ "undetermined",
    TRUE ~ n2o.src.snk)) 

dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {table(.$n2o.src.snk.with.all.error)} %>%
  kable(digits = 2, caption = "Number of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         |  408 |
| source       |  194 |
| undetermined |  382 |

Number of lakes classified as source/sink

``` r
dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {prop.table(table(.$n2o.src.snk.with.all.error))} %>%
  kable(digits = 2, caption = "Proportion of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         | 0.41 |
| source       | 0.20 |
| undetermined | 0.39 |

Proportion of lakes classified as source/sink

## 4.2 High precision GC

The GC used in the current study is optimized to quantify several
analytes across a broad range of concentrations but greater precision
can be achieved by optimizing the instrument for a single analyte across
a narrower range of concentrations. For example, laboratories
participating in an international analysis of N2O standards used for
atmsopheric monitoring reported a mean standard deviation of 7.3^{-4},
which is 10-fold lower than the standard deviation of N2O measurements
in this study (0.0078725). Utilizing more precise N2O measurements for
the equilibrated headspace and/or air will reduce uncertainty in the
estimated dissolved N2O concentration, dissolved N2O equilibrium value,
and N2O saturation ratio. Again, the simulations below assume the mixing
ratio used in the empirical field observations, where $R_M =$ 0.25.

### 4.2.1 Pure gas, standard thermometer

``` r
# set M_h to a lower value to generate a saturation ratio close to that used above
# M_h will be lower due to dilution with pure headspace gas (e.g. helium, nitrogen)
M_h_pg_hpgc <- 0.2

# two standard deviations reported from each of three labs participating in
# a laboratory intercomparison study
n2o_sd_lit <- mean(c(.00033, .00034, .00031, 0.00030, 0.0016, 0.0015)) # from literature

# simulate improved measurement of N2O in equilibrated headspace gas 
# assume TRUE value is M_h_pg_hpgc and error is simulated with with sd from literature
M_h_pg_hpgc_me <- rnorm(nsim, M_h_pg_hpgc, n2o_sd_lit) 

# Set true value
# under this scenario we don't need to subtract M_a_n2o (N2O concentration in air) from M)h (N2O concentration in equilibrated headspace)
S_pg_hpgc <- (1e-6 * B * (V_g * (M_h_pg_hpgc - 0) / (8.3144598 * (T_c + 273.15) * V_w) + H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15) ) * M_h_pg_hpgc)) /
  (M_a_n2o * B * 1e-6 * (H_n2o  * exp(2700*(1/(T_c + 273.15) - 1/298.15))))

# Container for simulations
S_pg_hpgc_me <- rep(NA, 1e4)

for(i in 1:nsim){
S_pg_hpgc_me[i] <- (1e-6 * B_me[i] * (V_g_me[i] * (M_h_pg_hpgc_me[i] - 0) / (8.3144598 * (T_c_me[i] + 273.15) * V_w_me[i]) + H_n2o * exp(2700 * (1 / (T_c_me[i] + 273.15) - 1 / 298.15) ) * M_h_pg_hpgc_me[i])) /
  (M_a_n2o_me[i] * B_me[i] * 1e-6 * (H_n2o  * exp(2700*(1 / (T_c_me[i] + 273.15) - 1/298.15)))) 
}

# combine into dataframe for ggplot. qplot, which accepts vectors,
# was deprecated in ggplot 3.4.0
S_pg_hpgc_df <- tibble(S_pg_hpgc_me = S_pg_hpgc_me)
# save absolute error for plot
S_pg_hpgc_me_abs <- S_pg_hpgc_me - S_pg_hpgc
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_ratio_gc_pure_gas_all-1.png" style="display: block; margin: auto;" />

``` r
# standard deviation of a saturation ratio value s:
sd(S_pg_hpgc_me) # 0.029
```

    ## [1] 0.02857339

``` r
# the 95% CI is therefore the computed value +/- 2 standard deviations
sd(S_pg_hpgc_me) * 1.96 # +/- 0.0569
```

    ## [1] 0.05600384

``` r
# so any ratio within 0.0569 of 1.0 cannot be discerned as a source or sink
```

``` r
# standard deviation of a measurement is:
round(sd(S_pg_hpgc_me_abs), 3) # 0.20 nmol L-1
```

    ## [1] 0.029

``` r
# central 95th percentile of absolute error
round(quantile(S_pg_hpgc_me_abs, probs = c(0.025, 0.975)), 3) # approx +/- 0.11
```

    ##   2.5%  97.5% 
    ## -0.054  0.058

``` r
# the 95% CI for an observation with measurement error can be computed 
# approximately as the value +/- 2 standard deviations; or the central 95th 
# percentile of the absolute error distribution.
round(quantile((S_pg_hpgc + S_pg_hpgc_me_abs), probs = c(0.025, 0.975)), 3) # +/- nmol N2O
```

    ##  2.5% 97.5% 
    ## 0.934 1.046

#### 4.2.1.1 Source-sink status

``` r
# two standard deviations = 0.3019449
# 95% CI is therefore x+/-0.3019449

#dg <- dg %>%
#  mutate(n2o.src.snk.with.all.error = case_when(
#    abs(n2o.sat.ratio - 1) < sd(C_me_ratio) ~ "undetermined",
#    TRUE ~ n2o.src.snk)) 
dg <- dg %>%
  mutate(n2o.src.snk.with.all.error = case_when(
    abs(n2o.sat.ratio - 1) < quantile(S_pg_hpgc_me_abs, probs = c(0.975)) ~ "undetermined",
    TRUE ~ n2o.src.snk)) 

dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {table(.$n2o.src.snk.with.all.error)} %>%
  kable(digits = 2, caption = "Number of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         |  521 |
| source       |  248 |
| undetermined |  215 |

Number of lakes classified as source/sink

``` r
dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {prop.table(table(.$n2o.src.snk.with.all.error))} %>%
  kable(digits = 2, caption = "Proportion of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         | 0.53 |
| source       | 0.25 |
| undetermined | 0.22 |

Proportion of lakes classified as source/sink

### 4.2.2 Pure gas, high precision thermometer

``` r
# set M_h to a lower value to generate a saturation ratio close to that used above
# M_h will be lower due to dilution with pure headspace gas (e.g. helium, nitrogen)
M_h_pg_hpgc_hpt <- 0.2

# two standard deviations reported from each of three labs participating in
# a laboratory intercomparison study
n2o_sd_lit <- mean(c(.00033, .00034, .00031, 0.00030, 0.0016, 0.0015)) # from literature

# simulate improved measurement of N2O in equilibrated headspace gas 
# assume TRUE value is C_e and error is simulated with with sd from literature
M_h_pg_hpgc_hpt_me <- rnorm(nsim, M_h_pg_hpgc, n2o_sd_lit) 

S_pg_hpgc_hpt_me <- rep(NA, 1e4)

for(i in 1:nsim){
S_pg_hpgc_hpt_me[i] <- (1e-6 * B_me[i] * (V_g_me[i] * (M_h_pg_hpgc_hpt_me[i] - 0) / (8.3144598 * (T_c_me[i] + 273.15) * V_w_me[i]) + H_n2o * exp(2700 * (1 / (T_c_me[i] + 273.15) - 1 / 298.15) ) * M_h_pg_hpgc_hpt_me[i])) /
  (M_a_n2o_me[i] * B_me[i] * 1e-6 * (H_n2o  * exp(2700*(1 / (T_c_me[i] + 273.15) - 1/298.15)))) 
}


# combine into dataframe for ggplot. qplot, which accepts vectors,
# was deprecated in ggplot 3.4.0
S_pg_hpgc_hpt_df <- tibble(S_pg_hpgc_hpt_me = S_pg_hpgc_hpt_me)
# save absolute error for plot
S_pg_hpgc_hpt_me_abs <- S_pg_hpgc_hpt_me - S_pg_hpgc
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_ratio_hpgc_pg_hpt_all-1.png" style="display: block; margin: auto;" />

``` r
# standard deviation of a saturation ratio value s:
sd(S_pg_hpgc_hpt_me) # 0.029
```

    ## [1] 0.02857729

``` r
# the 95% CI is therefore the computed value +/- 2 standard deviations
sd(S_pg_hpgc_hpt_me) * 1.96 # +/- 0.0569
```

    ## [1] 0.05601149

``` r
# so any ratio within 0.0569 of 1.0 cannot be discerned as a source or sink
```

``` r
# standard deviation of a measurement is:
round(sd(S_pg_hpgc_hpt_me_abs), 3) # 0.20 nmol L-1
```

    ## [1] 0.029

``` r
# central 95th percentile of absolute error
round(quantile(S_pg_hpgc_hpt_me_abs, probs = c(0.025, 0.975)), 3) # approx +/- 0.11
```

    ##   2.5%  97.5% 
    ## -0.054  0.058

``` r
# the 95% CI for an observation with measurement error can be computed 
# approximately as the value +/- 2 standard deviations; or the central 95th 
# percentile of the absolute error distribution.
round(quantile((S_pg_hpgc + S_pg_hpgc_hpt_me_abs), probs = c(0.025, 0.975)), 3) # +/- nmol N2O
```

    ##  2.5% 97.5% 
    ## 0.934 1.046

#### 4.2.2.1 Source-sink status

``` r
# two standard deviations = 0.3019449
# 95% CI is therefore x+/-0.3019449

#dg <- dg %>%
#  mutate(n2o.src.snk.with.all.error = case_when(
#    abs(n2o.sat.ratio - 1) < sd(C_me_ratio) ~ "undetermined",
#    TRUE ~ n2o.src.snk)) 
dg <- dg %>%
  mutate(n2o.src.snk.with.all.error = case_when(
    abs(n2o.sat.ratio - 1) < quantile(S_pg_hpgc_hpt_me_abs, probs = c(0.975)) ~ "undetermined",
    TRUE ~ n2o.src.snk)) 

dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {table(.$n2o.src.snk.with.all.error)} %>%
  kable(digits = 2, caption = "Number of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         |  521 |
| source       |  248 |
| undetermined |  215 |

Number of lakes classified as source/sink

``` r
dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {prop.table(table(.$n2o.src.snk.with.all.error))} %>%
  kable(digits = 2, caption = "Proportion of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         | 0.53 |
| source       | 0.25 |
| undetermined | 0.22 |

Proportion of lakes classified as source/sink

# 5 MIMS

In Membrane Inlet Mass Spectrometry (MIMS), dissolved gases (typically
N2, Ar, O2, N2O) are extracted from a water sample as it passes through
a semi-permeable membrane. The abundance of the extracted gases are then
measured using a quadrapole mass spectrometer. One advantage of this
approach relative to the traditional headspace equilibrium method is
that dissolved gases are extracted without contamination and/or dilution
from a headspace gas (e.g. air, nitrogen, helium). Dilution by a pure
gas isn’t much of a concern because N2O detection limits are very low
via GC, but correction for the presence of N2O in the headspace gas
reduces measurement precision.

Dissolved N2O concentration in a MIMS sample can be calculated from the
N2O signal strength and a calibration factor. The calibration factor is
determined by analyzing a water sample with known N2O concentration.
This is most frequently accomplished by equilibrating laboratory water
with air at constant temperature, barometric pressure and N2O partial
pressure. The dissolved N2O concentration of the equilibrated water is
calculated from solubility laws (see [Equilibrium
concentration](#3-equilibrium-concentration)) and the calibration
factor, $f$, is:

$$f_{N_2O} = \frac{C^{N_2O}_s}{X^{N_2O}_s}$$ where:

- $f_{N_2O} = \text{N}_2\text{O} \text{ calibration factor}$
- $C^{N_2O}_s = \text{Concentration of } \text{N}_2\text{O} \text{ in calibration standard}$
- $X^{N_2O}_s = \text{Signal strength of } \text{N}_2\text{O} \text{ in calibration standard}$

The N2O concentration of an unknown sample is then calculated as:

$$C^{N_2O}_{\theta} = \frac{X^{N_2O}}{f_{N_2O}}$$

where:

- $C^{N_2O}_{\theta} = \text{N}_2\text{O} \text{ concentration in sample}$
- $X^{N_2O}_{\theta} = \text{Signal strength of } \text{N}_2\text{O} \text{ in sample}$
- $f_{N_2O} = \text{N}_2\text{O} \text{ calibration factor}$

The equation simply scales the concentration of the N2O standard to the
ratio of the MIMS signal strength of the unknown and standard. Via
substitution and rearrangement, the equation can also be written as:

$$C^{N_2O}_{\theta} =\frac{X^{N_2O}_{\theta}}{X^{N_2O}_s} \cdot C^{N_2O}_s$$

MIMS provides greater precision when measuring gas ratios as compared to
measuring a single gas. High precision gas ratio data can be converted
to concentration data with the independent measurement of concentration
of one of the component gases. N2O:Ar ratios are particularly useful for
environmental samples because Ar is biologically inert, therefore its
concentration can be calculated from solubility, water temperature,
barometric pressure, and atmospheric partial pressure (essentially
constant across the globe). As with single gas measurements, N2O:Ar
ratios are converted to N2O concentration using a calibration factor
derived from measurement of laboratory water equilibrated with air:

$$\begin{aligned}
f^\prime_{N_2O} =\frac{R^{C}_s}{R^{X}_s} \\
R^{C}_s = \frac{C^{N_2O}_s}{C^{Ar}_s} \\
R^{X}_s = \frac{X^{N_2O}_s}{X^{Ar}_s}
\end{aligned}$$

where:

- $f^\prime_{N_2O} = \text{N}_2\text{O} \text{ (ratio) calibration factor}$
- $R^{C}_s = \text{Concetration ratio of } \text{N}_2\text{O} \text{ to Ar in calibration standard}$
- $R^{X}_s = \text{Signal ratio of } \text{N}_2\text{O} \text{ to Ar in calibration standard}$

The N2O concentration of an unknown sample is then calculated as:

$$C^{N_2O}_{\theta} = R^{X}_s \cdot f^\prime_{N_2O} \cdot C^{Ar}_s$$
where:

- $C^{N_2O}_{\theta} = \text{Concentration of } \text{N}_2\text{O} \text{ in the sample}$
- $C^{Ar}_s = \text{Concentration of } Ar \text{ in the calibration standard}$

As before, the equation above scales the N2O:Ar ratio measured in the
sample to that of the standard. Via substitution and rearrangement, the
equation can be written as:
$$C^{N_2O}_{\theta} = \frac{R^{X}_{\theta}}{R^{X}_s} \cdot R^{C}_s \cdot C^{Ar}_s$$
where:

- $R^{X}_\theta = \text{Signal ratio of } \text{N}_2\text{O} \text{ to Ar in the sample}$

Concentration of N2O and Ar in the standard, $C^{N_2O}_s$ is calculated
from temperature of the equilibrated water, $Tc$, laboratory barometric
pressure, $B$, and N2O and Ar partial pressure in the atmosphere,
$C^{N_2O}_a$, $C^{Ar}_a$, respectively (see [Equilibrium
concentration](#3-equilibrium-concentration)). By substitution, the
equation to estimate the concentration of N2O in the unknown sample
becomes:

$$\begin{align}
C^{N_2O}_{\theta} = \frac{R^{X}_{\theta}}{R^{X}_s} \times \\ \frac{ 1^{-6} \cdot B \cdot C^{N_2O}_a \cdot (H_{N_2O} \cdot e^{2700 \cdot \frac{1}{Tc} -  \frac{1}{298.15}})}{1^{-6} \cdot B \cdot C^{Ar}_a \cdot (H_{Ar} \cdot e^{1500 \cdot \frac{1}{Tc} -  \frac{1}{298.15}})} \times \\ 1^{-6}B \cdot C^{Ar}_a \cdot (H_{Ar} \cdot e^{1500 \cdot  \frac{1}{Tc} -  \frac{1}{298.15}})
\end{align}$$

where (repeating some variables from above for clarity):

- $C^{N_2O}_{\theta} = \text{concentration of } \text{N}_2\text{O} \text{ in the sample}$
- $R^{X}_\theta = \text{Signal ratio of } \text{N}_2\text{O} \text{ to Ar in the sample}$
- $R^{X}_s = \text{Signal ratio of } \text{N}_2\text{O} \text{ to Ar in calibration standard}$
- $B = \text{barometric pressure (kPa)}$
- $Tc = \text{temperature}$
- $C^{N_2O}_a = \text{concentration of } \text{N}_2\text{O} \text{ in the atmosphere}$
- $H_{N_2O} = \text{Henry's law constant for } \text{N}_2\text{O}$
- $C^{Ar}_a = \text{concentration of } Ar \text{ in the atmosphere}$
- $H_{Ar} = \text{Henry's law constant for } Ar$
- $Tc = \text{Temperature of the equilibrated water}$

N2O concentration in laboratory air, ($C^{N_2O}_a$), is derived from
direct measurement of N2O via gas chromatography and has a standard
deviation of 0.0078725 using the method employed in this study (see
“Empirical variation” under [Dissolved
concentration](#2-dissolved-concentration)) although more precise
methods are available (see
$$Dissolved $\text{N}_2\text{O}$: high precision
GC$$). Ar in the lab atmosphere ($C^{Ar}_a$) is 9340ppm and is assumed
to be constant and perfectly known (e.g. zero error). Genther et al
(2013) report a standard deviation of 0.022 for the N2O:Ar signal
strength ratio but Speir et al (2023) report a CV of 0.05% (sd ~
0.000096). Here we provisionally use the Speir value pending
confirmation. MIMS uses a very precise thermometer (+/-0.01 C) to
control the temperature of the water used as the analytical standard
(simulated below). Measurement error in $B$ was quantified above.

## 5.1 Dissolved concentration

### 5.1.1 Standard GC

``` r
#n2o_ar_standard_signal <- 0.27 # Speir et al. 0.27 pers comm. Genther et al. section 3.4 reports 0.192
#n2o_ar_sample_signal <- 0.23 # just a guess
#C_a_ar <- 9340 # ppm.  invariant
#T_c_mims <- T_c # set up for simulation of MIMS temperature bath (from GC simulations above)

R_X_s <- 0.27 # Speir et al. 0.27 pers comm. Genther et al. section 3.4 reports 0.192 (signal ratio of N2O:Ar in standard)
R_X_theta <- 0.23 # just a guess (signal ratio of N2O:Ar in sample)
C_Ar_a <- 9340 # ppm.  invariant (Concentration of Ar in atmospher)
# T_c_mims <- T_c # set up for simulation of MIMS temperature bath (from GC simulations above)


# this simulates MIMS measurement.
# assume TRUE value is n2o_ar_standard_signal and error is simulated with CV from Speier et al. 2023
# Speier et al reports "...our precision was good among all sample and standard replicated (average CV = 0.05%)."
# Using that CV to model measurement error in the N2O:Ar ratio yields a sd of dissolved N2O that exceeds
# the sd when measured via GC, which is weird.  Speier reported that the CV is for the N2O:Ar ratio.
# Genther et al reports a n2o:Ar sd of 0.022, but this also generated a sd for dissolved N2O which exceeds that 
# measured with GC!  Here we use the Speir data
R_X_s_sd <- 0.0005 * R_X_s # sd = CV * mean. 0.05% = 0.0005
R_X_theta_sd <- 0.0005 * R_X_theta # sd = CV * mean. 0.05% = 0.0005

R_X_s_me <- rnorm(nsim, R_X_s, R_X_s_sd) # me in N2O:Ar of standard  
R_X_theta_me <- rnorm(nsim, R_X_theta, R_X_theta_sd) # me in N2O:Ar of sample  


# High precision water batch used to control temperature of standards
T_c_mims_me <- rnorm(nsim, T_c, 0.005) # thermo stated bath precision is +/-0.01 Celsius
# hist(T_c_mims_me) # sd of 0.005 gives approx distribution for the specified precision

# True dissolved N2O concentration
C_mims <- ((R_X_theta / R_X_s) * # N2O:Ar ratios for sample and standard
  ((1e-6 * B * M_a_n2o * (H_n2o * exp( 2700 * (1 / (T_c + 273.15) - 1 / 298.15)))) / #N2O concentration in standard
  (1e-6 * B * C_Ar_a * (H_ar * exp( 1500 * (1 / (T_c + 273.15) - 1 / 298.15))))) * #Ar concentration in standard
  (1e-6 * B * C_Ar_a * (H_ar  * exp(1500 *(1/(T_c + 273.15) - 1/298.15))))) 

C_mims <- C_mims * 1e9 #convert mol to nmol

# variation in measured N2O concentration
C_mims_hpt_me <- rep(NA, nsim)

for(i in 1:nsim){
C_mims_hpt_me[i] <- (R_X_theta_me[i] / R_X_s_me[i]) * # N2O:Ar ratios for sample and standard
  ((1e-6 * B_me[i] * M_a_n2o_me[i] * (H_n2o * exp(2700 * (1 / (T_c_mims_me[i] + 273.15) - 1 / 298.15)))) / #N2O concentration in standard
  (1e-6 * B_me[i] * C_Ar_a * (H_ar * exp(1500 * (1 / (T_c_mims_me[i] + 273.15) - 1 / 298.15))))) * #Ar concentration in standard
  (1e-6 * B_me[i] * C_Ar_a * (H_ar * exp(1500 * (1 / (T_c_mims_me[i] + 273.15) - 1 / 298.15)))) 
}


# combine into dataframe for ggplot.
C_mims_hpt_df <- tibble(C_mims_hpt_me = C_mims_hpt_me * 1e9)
# save absolute error for plot
C_mims_hpt_me_abs <- (C_mims_hpt_me * 1e9) - C_mims
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_hpt_MIMS-1.png" style="display: block; margin: auto;" />

``` r
# standard deviation of dissolved N2O via MIMS:
sd(C_mims_hpt_me * 1e9) # 0.169 using Speir CV to calculate N2O:Ar sd.   0.15 with GC and headspace equilibrium!
```

    ## [1] 0.1691907

``` r
# 1.96 x sd
sd(C_mims_hpt_me * 1e9) * 1.96 
```

    ## [1] 0.3316138

``` r
# CV of dissolved N2O
(sd(C_mims_hpt_me * 1e9) / mean(C_mims_hpt_me * 1e9)) * 100 # 0.025 = 2.5% vs 12% using Genther sd.
```

    ## [1] 2.536562

``` r
# standard deviation of a measurement is:
round(sd(C_mims_hpt_me_abs), 3) # 0.20 nmol L-1
```

    ## [1] 0.169

``` r
# central 95th percentile of absolute error
round(quantile(C_mims_hpt_me_abs, probs = c(0.025, 0.975)), 3) # approx +/- 0.11
```

    ##   2.5%  97.5% 
    ## -0.330  0.332

``` r
# the 95% CI for an observation with measurement error can be computed 
# approximately as the value +/- 2 standard deviations; or the central 95th 
# percentile of the absolute error distribution.
round(quantile((C_mims + C_mims_hpt_me_abs), probs = c(0.025, 0.975)), 3) # +/- nmol N2O
```

    ##  2.5% 97.5% 
    ## 6.340 7.002

### 5.1.2 High precision GC

``` r
# simulate improved measurement precision of GC measurement of N2O in air
M_a_n2o_lit_me <- rnorm(nsim, M_a_n2o, n2o_sd_lit) # from intn'l labs = 0.00073 vs epa sd = 0.00787
```

``` r
# variation in measured N2O concentration
C_mims_hpgc_hpt_me <- rep(NA, nsim)

for(i in 1:nsim){
C_mims_hpgc_hpt_me[i] <- (R_X_theta_me[i] / R_X_s_me[i]) * # N2O:Ar ratios for sample and standard
  ((1e-6 * B_me[i] * M_a_n2o_lit_me[i] * (H_n2o * exp(2700 * (1 / (T_c_mims_me[i] + 273.15) - 1 / 298.15)))) / #N2O concentration in standard
  (1e-6 * B_me[i] * C_Ar_a * (H_ar * exp(1500 * (1 / (T_c_mims_me[i] + 273.15) - 1 / 298.15))))) * #Ar concentration in standard
  (1e-6 * B_me[i] * C_Ar_a * (H_ar  * exp(1500 * (1 / (T_c_mims_me[i] + 273.15) - 1 / 298.15)))) 
}


# combine into dataframe for ggplot.
C_mims_hpgc_hpt_df <- tibble(C_mims_hpgc_hpt_me = C_mims_hpgc_hpt_me * 1e9)
# save absolute error for plot
C_mims_hpgc_hpt_me_abs <- (C_mims_hpgc_hpt_me * 1e9) - C_mims
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_hpgc_hpt_MIMS-1.png" style="display: block; margin: auto;" />

``` r
# standard deviation of dissolved N2O via MIMS:
sd(C_mims_hpgc_hpt_me * 1e9) # 0.0166 using Speir CV to calculate N2O:Ar sd. 0.15 with GC and headspace equilibrium!
```

    ## [1] 0.01659803

``` r
sd(C_mims_hpgc_hpt_me * 1e9) * 1.96 # sd +/- 1.96 (for conf intervals on error)
```

    ## [1] 0.03253213

``` r
# CV of dissolved N2O
sd(C_mims_hpgc_hpt_me * 1e9) / mean(C_mims_hpgc_hpt_me * 1e9) * 100 # 0.0025 = 0.25% CV 
```

    ## [1] 0.2488408

``` r
# standard deviation of a measurement is:
round(sd(C_mims_hpgc_hpt_me_abs), 3)
```

    ## [1] 0.017

``` r
# central 95th percentile of absolute error
round(quantile(C_mims_hpgc_hpt_me_abs, probs = c(0.025, 0.975)), 3) 
```

    ##   2.5%  97.5% 
    ## -0.033  0.033

``` r
# the 95% CI for an observation with measurement error can be computed 
# approximately as the value +/- 2 standard deviations; or the central 95th 
# percentile of the absolute error distribution.
round(quantile((C_mims + C_mims_hpgc_hpt_me_abs), probs = c(0.025, 0.975)), 3) # +/- nmol N2O
```

    ##  2.5% 97.5% 
    ## 6.637 6.703

## 5.2 Saturation ratio

Calculate the “true” ratio via MIMS

``` r
# True ratio
S_mims <- ((R_X_theta / R_X_s) * # N2O:Ar ratios for sample and standard
  ( (1e-6 * B * M_a_n2o * (H_n2o * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15)))) / #N2O concentration in standard
  (1e-6 * B * C_Ar_a * (H_ar * exp(1500 * (1 / (T_c + 273.15) - 1 / 298.15)) ))) * #Ar concentration in standard
  (1e-6 * B * C_Ar_a *(H_ar  * exp(1500 * (1 / (T_c + 273.15) - 1 / 298.15))))) / # Ar concentration in standard
  (M_a_n2o * B * 1e-6 * (H_n2o  * exp(2700 * (1 / (T_c + 273.15) - 1 / 298.15)))) # N2o saturation value in waterbody 
```

### 5.2.1 Standard GC

#### 5.2.1.1 Standard field temperature

This simulation is for field measurment of temperature for the
equilibrium concentration with the standard glass thermometer.

``` r
# ratio simulated with measurement error in mims
S_mims_me <- rep(NA, nsim)

for( i in 1:nsim){
S_mims_me[i] <- (R_X_theta_me[i] / R_X_s_me[i]) * # N2O:Ar ratios for sample and standard
  ((1e-6 * B_me[i] * M_a_n2o_me[i] * (H_n2o * exp(2700 * (1 / (T_c_mims_me[i] + 273.15) - 1 / 298.15)))) / #N2O concentration in standard
  (1e-6 * B_me[i] * C_Ar_a * (H_ar * exp(1500 * (1 / (T_c_mims_me[i] + 273.15) - 1 / 298.15))))) * #Ar concentration in standard
  (1e-6 * B_me[i] * C_Ar_a * (H_ar  * exp(1500 *(1/(T_c_mims_me[i] + 273.15) - 1/298.15)))) / # Ar concentration in standard 
  (M_a_n2o_me[i] * B_me[i] * 1e-6 * (H_n2o  * exp(2700 * (1 / (T_c_me[i] + 273.15) - 1/298.15)))) # N2o saturation value in waterbody. Using regular field temp
}


# combine into dataframe for ggplot.
S_mims_df <- tibble(S_mims_me = S_mims_me)
# save absolute error for plot
S_mims_me_abs <- S_mims_me - S_mims
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_ratio_MIMS-1.png" style="display: block; margin: auto;" />

``` r
# standard deviation of a saturation ratio value s:
sd(S_mims_me) # 0.0011
```

    ## [1] 0.001071651

``` r
# the 95% CI is therefore the computed value +/- 2 standard deviations
sd(S_mims_me) * 1.96 # +/- 0.00216
```

    ## [1] 0.002100436

``` r
# so any ratio within 0.00216 of 1.0 cannot be discerned as a source or sink

# CV of sat
sd(S_mims_me) / mean(S_mims_me) * 100 # 0.13% CV 
```

    ## [1] 0.1258031

``` r
# standard deviation of a measurement is:
round(sd(S_mims_me_abs), 3) # 0.20 nmol L-1
```

    ## [1] 0.001

``` r
# central 95th percentile of absolute error
round(quantile(S_mims_me_abs, probs = c(0.025, 0.975)), 3) # approx +/- 0.11
```

    ##   2.5%  97.5% 
    ## -0.002  0.002

``` r
# the 95% CI for an observation with measurement error can be computed 
# approximately as the value +/- 2 standard deviations; or the central 95th 
# percentile of the absolute error distribution.
round(quantile((S_mims + S_mims_me_abs), probs = c(0.025, 0.975)), 3) # +/- nmol N2O 
```

    ##  2.5% 97.5% 
    ## 0.850 0.854

##### 5.2.1.1.1 Source-sink status

``` r
# two standard deviations = 0.3019449
# 95% CI is therefore x+/-0.3019449

#dg <- dg %>%
#  mutate(n2o.src.snk.with.all.error = case_when(
#    abs(n2o.sat.ratio - 1) < sd(C_me_ratio) ~ "undetermined",
#    TRUE ~ n2o.src.snk)) 
dg <- dg %>%
  mutate(n2o.src.snk.with.all.error = case_when(
    abs(n2o.sat.ratio - 1) < quantile(S_mims_me_abs, probs = c(0.975)) ~ "undetermined",
    TRUE ~ n2o.src.snk)) 

dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {table(.$n2o.src.snk.with.all.error)} %>%
  kable(digits = 2, caption = "Number of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         |  650 |
| source       |  322 |
| undetermined |   12 |

Number of lakes classified as source/sink

``` r
dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {prop.table(table(.$n2o.src.snk.with.all.error))} %>%
  kable(digits = 2, caption = "Proportion of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         | 0.66 |
| source       | 0.33 |
| undetermined | 0.01 |

Proportion of lakes classified as source/sink

#### 5.2.1.2 High precision temperature

``` r
# ratio simulated with measurement error in mims
S_mims_hpt_me <- rep(NA, nsim)

for( i in 1:nsim){
S_mims_hpt_me[i] <- (R_X_theta_me[i] / R_X_s_me[i]) * # N2O:Ar ratios for sample and standard
  ((1e-6 * B_me[i] * M_a_n2o_me[i] * (H_n2o * exp(2700 * (1 / (T_c_mims_me[i] + 273.15) - 1 / 298.15)))) / #N2O concentration in standard
  (1e-6 * B_me[i] * C_Ar_a * (H_ar * exp(1500 * (1 / (T_c_mims_me[i] + 273.15) - 1 / 298.15))))) * #Ar concentration in standard
  (1e-6 * B_me[i] * C_Ar_a * (H_ar  * exp(1500 *(1/(T_c_mims_me[i] + 273.15) - 1/298.15)))) / # Ar concentration in standard 
  (M_a_n2o_me[i] * B_me[i] * 1e-6 * (H_n2o  * exp(2700 * (1 / (T_c_mims_me[i] + 273.15) - 1/298.15)))) # N2o saturation value in waterbody. Using regular field temp
}


# combine into dataframe for ggplot.
S_mims_hpt_df <- tibble(S_mims_hpt_me = S_mims_hpt_me)
# save absolute error for plot
S_mims_hpt_me_abs <- S_mims_hpt_me - S_mims
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_ratio_hpt_MIMS-1.png" style="display: block; margin: auto;" />

``` r
# standard deviation of a saturation ratio value s:
sd(S_mims_hpt_me) # 0.0011
```

    ## [1] 0.0006046325

``` r
# the 95% CI is therefore the computed value +/- 2 standard deviations
sd(S_mims_hpt_me) * 1.96 # +/- 0.00216
```

    ## [1] 0.00118508

``` r
# so any ratio within 0.00216 of 1.0 cannot be discerned as a source or sink

# CV of sat
sd(S_mims_hpt_me) / mean(S_mims_hpt_me) * 100 # 0.13% CV 
```

    ## [1] 0.07097859

``` r
# standard deviation of a measurement is:
round(sd(S_mims_hpt_me_abs), 3) # 0.20 nmol L-1
```

    ## [1] 0.001

``` r
# central 95th percentile of absolute error
round(quantile(S_mims_hpt_me_abs, probs = c(0.025, 0.975)), 3) # approx +/- 0.11
```

    ##   2.5%  97.5% 
    ## -0.001  0.001

``` r
# the 95% CI for an observation with measurement error can be computed 
# approximately as the value +/- 2 standard deviations; or the central 95th 
# percentile of the absolute error distribution.
round(quantile((S_mims + S_mims_hpt_me_abs), probs = c(0.025, 0.975)), 3) # +/- nmol N2O 
```

    ##  2.5% 97.5% 
    ## 0.851 0.853

##### 5.2.1.2.1 Source-sink status

``` r
# two standard deviations = 0.3019449
# 95% CI is therefore x+/-0.3019449

#dg <- dg %>%
#  mutate(n2o.src.snk.with.all.error = case_when(
#    abs(n2o.sat.ratio - 1) < sd(C_me_ratio) ~ "undetermined",
#    TRUE ~ n2o.src.snk)) 
dg <- dg %>%
  mutate(n2o.src.snk.with.all.error = case_when(
    abs(n2o.sat.ratio - 1) < quantile(S_mims_hpt_me_abs, probs = c(0.975)) ~ "undetermined",
    TRUE ~ n2o.src.snk)) 

dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {table(.$n2o.src.snk.with.all.error)} %>%
  kable(digits = 2, caption = "Number of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         |  654 |
| source       |  323 |
| undetermined |    7 |

Number of lakes classified as source/sink

``` r
dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {prop.table(table(.$n2o.src.snk.with.all.error))} %>%
  kable(digits = 2, caption = "Proportion of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         | 0.66 |
| source       | 0.33 |
| undetermined | 0.01 |

Proportion of lakes classified as source/sink

### 5.2.2 High precision GC

#### 5.2.2.1 Standard temperature

This simulation is for field measurment of temperature for the
equilibrium concentration with the standard glass thermometer.

``` r
S_mims_hpgc_me <- rep(NA, nsim)

for( i in 1:nsim){
S_mims_hpgc_me[i] <- (R_X_theta_me[i] / R_X_s_me[i]) * # N2O:Ar ratios for sample and standard
  ((1e-6 * B_me[i] * M_a_n2o_lit_me[i] * (H_n2o * exp(2700 * (1 / (T_c_mims_me[i] + 273.15) - 1 / 298.15)))) / #N2O concentration in standard
  (1e-6 * B_me[i] * C_Ar_a * (H_ar * exp(1500 * (1 / (T_c_mims_me[i] + 273.15) - 1 / 298.15))))) * #Ar concentration in standard
  (1e-6 * B_me[i] * C_Ar_a * (H_ar  * exp(1500 *(1/(T_c_mims_me[i] + 273.15) - 1/298.15)))) / # Ar concentration in standard 
  (M_a_n2o_lit_me[i] * B_me[i] * 1e-6 * (H_n2o  * exp(2700 * (1 / (T_c_me[i] + 273.15) - 1/298.15)))) # N2o saturation value in waterbody. Using regular field temp
}

# combine into dataframe for ggplot.
S_mims_hpgc_df <- tibble(S_mims_hpgc_me = S_mims_hpgc_me)
# save absolute error for plot
S_mims_hpgc_me_abs <- S_mims_hpgc_me - S_mims
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_ratio_hpgc_MIMS-1.png" style="display: block; margin: auto;" />

``` r
# standard deviation of a saturation ratio value s:
sd(S_mims_hpgc_me) # 0.0011
```

    ## [1] 0.001071651

``` r
# the 95% CI is therefore the computed value +/- 2 standard deviations
sd(S_mims_hpgc_me) * 1.96 # +/- 0.00216
```

    ## [1] 0.002100436

``` r
# so any ratio within 0.00216 of 1.0 cannot be discerned as a source or sink

# CV of sat
sd(S_mims_hpgc_me) / mean(S_mims_hpgc_me) * 100 # 0.07% CV 
```

    ## [1] 0.1258031

``` r
# standard deviation of a measurement is:
round(sd(S_mims_hpgc_me_abs), 3) # 0.20 nmol L-1
```

    ## [1] 0.001

``` r
# central 95th percentile of absolute error
round(quantile(S_mims_hpgc_me_abs, probs = c(0.025, 0.975)), 3) # approx +/- 0.11
```

    ##   2.5%  97.5% 
    ## -0.002  0.002

``` r
# the 95% CI for an observation with measurement error can be computed 
# approximately as the value +/- 2 standard deviations; or the central 95th 
# percentile of the absolute error distribution.
round(quantile((S_mims + S_mims_hpgc_me_abs), probs = c(0.025, 0.975)), 3) # +/- nmol N2O
```

    ##  2.5% 97.5% 
    ## 0.850 0.854

##### 5.2.2.1.1 Source-sink status

``` r
# two standard deviations = 0.3019449
# 95% CI is therefore x+/-0.3019449

#dg <- dg %>%
#  mutate(n2o.src.snk.with.all.error = case_when(
#    abs(n2o.sat.ratio - 1) < sd(C_me_ratio) ~ "undetermined",
#    TRUE ~ n2o.src.snk)) 
dg <- dg %>%
  mutate(n2o.src.snk.with.all.error = case_when(
    abs(n2o.sat.ratio - 1) < quantile(S_mims_hpgc_me_abs, probs = c(0.975)) ~ "undetermined",
    TRUE ~ n2o.src.snk)) 

dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {table(.$n2o.src.snk.with.all.error)} %>%
  kable(digits = 2, caption = "Number of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         |  650 |
| source       |  322 |
| undetermined |   12 |

Number of lakes classified as source/sink

``` r
dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {prop.table(table(.$n2o.src.snk.with.all.error))} %>%
  kable(digits = 2, caption = "Proportion of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         | 0.66 |
| source       | 0.33 |
| undetermined | 0.01 |

Proportion of lakes classified as source/sink

#### 5.2.2.2 High precision temperature

The thermometers used to measure water temperature in this study have an
uncertainty of +/- 0.058C but electronic thermisters are commonly
available (e.g. YSI sondes) with an uncertainty of +/-0.01 Celsius. More
precise water temperature measurements will reduce uncertainty in the
equilibrium N2O conentration and the saturation ratio.

``` r
S_mims_hpgc_hpt_me <- rep(NA, nsim)

for( i in 1:nsim){
S_mims_hpgc_hpt_me[i] <- (R_X_theta_me[i] / R_X_s_me[i]) * # N2O:Ar ratios for sample and standard
  ((1e-6 * B_me[i] * M_a_n2o_lit_me[i] * (H_n2o * exp(2700 * (1 / (T_c_mims_me[i] + 273.15) - 1 / 298.15)))) / #N2O concentration in standard
  (1e-6 * B_me[i] * C_Ar_a * (H_ar * exp(1500 * (1 / (T_c_mims_me[i] + 273.15) - 1 / 298.15))))) * #Ar concentration in standard
  (1e-6 * B_me[i] * C_Ar_a * (H_ar  * exp(1500 *(1/(T_c_mims_me[i] + 273.15) - 1/298.15)))) / # Ar concentration in standard 
  (M_a_n2o_lit_me[i] * B_me[i] * 1e-6 * (H_n2o  * exp(2700 * (1 / (T_c_mims_me[i] + 273.15) - 1/298.15)))) # N2o saturation value in waterbody. Using high precision field temp
}


# combine into dataframe for ggplot.
S_mims_hpgc_hpt_df <- tibble(S_mims_hpgc_hpt_me = S_mims_hpgc_hpt_me)

# save absolute error for plot
S_mims_hpgc_hpt_me_abs <- S_mims_hpgc_hpt_me - S_mims
```

<img src="DG_sensitivity_to_measurement_error_files/figure-gfm/plot_me_ratio_hpgc_hpt_MIMS-1.png" style="display: block; margin: auto;" />

``` r
# standard deviation of a saturation ratio value s:
sd(S_mims_hpgc_hpt_me) # 0.0006603624
```

    ## [1] 0.0006046325

``` r
# the 95% CI is therefore the computed value +/- 2 standard deviations
sd(S_mims_hpgc_hpt_me) * 1.96 # +/- 0.00129431
```

    ## [1] 0.00118508

``` r
# so any ratio within 0.00129 of 1.0 cannot be discerned as a source or sink
```

``` r
# standard deviation of a measurement is:
round(sd(S_mims_hpgc_hpt_me_abs), 3) # 0.20 nmol L-1
```

    ## [1] 0.001

``` r
# central 95th percentile of absolute error
round(quantile(S_mims_hpgc_hpt_me_abs, probs = c(0.025, 0.975)), 3) # approx +/- 0.11
```

    ##   2.5%  97.5% 
    ## -0.001  0.001

``` r
# the 95% CI for an observation with measurement error can be computed 
# approximately as the value +/- 2 standard deviations; or the central 95th 
# percentile of the absolute error distribution.
round(quantile((S_mims + S_mims_hpgc_hpt_me_abs), probs = c(0.025, 0.975)), 3) # +/- nmol N2O
```

    ##  2.5% 97.5% 
    ## 0.851 0.853

##### 5.2.2.2.1 Source-sink status

``` r
# two standard deviations = 0.3019449
# 95% CI is therefore x+/-0.3019449

#dg <- dg %>%
#  mutate(n2o.src.snk.with.all.error = case_when(
#    abs(n2o.sat.ratio - 1) < sd(C_me_ratio) ~ "undetermined",
#    TRUE ~ n2o.src.snk)) 
dg <- dg %>%
  mutate(n2o.src.snk.with.all.error = case_when(
    abs(n2o.sat.ratio - 1) < quantile(S_mims_hpgc_hpt_me_abs, probs = c(0.975)) ~ "undetermined",
    TRUE ~ n2o.src.snk)) 

dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {table(.$n2o.src.snk.with.all.error)} %>%
  kable(digits = 2, caption = "Number of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         |  654 |
| source       |  323 |
| undetermined |    7 |

Number of lakes classified as source/sink

``` r
dg %>% 
  filter(sample.source == "DG", visit.no == 1, sitetype == "PROB") %>%
  {prop.table(table(.$n2o.src.snk.with.all.error))} %>%
  kable(digits = 2, caption = "Proportion of lakes classified as source/sink")
```

| Var1         | Freq |
|:-------------|-----:|
| sink         | 0.66 |
| source       | 0.33 |
| undetermined | 0.01 |

Proportion of lakes classified as source/sink

# 6 Summary of ratio results

Precision of the N2O saturation ratio can be improved using high
precision GC analytics and N2O-free headspace gas (e.g. helium,
nitrogen) but the greatest improvement is seen when dissolved N2O is
directly measured using MIMS.

``` r
#results <- tibble(ratio_sd = c(0.055,
#                               0.048,
#                               0.029,
#                               0.001207747,
#                               0.001207747,
#                               0.000690139),
#                  method = c("GC",
#                             "GC + helium",
#                             "High precision GC + helium",
#                             "MIMS",
#                             "MIMS + high precision GC",
#                             "MIMS + high precision GC + \nhigh precision water temp"))


#results %>%
#  mutate(lower.95 = C_ratio - (1.96 * ratio_sd),
#         upper.95 = C_ratio + (1.96 * ratio_sd),
#         C_ratio = C_ratio) %>%
#  ggplot(aes(C_ratio, method)) +
#  geom_point() +
#  geom_segment(aes(x=lower.95, xend = upper.95, y=method, yend=method)) +
#  xlab("True saturation ratio (black dot) +/- measurement error (1.96 standard deviations))") +
#  theme(axis.text.x = element_text(angle = 45, vjust = 0.9, hjust = 1))

results <- tibble(GC_me= S_a_me_abs,
                  GC_He_me = S_pg_me_abs,
                  GC_hpt_me = S_a_hpt_me_abs,
                  GC_He_hpt_me = S_pg_hpt_me_abs,
                  HPGC_He_me = S_pg_hpgc_me_abs,
                  HPGC_He_hpt_me = S_pg_hpgc_hpt_me_abs,
                  MIMS_me = S_mims_me_abs,
                  MIMS_hpt_me = S_mims_hpt_me_abs,
                  MIMS_HPGC_me = S_mims_hpgc_me_abs,
                  MIMS_HPGC_HPT_me = S_mims_hpgc_hpt_me_abs) %>%
  pivot_longer(cols = ends_with("me"), names_to = "Method", values_to = "Error")


results %>%
  mutate(Method = factor(Method)) %>%
  mutate(Method = fct_relevel(Method, c("MIMS_HPGC_HPT_me",
                                        "MIMS_HPGC_me",
                                        "MIMS_hpt_me",
                                        "MIMS_me",
                                        "HPGC_He_hpt_me",
                                        "HPGC_He_me",
                                        "GC_He_hpt_me",
                                        "GC_hpt_me",
                                        "GC_He_me",
                                        "GC_me"
                                        ))) %>%
  ggplot(aes(x = Error, y = Method)) +
  stat_pointinterval() +
  scale_y_discrete(labels=c("GC_me" = "Standard GC, ambient air, \n standard field temperature",
                            "GC_He_me" = "Standard GC, helium, \n standard field temperature",
                            "GC_hpt_me" = "Standard GC, ambient air, \n high precision temperature",
                            "GC_He_hpt_me" = "Standard GC, helium, \nhigh precision temperature",
                            "HPGC_He_me" = "High precision GC, helium, \nstandard field temperature",
                            "HPGC_He_hpt_me" = "High precision GC, helium, \nhigh precision temperature",
                            "MIMS_me" = "MIMS: standard GC, \nstandard field temperature",
                            "MIMS_hpt_me" = "MIMS: standard GC, \nhigh precision temperature",
                            "MIMS_HPGC_me" = "MIMS: high precision GC, \nstandard field temperature",
                            "MIMS_HPGC_HPT_me" = "MIMS: high precision GC, \nhigh precision temperature")) +
  xlab("Absolute error in saturation ratio") +
  ylab("")
```

![](DG_sensitivity_to_measurement_error_files/figure-gfm/final_summary-1.png)<!-- -->

Because the MIMS method is so much more precise than the traditional
method, the differences between the MIMS-based variations above are
difficult to see. Below, just the MIMS-based variations are compared.

Precision of the N2O saturation ratio can be improved using high
precision GC analytics and N2O-free headspace gas (e.g. helium,
nitrogen) but the greatest improvement is seen when dissolved N2O is
directly measured using MIMS.

``` r
results_MIMS <- tibble(MIMS_me = S_mims_me_abs,
                       MIMS_hpt_me = S_mims_hpt_me_abs,
                       MIMS_HPGC_me = S_mims_hpgc_me_abs,
                       MIMS_HPGC_HPT_me = S_mims_hpgc_hpt_me_abs) %>%
  pivot_longer(cols = ends_with("me"), names_to = "Method", values_to = "Error")


results_MIMS %>%
  mutate(Method = factor(Method)) %>%
  mutate(Method = fct_relevel(Method, c("MIMS_HPGC_HPT_me",
                                        "MIMS_HPGC_me",
                                        "MIMS_hpt_me",
                                        "MIMS_me"
                                        ))) %>%
  ggplot(aes(x = Error, y = Method)) +
  stat_pointinterval() +
  scale_y_discrete(labels=c("MIMS_me" = "MIMS: standard GC, \nstandard field temperature",
                            "MIMS_hpt_me" = "MIMS: standard GC, \nhigh precision temperature",
                            "MIMS_HPGC_me" = "MIMS: high precision GC, \nstandard field temperature",
                            "MIMS_HPGC_HPT_me" = "MIMS: high precision GC, \nhigh precision temperature")) +
  xlab("Absolute error in saturation ratio") +
  ylab("")
```

![](DG_sensitivity_to_measurement_error_files/figure-gfm/final_summary_MIMS-1.png)<!-- -->

# 7 Update data with uncertainty info

Finally, we update the dg data frame to include information on estimated
measurement error

``` r
dg <- dg %>%
  mutate(# recompute using error from uncertainty simulations
         n2o.src.snk.error = case_when(
           abs(n2o.sat.ratio - 1) < quantile(S_a_me_abs, probs = c(0.975)) ~ "undetermined",
           TRUE ~ n2o.src.snk))

save(dg, file = "./../inputData/dg.RData")
```

# 8 Session Info

``` r
sessionInfo()
```

    ## R version 4.4.0 (2024-04-24)
    ## Platform: x86_64-pc-linux-gnu
    ## Running under: Ubuntu 22.04.4 LTS
    ## 
    ## Matrix products: default
    ## BLAS:   /usr/lib/x86_64-linux-gnu/openblas-pthread/libblas.so.3 
    ## LAPACK: /usr/lib/x86_64-linux-gnu/openblas-pthread/libopenblasp-r0.3.20.so;  LAPACK version 3.10.0
    ## 
    ## locale:
    ##  [1] LC_CTYPE=en_US.UTF-8       LC_NUMERIC=C              
    ##  [3] LC_TIME=en_US.UTF-8        LC_COLLATE=en_US.UTF-8    
    ##  [5] LC_MONETARY=en_US.UTF-8    LC_MESSAGES=en_US.UTF-8   
    ##  [7] LC_PAPER=en_US.UTF-8       LC_NAME=C                 
    ##  [9] LC_ADDRESS=C               LC_TELEPHONE=C            
    ## [11] LC_MEASUREMENT=en_US.UTF-8 LC_IDENTIFICATION=C       
    ## 
    ## time zone: Etc/UTC
    ## tzcode source: system (glibc)
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ##  [1] usmap_0.7.1      sf_1.0-16        janitor_2.2.1    readxl_1.4.3    
    ##  [5] gridExtra_2.3    ggExtra_0.10.1   tidybayes_3.0.7  lubridate_1.9.3 
    ##  [9] forcats_1.0.0    stringr_1.5.1    dplyr_1.1.4      purrr_1.0.2     
    ## [13] readr_2.1.5      tidyr_1.3.1      tibble_3.2.1     tidyverse_2.0.0 
    ## [17] ggpubr_0.6.0     ggplot2_3.5.1    kableExtra_1.4.0 bayesplot_1.11.1
    ## [21] rmarkdown_2.26   knitr_1.46      
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] DBI_1.2.2            rlang_1.1.3          magrittr_2.0.3      
    ##  [4] snakecase_0.11.1     e1071_1.7-14         compiler_4.4.0      
    ##  [7] systemfonts_1.2.1    vctrs_0.6.5          pkgconfig_2.0.3     
    ## [10] arrayhelpers_1.1-0   fastmap_1.1.1        backports_1.4.1     
    ## [13] labeling_0.4.3       utf8_1.2.4           promises_1.3.0      
    ## [16] tzdb_0.4.0           xfun_0.43            highr_0.10          
    ## [19] later_1.3.2          broom_1.0.5          R6_2.5.1            
    ## [22] stringi_1.8.3        car_3.1-3            cellranger_1.1.0    
    ## [25] Rcpp_1.0.12          httpuv_1.6.15        timechange_0.3.0    
    ## [28] tidyselect_1.2.1     rstudioapi_0.16.0    abind_1.4-5         
    ## [31] yaml_2.3.8           miniUI_0.1.1.1       lattice_0.22-6      
    ## [34] shiny_1.8.1.1        withr_3.0.0          posterior_1.6.1     
    ## [37] coda_0.19-4.1        evaluate_0.23        units_0.8-5         
    ## [40] proxy_0.4-27         ggdist_3.3.2         xml2_1.3.6          
    ## [43] pillar_1.9.0         carData_3.0-5        tensorA_0.36.2.1    
    ## [46] KernSmooth_2.23-22   checkmate_2.3.2      distributional_0.5.0
    ## [49] generics_0.1.3       hms_1.1.3            munsell_0.5.1       
    ## [52] scales_1.3.0         xtable_1.8-4         class_7.3-22        
    ## [55] glue_1.7.0           tools_4.4.0          ggsignif_0.6.4      
    ## [58] usmapdata_0.3.0      cowplot_1.1.3        grid_4.4.0          
    ## [61] colorspace_2.1-0     Formula_1.2-5        cli_3.6.2           
    ## [64] fansi_1.0.6          svUnit_1.0.6         viridisLite_0.4.2   
    ## [67] svglite_2.1.3        gtable_0.3.5         rstatix_0.7.2       
    ## [70] digest_0.6.35        classInt_0.4-10      farver_2.1.1        
    ## [73] htmltools_0.5.8.1    lifecycle_1.0.4      mime_0.12
