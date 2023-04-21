---
layout: default
altair-loader:
  altair-chart-2: "charts/Philly_mode_income.json"
  altair-chart-3: "charts/Philly_mode_age.json"
hv-loader:
  hv-chart-1: ["charts/MAE.html", "500"]
  hv-chart-2: ["charts/tabs.html", "500"]
  hv-chart-3: ["charts/tabs0.html", "500"]
  hv-chart-4: ["charts/MAE_comparison.html", "600"]
---

# Welcome!

I am **Yiping Ying**, currently a student of [Master of Urabn Spatial Analytics](https://www.design.upenn.edu/urban-spatial-analytics) at UPenn.

This website is used to display my capstone project at this program.

# I. Motivation

Bike sharing is a new type of micro-transportation that has emerged in urban areas in recent years. Its appearance provides a new way to solve the “last mile” problem. However, while bringing us convenience, shared bicycles also have some difficult problems to solve, among which the re-balancing problem is more prominent.

The problem of re-balancing arises because the number of shared bicycles is not evenly distributed among various sites. In some remote sites, people ride bikes out of the site, but no one returns the shared bicycles. A common solution is that the shared bicycle companies will send some trucks to transport the shared bicycles to these sites to complete the supplement of shared bicycles. The question with this type of solution is how to distribute the volume of these trucks to maximize efficiency?

In general, there are two steps in truck-based rebalancing approaches, i.e., demand prediction and station rebalancing. First, it is crucial to accurately predict the demand at each station to foresee the bike and dock availability in the future. Second, it is important to design effective strategies for truck operators to reposition bikes among stations.

This project will focus on the first step of the problem by taking [Indego](https://www.rideindego.com/), a bike-sharing system in the Philadelphia area, as a research sample, on which we will use different models (OLS, Random Forest, ARIMA, Neural Networks, etc.) to make accurate demand forecasts. Indego started operation on April 23, 2015, with 125 stations and 1,000 bicycles. After 7 years of development, the function of this system has tended to be comprehensive, which is appropriate for research. In this analysis, we select a **5-week** period from **May 17** to **June 20**, **2021** for temporal/spatial analysis.

This section will show embedding interactive charts produced using [Altair](https://altair-viz.github.io) and [Hvplot](https://hvplot.pyviz.org/).

# II. Exploratory Data Analysis

## 2.1 Temporal Pattern

![fig1]({{ site.url }}{{ site.baseurl }}/assets/img/fig1.png)

There is clearly a daily periodicity and there are lull periods on weekends. Notice that the weekend near the 31st of May (Memorial Day) doesn’t have the same dip in activity.

![fig2]({{ site.url }}{{ site.baseurl }}/assets/img/fig2.png)

From the morning, the number of rides gradually increases, and as we can see, the number peaks in the late afternoon (16-18), this may be attributed to the evening rush in Philadelphia, the number falls quickly. 

![fig3]({{ site.url }}{{ site.baseurl }}/assets/img/fig3.png)

For weekends and weekdays, the trends of rides in the day are almost the same, while the rides in the daytime (5-18) on the weekends are much less than that on the weekdays.

![fig4]({{ site.url }}{{ site.baseurl }}/assets/img/fig4.png)

![fig5]({{ site.url }}{{ site.baseurl }}/assets/img/fig5.png)

As the *Indego* shared bike trip duration density suggests, most of the trips’ have duration less than 180 minutes (3 hours), therefore we discard trips with longer duration (more than 3 hours). (Trips: **97435**)

## 2.2 Spatial Pattern

In this part, we check the spatial characteristics in the following categories:

- **Internal Characteristics**

- **Demographic (Census)**

- **Amenities/Disamenities**

- **Transportation network**

- **Neighboring stations**

### 2.2.1 Internal Characteristics

- Active Station: **139**

- Number of Docks in Each Station, Philadelphia 2021

![fig6]({{ site.url }}{{ site.baseurl }}/assets/img/fig6.png)

Below are the scatter plots of the stations, we can see that they are clustered around the center city. As for the trips, a station tends to have almost the same number of start and end trips, given the high similarity shown in the plots. Correspondingly, in the modeling part, we only select the number of start trips.

<div id="hv-chart-3"></div>

### 2.2.2 Census demographic data

In this part, we download data from [American Community Survey 5-Year Data]( American Community Survey 5-Year Data (2009-2021) (census.gov)) in Philadelphia (2021) to check the relation between the social/economical features and the shared bike demand.

- Choropleth of Social/Economical Features

<div id="hv-chart-2"></div>

From the choropleths, we can see that the existing shared bike stations generally locate at the places where the people tend to be younger, having higher income and no car. The below 2 `hvplot` interactive charts present this point more accurately.

- Chart of Median Income vs Transit Mode Selection in Philadelphia.

<div id="altair-chart-2"></div>

- Chart of Median Age vs Transit Mode Selection in Philadelphia.

<div id="altair-chart-3"></div>

### 2.2.3 Amenities/Disamenities

Add two new features:

1. Distances to the nearest **10 restaurants** from [Open Street Map](https://wiki.openstreetmap.org/wiki/Map_Features);

2. Whether the station is located within Center City.

- ***Plot:*** Mean Distance to Nearest 10 Restaurants (log transformed)

![fig8]({{ site.url }}{{ site.baseurl }}/assets/img/fig8.png)

### 2.2.4 Transportation Network

Add a feature that calculates the distance to the nearest intersections.

- Intersections

![fig9]({{ site.url }}{{ site.baseurl }}/assets/img/fig9.png)

- ***Plot:*** Mean Distance to Nearest 3 Intersections (log transformed)

![fig10]({{ site.url }}{{ site.baseurl }}/assets/img/fig10.png)

### 2.2.5 Neighboring Stations

- **Spatial Lag**: a feature that encodes the fact that demand for a specific station is likely related to the demand in neighboring stations.

We add two new features:

1. The average distance to the nearest 5 stations

2. The average trip total for the nearest 5 stations

- ***Plot***: Mean Distance to Nearest 5 Stations (log transformed)

![fig11]({{ site.url }}{{ site.baseurl }}/assets/img/fig11.png)

- **Check the correlation between the trip counts and the spatial lag**

![fig12]({{ site.url }}{{ site.baseurl }}/assets/img/fig12.png)

From the figure, we can see that there is a linear correlation between the trip counts and the spatial lag.

### 2.2.6 Correlation Matrix

![fig13]({{ site.url }}{{ site.baseurl }}/assets/img/fig13.png)

# III. Modeling

In this section we will try several models to predict the *Indego* Shared Bike Demand. Following is the plan:

## 3.1 General Demand Prediction

- **OLS** Models (Base line)

  - Unregularized Linear Regression

  - Ridge Regression
 
  - Lasso Regression

  - Elastic Net Regression

- **Random Forests**

  - Fit a random forest model, using cross validation to optimize (some) hyperparameters;

  - Find out the most important features in the random forest model;

  - Compare to the baseline linear regression models.

## 3.2 Time Series Prediction:

Predict overall *MAE* for all **139** stations, and select the station: ***Amtrak 30th Street Station*** as a case study.

- **GRU**

  -	GRU architecture built on PyTorch.
  
  -	Generate a panel of interval 60 minutes. 

  -	Split Train/Test Set ratio: **0.8**.

  -	Use **3 week’s** records to predict **one hour’s** demand.

  -	Select a specific station: Amtrak 30th Street Station as an example.

  -	Metrics: *MSE*: 1.96; *MAE*: **1.03**; *RMSE*: 1.40; *R-Squared*: 0.17

  -	Tuning hyperparameter

![Figure_1]({{ site.url }}{{ site.baseurl }}/assets/img/Figure_1.png)

- **GCN**

<div id="hv-chart-1"></div>

<div id="hv-chart-4"></div>

- **GCN-GRU**


