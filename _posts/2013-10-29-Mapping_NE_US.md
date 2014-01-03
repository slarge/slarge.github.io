---
title: Coastal map of NE US
layout: post
type: post
---

I've spent the past few weeks working hard to finalize a couple of manuscripts. After a few short years, I'm excited to finally be sending out the last chapter of my dissertation for review. The chapter quantifies genetic diversity in <i> Nucella lapillus </i> across a few portions of the northeast US. If you are interested in an interesting natural history, I encourage you to check out <a href="http://www.jstor.org/stable/2845711" target="_blank">Ingolfsson 1992</a> and <a href= "http://webpages.icav.up.pt/fabulousfabalis/1%20Ecology/Waltari%20and%20Hickerson%20JBiog%202012.pdf" target="_blank">Waltari and Hickerson 2013 </a>. For snails that have limited mobility, it seems that they are able to disperse beyond what we might expect. To complete my manuscript, I needed to provide a map of my sample sites. A friend helped me make one using ArcGIS for my dissertation, but some site names were not consistent and in general, it needed some work. I decided to re-do the whole thing in ```R``` using ```ggplot2``` and ```PBSmapping```. 

Here's the map. A few final touches and it should be set for publication! 

![plot of chunk sampleMap_v_002.png](/assets/sampleMap_v_002.png) 

See below for the code. It's quite long for a simple map because there were a lot of nuances that I needed to manually adjust.


```{r}
require(maptools)
require(mapdata)
require(maps)
require(sp)
require(ggplot2)
require(geosphere)
require(PBSmapping)
require(gridExtra)
```

```{r}
## Create some folders to hold the data
zipData<-"/Users/Scott/Desktop/zipData"
mapData<-"/Users/Scott/Desktop/mapData/"
figuredir="/Users/Scott/Desktop/figures/"
setwd(mapData)

# Fantastic coastal maps from NOAA Geophysical Data Center

# http://www.ngdc.noaa.gov/mgg/shorelines/data/gshhg/latest/
# url_data = "http://www.ngdc.noaa.gov/mgg/shorelines/data/gshhg/latest/gshhg-bin-2.2.2.zip" 
# zip_file = sprintf("%s/gshhg-bin-2.2.2.zip", zipData) 
# download.file(url= url_data, destfile= zip_file)
# unzip(zip_file, exdir = mapData) #Download according to the above filestructure

# The PBSmapping package has some nice functions to deal with the GHHS data
# rawMap <- importGSHHS(paste0(mapData, "/gshhs_f.b"), 
#                      xlim = c(360-75,360-62), 
#                      ylim = c(40,48), maxLevel = 1, n = 0)
# NEUSdata <- thinPolys(rawMap, tol=0.1, filter = 3)
# save(NEUSdata, file = "sampleMap.Rdata")
```

