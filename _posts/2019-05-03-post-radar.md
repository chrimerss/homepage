---
layout: post
title: "Radar Project in Singapore"
categories:
  - Hydroinformatics
  - hydrometeorology
  - deep learning
tags:
  - machine learning
  - deep learning
  - hydroinformatics
  - hydrometeorology
---

# Radar Project in Singapore

## Contents

### 1. [Introduction](#introduction)  

#### 1.1 [Background](#background) 

#### 1.2 [Radar Information](#info)  

### 2. [Radar Observation](#observation)  

#### 2.1 [Z-R relationship](#z-r)  

#### 2.2 [Image Integration](#img_inte)  

### 3. [Radar Nowcast](#nowcast)  

#### 3.1 [Convection](#convection)  

#### 3.2 [Growth and Decay](#growthdecay)  

#### 3.3 [Memory Integration](#memo)  

### 4. [Radar Improvement](#improvement)  

### 5. [Miscellaneous](#misc)  


## _Introduction_<a name="introduction"></a>

<figure style="width: 60%" class="align-center">
<img align="center" src="{{ site.url }}/images/radarproject.png"/>
<figcaption>Fig.1 radar project overview</figcaption>
</figure>

This Project ["RADAR RAINFALL MONITORING AND NOWCASTING SYSTEM FOR URBAN FLOOD MANAGEMENT IN SINGAPORE"](http://www.h2i.sg/radar-rainfall-monitoring-and-nowcasting-system-for-urban-flood-management-in-singapore) is funded by PUB, Singapore. H2i (Hydroinformatics Institute) is going to hand over a peoduct including three X-band radars that cover Marina Catchment. The radar system consists of two essential parts, one is for rainfall observation, and the other is for nowcast. These two parts will be discussed in the following charpters.

<figure style="width: 60%" class="align-center">
<img align="center" src="{{ site.url }}/images/radar_intro.png"/>
<figcaption>Fig.2 radar project introduction</figcaption>
</figure>

### Background<a name="background"></a>

The first question: How radar measures rainfall?

As we know, radar is capable of detecting the obstacle and bounce back the signal. With this analogy, rain drop above the ground will be detected by radar signals, and also drop size etc. Then the next thing left is to construct a relationship between radar reflectivity and rainfall rate, known as Z-R relashionshp, which will be covered in [Z-R Relationship](#z-r)

Then we need to answer why radar?

Traditionally, we measure rainfall through some ground based devices, called gauge data. gauge data are usually considered as ground truth data. However, if the stakeholders need information like how is it possible to get inundated in city, or the government wants to alert citizens that there will be flooded, then gauge is of course not able to send off these signals. We need something more than ground! Here is why radar comes in to save lives!

Additionally, rain gauge data will not give you a full map of rain rate over a city, which means they are spatial constrained. But radar, on contrast, has a higher resolution over the domain and specialize at producing spatial rainfall rate.

After the mechanism explained, I would like to emphisize some limitations that radar would be confined or conditions where radar is not able to detect rainfall.

One severe problem for radar is that, its sight often get blocked by some skyscrapers especially in mega-cities.

Another problem is sometimes, radar imposes fake signals like clusters or non-precipitating dots.

In order to build a real-time radar monitoring system, more challenges is coming for the sake of computations. Appropriate algorithms should be selected for the tradeoff between accuracy and effort.

### Radar Information<a name="info"></a>

Three X-band radars are built in the western, eastern and norther part of Singapore, operating in observing rain and producing forecast every two minutes. Spatially, three radars will integrate together and produce a map with 100x100 meters resolution over 100kmx100km domain.

X-band radar contains around 9 layers of information, like rainfall, reflectivity, wind vector, etc. But in the first phase of this project, we only consider using reflectivity to calculate rainfall.

## _Radar Observation_<a name="observation"></a>

### Z-R Relationship<a name="z-r"></a>

Z-R relationship directly maps reflectivity with rain rate. We construct this relationship based on emperial expression that has been provided with lots of practices.

$$
Z=AR^b
$$

To calibrate this relationship, we selected several year-long events and calibrated the A, b parameter for each of three radar.

### Image Integration<a name="img_inte"></a>

Three radars need to be integrated to produce only one spatially distributed rainfall map. Integration is based on maximizing the coverage of the radars. As described before, some radars fall into the poor sight, which means they are blocked by nearby buildings or telecommunications.

<figure style="width: 50%" class="align-center">
<img align="center" src="{{ site.url }}/images/blocked_radar.png"/>
<figcaption>Fig.3 representation of blocked radar image</figcaption>
</figure>

The figure above gives an illustrative example that one radar is blocked known as blind spots (radius)

To compensate for this, we analyzed the blind spots for each radar, and try to rely on each other to correct the blinds. Here we used weighted average value to determine which area is highly dependent on which radar.

After this correction, the final map looks like this...

<figure style="width: 50%" class="align-center">
<img align="center" src="{{ site.url }}/images/blind_spots_correction.png"/>
<figcaption>Fig.4 radar blind spots correction</figcaption>
</figure>

## _Radar Nowcast_<a name="nowcast"></a>

Radar nowcast plays an important role in releasing alert in the city if a potential flood is going to happen. The important signal for predicting rain over the large domain is relied on the movement of clouds.

With analogy to water quality model, the movement of clouds similarly will be decomposed as [convection](#convection) and [diffusion](#growthdecay).

### Convection<a name="convection"></a>

Singapore, mostly suffers from convective storms, which simply conceptualize as decomposition of movement to velocity driven. Though this method is not accurate enough, it still supports us a big picture about where those rainy clouds to move.

The velocity for prediction is obtained by analyzing the movement of most clouds in the image domain. and then assume the persistence of Euler method, which freezes the last image (base image). After gaining information (velocity, moving direction), we forecast the cloud at next time step. With this method, we provide 45 minutes long-term forecasting in real-time(2 minutes latency).

### Growth & Decay<a name="growthdecay"></a>

Even though convective storm dominates over tropical regions like Singapore, the sophisticated atmosphiric model doesnot only account for 2-dimensional movement. As well, the growth and decay of the clouds should be taken into account. Since then, we propose a linear growth and decay model with following formula:
<p align="center">
F_{T+i}^j=(C_i^1G_T^j+C_i^2R_T^j)/R_T^j
</p>

### Memory Integration<a name="memo"></a>

Memory integration comes into play because we observed the effect of previous velocity states. For instance, the continuity equation suggests that nothing will occur abruptly or dispear suddenly. This give us insights about using the combination of previous states of velocities to enhance our forecast model.

## _Radar Improvements_<a name="improvement"></a>

Obviously, there are bunch of ways to improve our existing radar performance. we will project to address these problems by alternating methods, even deep learning if possible.

1. For radar observation

In the future, we will be given more "reliable" data and not limited to ground proof gauge data. NEA will provide us another X-band radar data to better cross-validate data.

In such sense, to deal with multi-sourced data, it becomes obvious to utilize [Kalman Filter](https://en.wikipedia.org/wiki/Kalman_filter) for better data management.

Besides, we can also improve Z-R relationship with some machine learning techniques like Artificial Neural Network(ANN) or Random Forest (RF), but in my perspective, I prefer to use random forest because RF does not suffer from over-training, and should be enough to compensate all situations.

Meanwhile, if it is needed to clean radar raw data — reflectivity, we can train our data with autoencoder network before feeding into rainfall calculation. But I suspect that the training process and even evaluation will dramatically increase the calculation time. As for the real-time forecast, we may not be able to afford it.

2. Radar Nowcast

Here is more appropriate to use deep learning model to forecast. It turns out some researches have been conducted in this topic. Mostly, [LSTM](https://colah.github.io/posts/2015-08-Understanding-LSTMs/) (known as long short-term memory) fits into this topic, because it can recognize and memorize long sequence. Before actually implement this, I will post some repos on [github](https://github.com/chrimerss/RadarEnhancement) to see the possibility.


## _Miscellaneous_<a name="misc"></a>

### Statistical Measurement

This chapter, I will introduce some statistical measurement used in this project, and also very wide-approached in the research. I will split this part into comparing with rain gauge which corresponds to point time series comparison and image comparison which is multi-dimensional statistics.

#### Radar Gauge Comparison

1. Root Mean Square Error

I believe everyone is familliar with this term RMSE. This should be introduced in high scholl when we measure how close two arrays are compared, like in calculating the variance.

2. Total Volume Ratio(TVR)

TVR is intuitively understandable. It compares the total rainfall observed by gauge and radar over a specific time duration (say an event). This statistic is useful when you want to emphisize whether radar is over-estimated or under-estimated. We mentioned that radar fails when it is blocked by high buildings, and typically in such regions, radar is underestimated comparing to gauge. Here we provide a good example showing the way to stress that radar is unreliable.

<figure style="width:60%" class="align-center">
<img src="{{ site.url }}/images/TVMcompare.png">
<figcaption>Fig.5 Total Volumn Ratio</figcaption>
</figure>

3. Mean Absolute Error(MAE)

Again, this statistic is not complex. I will not explain it any further.

I want to make a point here that above statistics I mentioned have problem when accounting for different rainfall rate during event. For instance, one location falls more rainfall than other, would you expect this location has smaller error than other rain-free location?

Of course not, rain-free location has zero error but more rainfall brings with larger error. But the thing is, we donot care about rain-free areas, we only want to know how our rainfall measured by radar is close to gauge. Then we may need a trick — Normalized statistics.

Another technique here we discovered is that, these statistics are better during large events which means more rainfall, while worse during small rainfall. This will refer to the tipping bucket effect of gauge data. This graph to illustrate here is amazzzing...

<figure style="width:60%" class="align-center">
<img src="{{ site.url }}/images/R2.png">
<figcaption>Fig.6 Correlation coefficient</figcaption>
</figure>

#### Radar Image Comparison

1. Probability of Detection (POD)  
<p align="center">
 POD= \frac{a,a+b}
</p>  
Where a is the number of grid cells where rain occurred and they were successful predicted as rain cells by the nowcasting model; and b is the number of grid cells where rain occurred but they were not predicted by the nowcasting model. What is marked as ‘rain’ and what as ‘no rain’ is determined using a particular threshold Z. The threshold is used because the measured radar data can have clutter and noise which can produce spurious values at low intensities. The threshold is set at Z = 0.5 mm/hr. This means that whenever rainfall is measured above this threshold it will be marked as an ‘rain’; otherwise as ‘no rain’.

<table style="width:40%" align="center">
  <tr>
    <th>Measurement</th>
    <th>Forecast rain</th> 
    <th>Forecast no-rain</th>
  </tr>
  <tr>
    <td>Rain</td>
    <td>a</td> 
    <td>b</td>
  </tr>
  <tr>
    <td>no-rain</td>
    <td>c</td> 
    <td>d</td>
  </tr>
</table>

<figure style="width:60%" class="align-center">
<img src="{{ site.url }}/images/POD.png"><br>
<figcaption>Fig.7 POD for 30 mins lead time</figcaption>
</figure>


2. False Alarm Rate(FAR)  
<p align="center">
 FAR= \frac{c,a+c}
</p>   
where the a,c has the same meaning as before.  
This indicates the probability of producing false signal.

<figure style="width:60%" class="align-center">
<img src="{{ site.url }}/images/FAR.png">
<figcaption>Fig.7 FAR for 30 mins lead time</figcaption>
</figure>

3. Critical Success Index (CSI)  
<p align="center">
CSI=\frac{a, a+b+c}
</p>    
where the a,b,c has the same meaning as before.

<figure style="width:60%" class="align-center">
<img src="{{ site.url }}/images/CSI.png">
<figcaption>Fig.8 CSI for 30 mins lead time</figcaption>
</figure>