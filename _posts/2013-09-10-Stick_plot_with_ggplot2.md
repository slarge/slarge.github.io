---
title: Stick plot of local wind forecast with ggplot2
layout: post
type: post
---


Earlier this week I came across a post on 
<a href="http://rud.is/b/2013/09/08/rforecastio-simple-r-package-to-access-forecast-io-weather-data/" target="_blank">Bob Rudis' blog</a> detailing a new package, ```Rforecastio```, which connects R to the <a href="http://forecast.io/" target="_blank">forecast.io</a> API. Beyond temperature, precipitation, and cloud cover, the forecasts include wind speed and direction. I have been riding my bike to/from work for the past few months and have experienced the joy of a strong southern wind coming home, and the frustration of a strong southern wind heading into work... I first saw a <a href="http://tabs.gerg.tamu.edu/Tglo/stick.html" target="_blank">stick plot</a> a few months ago in a <a href="http://www.sciencedirect.com/science/article/pii/S0278434313001398" target="_blank">paper</a>
 that I was reading. Basically, the length of the stick represents magnitude of the wind (or current) and the direction represents the cardinal direction (i.e., 0 degrees = north and 180 degrees = south). I've been enjoying ```ggplot2``` lately and wanted to see if I could make such a plot.

First, I went to <a href="http://forecast.io/" target="_blank">forecast.io</a> and registered my email address to access the developer API. It's a good idea to keep the API number private so I saved it as a .txt file and loaded it using ```readLines()```. The ```geocode()``` function from ```ggmap``` is really helpful to find lat/lon for physical addresses. Here, I used it to find the lat/lon for Woods Hole, MA. The main function for ```Rforcastio``` is ```fio.forecast()```, which takes the forcast.io API, latitude, and longitude, and creates a list with minute-by-minute, hourly, and daily forecasts for the next 48 hours. I need to read more about their methods, but they seem to compile from a lot of credible sources.

Stick plots are basically line segments for each hourly reading. The x- and y-coordinates for the beginning and end of each line segment must be specified to create stick plots, however, our data only provides the magnitude and direction for each hourly reading. Using some vector algebra, I transfered the wind speed (magnitude) and wind bearing (direction) into "u" and "v" components. The ```aspace``` package includes convenient functions that calculate sine and cosine for degree measurements (see, ```sin_d()``` and ```cos_d()```). For time-1, the x-component of each begins at 1 and ends at 1 plus the "v" component". Since I wanted to center the magnitude on zero, the y-component begins at zero and ends at "u". 

![plot of chunk allAction](/notebook/assets/allAction.png) 

A few things to note... other than the fact the wind will probably be blowing hard from the south and riding in will be slow going. I am not happy with the x-axis. I tried to simply use time as specified from the data pull, but things get complicated adding "v" to ISO specified time. That is, -3.2 + 2003-09-11 08:00:00 isn't really what I'm trying to accomplish. One of the "positives" with ```ggplot2``` is that it only plots data, and even adjusting the x- scale using ```scale_x_continuous()``` is really only supposed to be used for data transformations. In practice, I would probably want to include a plot of temperature or precipitation and would piggy back x-axes. However, since a southern wind is not actually negative it was rather easy to plot only the positive labels with ```scale_y_continuous()```.


Read below if you want to see how I put together the stick plot with ```ggplot2```


```r
library("devtools", quietly = T)
install_github("Rforecastio", "hrbrmstr")
require(Rforecastio, quietly = T)
require(ggplot2, quietly = T)
require(ggmap, quietly = T)
require(aspace, quietly = T)
require(grid, quietly = T)
```



```r
location <- geocode("Woods Hole, MA") # Lat/lon for physical address
fio.api.key <- readLines("~/forcastioAPI.txt") # keep API# private!!
my.longitude <- location[1]
my.latitude <- location[2]

# We only want the hourly data for this exercise... so no need to load full list
fio.list <- fio.forecast(fio.api.key, my.latitude, my.longitude)$hourly.df
wind <- with(fio.list, data.frame("day" = time, 
                                  "windSpeed" = windSpeed,
                                  "windBearing" = windBearing))

# I will need to look into why time is loaded as 2003... odd.
time <- seq(1, nrow(wind), 1)
wind$u <- wind$windSpeed * sin_d(wind$windBearing) #y component
wind$v <- wind$windSpeed * cos_d(wind$windBearing) #x component

SP <- ggplot(wind, aes(x = time, y = windSpeed)) +
            geom_abline(intercept = -10, slope = 0, colour = "grey70", linetype = "dotted") +
            geom_abline(intercept = 10, slope = 0, colour = "grey70", linetype = "dotted") +
            geom_abline(intercept = -20, slope = 0, colour = "red", linetype = "longdash") +
            geom_abline(intercept = 20, slope = 0, colour = "red", linetype = "longdash") +
            geom_abline(intercept = 0, slope = 0, colour = "grey80") +
            geom_segment(aes(x= 0, xend = 0, y = 15, yend = 22), 
                         arrow = arrow(length = unit(0.2, "cm"))) +
            geom_text(aes(x = 2, y = 17, label = "N")) + 
            geom_segment(aes(x = time,
                             xend = time + u,
                             y = 0,
                             yend = v)) +
            scale_y_continuous(breaks = c(0, 10, 20)) +
            xlab("Time") +
            ylab("Wind speed (mph)") + 
            ggtitle("48 hr forecasted wind speed for Woods Hole, MA") +
            theme_bw()
print(SP)
```


