---
layout: default
altair-loader:
  altair-chart-2: "charts/Philly_mode_income.json"
  altair-chart-3: "charts/Philly_mode_age.json"
  altair-chart-4: "charts/rebalance.json"
hv-loader:
  hv-chart-1: ["charts/MAE.html", "500"]
  hv-chart-2: ["charts/tabs.html", "500"]
  hv-chart-3: ["charts/tabs0.html", "500"]
  hv-chart-4: ["charts/MAE_comparison.html", "450"]
---

# Welcome!

I am **Yiping Ying**, currently a student of [Master of Urabn Spatial Analytics](https://www.design.upenn.edu/urban-spatial-analytics) at UPenn.

This website is used to display my capstone project at this program.

# I. Motivation

Bike sharing is a new type of micro-transportation that has emerged in urban areas in recent years. Its appearance provides a new way to solve the “last mile” problem. However, while bringing us convenience, shared bicycles also have some difficult problems to solve, among which the re-balancing problem is more prominent.

The problem of re-balancing arises because the number of shared bicycles is not evenly distributed among various sites. In some remote sites, people ride bikes out of the site, but no one returns the shared bicycles. A common solution is that the shared bicycle companies will send some trucks to transport the shared bicycles to these sites to complete the supplement of shared bicycles. The question with this type of solution is how to distribute the volume of these trucks to maximize efficiency?

In general, there are two steps in truck-based rebalancing approaches, i.e., demand prediction and station rebalancing. **First**, it is crucial to accurately predict the demand at each station to foresee the bike and dock availability in the future. **Second**, it is important to design effective strategies for truck operators to reposition bikes among stations.

This project will focus on the first step of the problem by taking [Indego](https://www.rideindego.com/), a bike-sharing system in the Philadelphia area, as a research sample, on which we will use different models (OLS, Random Forest, ARIMA, Neural Networks, etc.) to make accurate demand forecasts. Indego started operation on April 23, 2015, with 125 stations and 1,000 bicycles. After 7 years of development, the function of this system has tended to be comprehensive, which is appropriate for research. In this analysis, we select a **5-week** period from **May 17** to **June 20**, **2021** for temporal/spatial analysis.

![procedure]({{ site.url }}{{ site.baseurl }}/assets/img/procedure.png)

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

- Predict overall *MAE* for all **139** stations

- select the station: ***Amtrak 30th Street Station*** as a case study.

## 3.1 **GRU** : Gated Recurrent Units

Gated Recurrent Unit. Recurrent Neural Networks (RNNs) can model the dependency of time series effectively. However, the traditional RNN models have limitations for long-term prediction due to gradient vanishing and gradient exploding problems. Although Long Short-term Memory (LSTM) can address these challenges, it has the defect of more training time consumption special for complex structures. Hence, we introduce Gated Recurrent Unit (GRU), the variant of RNNs, to model the temporal dependency. Compared with LSTM, GRU model has a relatively simple structure with fewer parameters and faster training speed.

## 3.2 **GCN** ： Graph Convolutional Networks

In spatial correlation modeling, current works usually employ grid to divide the urban area, converting urban data to Euclidean domains, and then use Convolutional Neural Networks (CNN) to model spatial correlation. However, in our problem, the spatial distribution of bike stations is irregular and in non-Euclidean domains. Hence, we introduce graph structure to model the spatial distribution of bike stations and exploit Graph Convolutional Networks (GCN) to model spatial correlation. 

## 3.3 General Results

- ***MAE***: by interval, all stations

<div id="hv-chart-4"></div>

From the figures, we can see that for GRU, MAE changes periodically, generally having the lowest point in the morning, while in the afternoon, the number becomes bigger, which may be resulted from the fact that people ride more bikes in the afternoon. For GCN, the magnitude of the value change is not very large, basically around 0.8-1.0. However, this fact also indicates the weakness of the GCN model, even though it is stable.

- ***MAE***: by station, all intervals

<div id="hv-chart-1"></div>

The scatter patterns of the two models also suggest what we have discovered in the above interval MAE check. One thing the two have in common is that they tended to have larger MAE in center city, where the stations generally have more docks and more rides which is correspondent with the above. For the accuracy, GRU performs better on most stations than GCN.

## 3.3 **GCN-GRU**

With both spatial correlation and temporal dependency modeling, we build a spatiotemporal graph neural network (GCN-GRU) to accurately predict the bike demand. Following figure shows the model architecture of GCN-GRU. The objective function of GCN-GRU is to maximize the likelihood of predicting the time series in a future period of time. Specifically, the GCN-GRU model consists of two parts, i.e. GCN-based spatial correlation modeling and GRU-based temporal dependency modeling. 

![gcn]({{ site.url }}{{ site.baseurl }}/assets/img/gcn.png)

- **Case Study**: *Amtrak 30th Street Station*

Finally, we conduct a case study analysis to further understand the benefit and limits of our demand prediction. The selected station is the Amtrak 30th Street Station (18 docks).

![amtrak]({{ site.url }}{{ site.baseurl }}/assets/img/amtrak.png)

This *indego* bike station is just beside the Amtrak station and is in the university city. Its location (beside a transportation hub, close to center city), the highly educated ratio and younger median age in this block group all suggests that the number of bike trips start here will be large, since shared bikes serve as a kind of assisted transportation. Also, the number of docks is also very large (18), based on the last section, 

All of these would contribute to inaccurate demand prediction here. Therefore, we tested our new model GCN-GRU, which synthesized the two that have been tested, and wanted to see whether the new model could combine the advantages of both and improve the accuracy of demand prediction. Below are the test results.

![GRU_result]({{ site.url }}{{ site.baseurl }}/assets/img/GRU_result.png)

![GCN_result]({{ site.url }}{{ site.baseurl }}/assets/img/GCN_result.png)

![GCN-GRU]({{ site.url }}{{ site.baseurl }}/assets/img/GCN-GRU.png)

- ***MAE***:

  - **GRU**: 1.03
  - **GCN**: 1.23
  - **GCN-GRU**: **0.98***

The above figures show the results of both baseline models and the new combined one. Generally, all three performed well, but the GCN-GRU is the best, with the lowest MAE 0.98. The other two have MAE around 1. We can also see, that GRU model can capture the periodical change in MAE, while the GCN with geographic information can depict the detail of changes (especially in the scale of hours in a day), even though the total MAE is slightly higher. GCN-GRU has both the strengths, it can predict very well when the demand is low (in the morning) and can catch the trend. However, when the demand is high, in some extreme cases like late afternoon on June 15th, 2021, the demand was historically high (9 in an hour), the prediction was not so accurate, with MAE of 3 at that time. This is a general problem, maybe more advanced techniques should be applied here, or we should just filter them out as outliers.

Apart from the generally good MAE, one thing should not be ignored. The number of demands in one station is of integer form, while our prediction is decimal. Even the MAE is improved, how should we interpret the number? Should 0.98 be the same as 1.23 when making decisions? This also requires further studies.

# IV. Rebalancing

- Define the reasonable number of the bikes in a station: **20%** to **65%**

- ***Integer Linear Programming***

<div id="altair-chart-4"></div>

For the station rebalancing, we select data of 3pm, June 16th, 2021, as the study period, as afternoon is generally the time that rebalancing is needed to prepare for demand increase in the later evening rush hours. We select the station: 19th & Lombard as our start point, with 3 bikes on the truck.

Our selection criterion of the reasonable range of bikes in a station is around 20%-65% of total docks in that station. Based on this, we filtered out station need rebalancing, as shown above. In this figure, the yellow, positive number indicates that the station needs bikes suppled from other stations, while the blue, negative ones are those that can give away their extra bikes.

We make the model based on the objective and constraints in section V. As for the station rebalancing results, with the help of *Gurobi* Optimizer, we find that the shortest path to re-balance all the overdemand and oversupply stations is ***56.409km***, travelling between 80 stations.

# V. Summary

Overall, our analysis aimed to take a deep dive into Philadelphia’s Indego bike sharing operations by using Graph Neural Networks and mixed integer programming techniques to optimize the rebalancing of bikes in a way which meets anticipated demands, alleviates surpluses and shortages in the system, and remains cost efficient. While we focused on a few select stations/regions and time periods, the models created are scalable and can be extended to other boroughs or time periods. By providing key insights into the requirements of maintaining such a dynamic system, we ultimately sought to demonstrate the opportunities which exist to further enhance the value of friendly and efficient transportation shared transportation systems in Philadelphia.

