---
title: ggmap of Vermont
layout: post
type: post
---

### Making a map of Vermont with ggmaps ###
==========================================
Not everybody has a destination wedding in the tropics. Since my wife-to-be has worked and traveled throughout the US and I have family and friends from all over, we decided on Vermont for our wedding. We didn't pick Vermont for any reason in particular, other than it is scenic and within the 5 hours from our house. To help people figure out where to visit and what to do, we wanted to create a map of local favorites. I also wanted to get more familiar with the ```ggmap``` package.

#### Loading the destinations ####
Our friends and family are generally entertained with three things: hiking, eating, and drinking, so we searched for these types of activities and entered them into a google spreadsheet. I used the package ```RCurl``` to download a shared google document spreadsheet. To share a google spreadsheet: File -> Publish to the web.... Next, click "start publishing", select ".csv" and copy the link in the box below. If you only want to select certain columns or rows, you can do that, too.


```r
# Loading the necessary packages, and downloading the location data #
rm(list = ls())
require(ggmap, quietly = T)
require(RCurl, quietly = T)
require(mapproj, quietly = T)
require(RColorBrewer, quietly = T)
require(knitr, quietly = T)

cert <- system.file("CurlSSL/cacert.pem", package = "RCurl")
dat <- getURL("https://docs.google.com/spreadsheet/pub?key=0Al2XpDDirUGRdGNtV0gxQi05MHNHcUJFTDlrdk9BQXc&single=true&gid=0&output=csv", 
    cainfo = cert)
ad <- read.csv(textConnection(dat))
#head(ad)
```
#### Find coordinates and calculate distances ####
The R package ```ggmap``` has quite a few neat components. With a physical address (i.e., Number Street, City, State), I can use the ```geocode()``` function to locate the coordinates (lat/long).

```r
addresses <- with(ad, paste(Address, City, "VT", sep = ", "))
lonlat <- geocode(location = addresses, messaging = TRUE)

ad$lon <- lonlat[, 1]  # Longitude
ad$lat <- lonlat[, 2]  # Latitude
#head(ad, 5)
```

I can also calculate distances and travel time between two points using the ```mapdist()``` function, which uses the google maps API. Since it provides the option to specify the mode of transportation (i.e., driving, biking, etc) I assume it to be pretty accurate. Although Vermont is relatively small, there are a lot of small roads and towns and travel tends to be slow. For some reason, we ended up with a few replicate listings, which I removed manually.


```r
from <- addresses[addresses != "195 Mountain Top Road, Chittenden, VT"]  # The lodge where people will be traveling from
to <- addresses[addresses == "195 Mountain Top Road, Chittenden, VT"]  # all of the other locations of interest
dists <- mapdist(from = from, to = to, mode = "driving", output = "simple")

nrow(dists) == nrow(ad)  # uh-oh... we have a few duplicate listings
dists <- dists[-c(22, 30), ]  # row 22 and 30 seem to be the culprits, so they are manually removed

ad <- ad[ad$Location != "Mountain Top Inn", ]
ad$distance <- dists$miles  # Miles
ad$time <- dists$hours  # Hours
```

#### Making the map ####
Woodstock, Vermont is a picturesque village with many interesting places to eat, shop, and play. To make the map easier to read, I created an overlay of this area. Quechee State Park is located nearby Woodstock, so I will use that as the center of the overlay map. The main map will be centered on the Lodge (i.e., the "to" object from the ```mapdist()``` function). 

```r
quechee <- with(ad[ad$Location == "Quechee State Park", ], paste(Address, City, 
    "VT", sep = ", "))
map.obj <- get_map(to, source = "google", maptype = "roadmap", color = "color", 
    scale = 4, zoom = 8)
over.obj <- get_map(quechee, source = "google", maptype = "roadmap", color = "color", scale = 2, zoom = 11)
```
#### The map ####
Below is the plotting routine for a ```ggmap```. It follows the same general format as ```ggplot2``` and is pretty logical... after you figure out what all the different ```geoms_XYZ``` and ```aes()``` mean. 

```r
colors <- brewer.pal(10, "Paired")            # Rcolorbrewer list of colors that are easy to differentiate between
ad$name <- as.character(seq(1:length(ad$Location))) # Create a new list of numbers corresponding to each location

VTmap <- ggmap(map.obj, extent = "panel") +   # Map centered on the Lodge 
  geom_point(aes(x=lon, y=lat, color = Type), # Points are added and color coded according to "Type"
             data = ad, size = 10, alpha = 0.5, position = "jitter") +
  scale_color_manual(values = colors) + 
  geom_text(data = ad, aes(x = lon, y = lat, label = name), # Numbers are added to the points for reference
            size = 5, vjust = 0.5, hjust = 0.5, colour='black') 
print(VTmap)
```

![plot of chunk unnamed-chunk-5](/notebook/assets/unnamed-chunk-5.png) 

Here is the map zoomed in on Quechee State Park. Both  ```geom_point()``` and ```geom_text()``` kick back errors, because many of the points are plotted outside the range of the zoomed map. Here is a sample of what it will look like before it becomes the inset.

```r
overlay <- ggmap(over.obj, extent = "device", legend = "none") + # Map centered on "Quechee State Park"
  geom_point(aes(x=lon, y=lat, color = Type),  # Points are added and color cooded according to "Type"
             data = ad, size = 10, alpha = 0.5, position = "jitter") +
  scale_color_manual(values = colors) +
  geom_text(data = ad, aes(x = lon, y = lat, label = name), # Numbers are added to the points for reference
            size = 5, vjust = 0.5, hjust = 0.5, colour='black') 
print(overlay)

```

![plot of chunk unnamed-chunk-6](/notebook/assets/unnamed-chunk-6.png) 

#### Map of Vermont with inset of Woodstock area ####

```r
fullMap <- VTmap + inset(grob = ggplotGrob(overlay + theme_inset()), xmin = -74.75, 
    xmax = -73.25, ymin = 42.5, ymax = 43.5)
print(fullMap)
```
![plot of chunk unnamed-chunk-7](/notebook/assets/unnamed-chunk-7.png) 

### Conclusions ###
Overall, I'm pretty happy with how the map turned out. I like how easy it was to layer different components with ```ggmap```. I was also plesantly surprised with how simple it was to insert the overlay. The ```geocode()``` and ```mapdist()``` functions worked quite well. Distance could easily have been incorporated by changing the color or size, accordingly: 

```r
DISTmap <- ggmap(map.obj, extent = "panel") +   # Map centered on the Lodge 
  geom_point(aes(x=lon, y=lat, color = distance), data = ad, size = 10, alpha = 0.5) +
    scale_colour_gradient(low = "green", high = "blue") 
print(DISTmap)
##
DISTmap2 <- ggmap(map.obj, extent = "panel") +   # Map centered on the Lodge 
  geom_point(aes(x=lon, y=lat, size = distance), data = ad, alpha = 0.5, color = "black")
print(DISTmap2)
```

![plot of chunk unnamed-chunk-8](/notebook/assets/unnamed-chunk-81.png) ![plot of chunk unnamed-chunk-8](/notebook/assets/unnamed-chunk-82.png) 


However, one thing really frustrated me... when points are close together ```position = 'jitter'``` and 'dodge' can be used to move them to avoid overplotting. Adjusting the position in ```geom_point``` and ```geom_text```, however, don't seem to correspond and many of the points are difficult to distinguish. For the final version, I used photoshop to manually label the markers and to add a list of locations and numbers.