```{r}
load("sampleMap.RDATA")

# convert the GSHHS data into a ggplot2 readable data frame
NEUS <- fortify(NEUSdata)

loc<-read.csv("sampleMap_v001.csv", header=T, sep=",", na.strings=c("",".","NULL"," ", "NA"))
north <- data.frame("y" = loc$Latitude[loc$Location == "North"],
                    "x" = loc$Longitude[loc$Location == "North"])
south <- data.frame("y" = loc$Latitude[loc$Location == "South"],
                    "x" = loc$Longitude[loc$Location == "South"])

names(loc)[4] <- "x" 
names(loc)[3] <- "y"

# I needed to create a scale bar. The destPoint() function is helpful for creating a starting
# and end point for a certain distance

scales = c(0, 100000, 200000)
scaleBar <- data.frame("DISTANCE" = scales/1000,
                       "b.lon" = -68,
                       "b.lat" = 40.75,
                       destPoint(p = c(-68, 40.75), b = 90, d = scales))
scaleLength <- scaleBar[3,]

scaleSmall <- c(0, 5000, 10000)
scaleBarS <- data.frame("DISTANCE" = scales/1000,
                        "b.lon" = -69.5,
                        "b.lat" = 43.7,
                         destPoint(p = c(-69.5, 43.7), b = 90, d = scaleSmall))

scaleBarN <- data.frame("DISTANCE" = scales/1000,
                        "b.lon" = -67.0,
                        "b.lat" = 44.65,
                        destPoint(p = c(-67.0, 44.65), b = 90, d = scaleSmall))
scaleLengthS <- scaleBarS[3,]
scaleLengthN <- scaleBarN[3,]

# The centroid() function finds the center of a group of points. I add a few degrees onto the centroid
# in each direction to create a rectangle around my study sites

northoid <- data.frame("xmin" = centroid(cbind(north[,2], north[,1]))[1] - .35,
                       "xmax" = centroid(cbind(north[,2], north[,1]))[1] + .35,
                       "ymin" = centroid(cbind(north[,2], north[,1]))[2] - .2, 
                       "ymax" = centroid(cbind(north[,2], north[,1]))[2] + .2)

southoid <- data.frame("xmin" = centroid(cbind(south[,2], south[,1]))[1] - .35,
                       "xmax" = centroid(cbind(south[,2], south[,1]))[1] + .35,
                       "ymin" = centroid(cbind(south[,2], south[,1]))[2] - .2, 
                       "ymax" = centroid(cbind(south[,2], south[,1]))[2] + .2)
cents <- rbind(northoid, southoid)

# Making .png files and ggplot2 cooperate with reasonable text sizes can be challenging
# I am still not convinced this is the best answer

textSize <- rel(3)

# Every good map needs an arrow pointing north!
nArrow <- data.frame("b.lon" = scaleBar[3,4],
                     "b.lat" = 41.0,
                     destPoint(p = c(scaleBar[3,4], 41.0), b = 0, d = 50000))

# The large map of the whole study system, with rectangles drawn around the smaller areas
# I used the annotate() and geom_segment functions to  make a scale bar
g <- ggplot()
g <- g + geom_polygon(data = NEUS, aes(x = X,
                                       y = Y, group = PID), fill = "gray75") +        
         geom_rect(data = cents, 
                   aes(xmin = cents$xmin,
                       xmax = cents$xmax,
                       ymin = cents$ymin, 
                       ymax = cents$ymax),
                   fill = "transparent", color = "black", size = 1) +
         geom_segment(data = scaleLength, 
                      aes(x = b.lon, 
                          y = b.lat, x
                          end = lon, 
                          yend = b.lat), color = "BLACK") +  
         geom_segment(data = scaleBar, 
                      aes(x = lon, 
                      y = b.lat, 
                      xend = lon, 
                      yend = b.lat + 0.05), color = "BLACK") + 
        annotate("text", label = c("0", "100", "200"), x = scaleBar[,4], y = scaleBar[,3] + 0.15, 
                  color = "BLACK", size = textSize) + 
        annotate("text", label = "km", x = scaleBar[3,4] + .2, y = scaleBar[1,3], 
                  color = "BLACK", size = textSize) + 
        geom_segment(data = nArrow, 
                     aes(x = b.lon, 
                     y = b.lat, 
                     xend = lon, 
                     yend = lat), arrow = arrow(length = unit(0.5, "cm"))) +
        annotate("text", label = "N", x = nArrow$b.lon + .1, y = nArrow$b.lat + .1, size = textSize) + 

  coord_cartesian(xlim = c(-73.50,-65), ylim = c(40.5, 45.25)) +
  theme_bw(base_size=12*(81/169)) + labs(x = "Longitude", y = "Latitude") +
  theme(text = element_text(size = textSize))

# Northern sites. Since the labels are so close together, I had to manually annotate them. Also, I used
# a smaller scale for the scale bar

n <- ggplot()
n <- n + geom_polygon(data = NEUS, aes(x = X,
                                       y = Y, group = PID), fill = "gray75") +         
     geom_point(data = loc, aes(x = x,
                                y = y,
                                color = Location, 
                                shape = Habitat), size = 3, color = "BLACK") +
     annotate("text", label = "QU", x = loc$x[loc$Abbrev == "QU"] + .05, 
                                    y = loc$y[loc$Abbrev == "QU"], 
              color = "BLACK", size = textSize) + 
     annotate("text", label = "CP", x = loc$x[loc$Abbrev == "CP"] - .05, 
                                    y = loc$y[loc$Abbrev == "CP"],
              color = "BLACK", size = textSize) + 
     annotate("text", label = "JP", x = loc$x[loc$Abbrev == "JP"] + .03, 
                                    y = loc$y[loc$Abbrev == "JP"] + .015, 
              color = "BLACK", size = textSize) + 
     annotate("text", label = "FDR", x = loc$x[loc$Abbrev == "FDR"] - .05, 
                                     y = loc$y[loc$Abbrev == "FDR"] - .015, 
              color = "BLACK", size = textSize) + 
     geom_segment(data = scaleLengthN, 
                  aes(x = b.lon, 
                      y = b.lat, 
                      xend = lon, 
                      yend = b.lat), color = "BLACK") +  
     geom_segment(data = scaleBarN, 
                  aes(x = lon, 
                      y = b.lat, 
                      xend = lon, 
                      yend = b.lat + 0.01), color = "BLACK") + 
     annotate("text", label = c("0", "5", "10"), x = scaleBarN[,4], y = scaleBarN[,3] + 0.03,
              color = "BLACK", size = textSize) + 
     annotate("text", label = "km", x = scaleBarN[3,4] + .05, y = scaleBarN[1,3],
              color = "BLACK", size = textSize) + 
     coord_cartesian(xlim = c(northoid$xmin, northoid$xmax), ylim = c(northoid$ymin, northoid$ymax)) +
  theme_bw(base_size=12*(81/169)) + labs(x = "Longitude", y = "Latitude") + 
  theme(legend.position = "none", text = element_text(size = textSize))

# Southern sites.
s <- ggplot()
s <- s + geom_polygon(data = NEUS, aes(x = X,
                                       y = Y, group = PID), fill = "gray75") +         
    geom_point(data = loc, aes(x = x,
                               y = y,
                               color = Location, 
                               shape = Habitat), size = 3, color = "BLACK") +
    annotate("text", label = "LNE", x = loc$x[loc$Abbrev == "LNE"] + .03, 
                                    y = loc$y[loc$Abbrev == "LNE"] + .03, 
             color = "BLACK", size = textSize) + 
    annotate("text", label = "LNW", x = loc$x[loc$Abbrev == "LNW"] - .05, 
                                    y = loc$y[loc$Abbrev == "LNW"] - .03, 
             color = "BLACK", size = textSize) + 
    annotate("text", label = "LPC", x = loc$x[loc$Abbrev == "LPC"],
                                    y = loc$y[loc$Abbrev == "LPC"] + .03, 
             color = "BLACK", size = textSize) + 
    annotate("text", label = "PP", x = loc$x[loc$Abbrev == "PP"], 
                                   y = loc$y[loc$Abbrev == "PP"] - .03, 
             color = "BLACK", size = textSize) + 
  
    geom_segment(data = scaleLengthS, 
                 aes(x = b.lon, 
                     y = b.lat, 
                     xend = lon, 
                     yend = b.lat), color = "BLACK") +  
    geom_segment(data = scaleBarS, 
                 aes(x = lon, 
                 y = b.lat, 
                 xend = lon, 
                 yend = b.lat + 0.01), color = "BLACK") + 
    annotate("text", label = c("0", "5", "10"), x = scaleBarS[,4], y = scaleBarS[,3] + 0.03, 
             color = "BLACK", size = textSize) + 
    annotate("text", label = "km", x = scaleBarS[3,4] + .05, y = scaleBarS[1,3], 
             color = "BLACK", size = textSize) + 
    coord_cartesian(xlim = c(southoid$xmin, southoid$xmax), ylim = c(southoid$ymin, southoid$ymax)) +
    theme_bw(base_size=12*(81/169)) + labs(x = "Longitude", y = "Latitude") + 
    theme(legend.position = "none", text = element_text(size = textSize))

# Now to add 'g', 'n', and 's' into an appropriate layout
# The journal I am submitting to requires figures width to be less than 17 cm

map.plot <- paste(mapData, "sampleMap_", "v_002.png", sep="")
png(file = map.plot, width= 170, height = 170, units = "mm", res = 600)
# pdf("figure.pdf")
grid.arrange(g, arrangeGrob(s, n, ncol = 2, nrow = 1), 
             nrow = 2, heights = c(5,3))
dev.off()

```



