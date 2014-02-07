---
title: Animated ggplot
layout: post
type: post
---
Working with time series data can be challenging to comprehend. Some colleagues are going to be plotting model output across spatial regions. I suggested an animation might be helpful to see potential changes as the model progresses through time. ```animation ``` is a relatively simple and robust program that basically creates a stop-motion animation from slightly modified images. Here is a .gif of a sample:


Before running the following code, make sure that <a href="http://www.imagemagick.org" target="_blank">ImageMagick</a> is installed on your machine. 


```{r rCode}
rm(list = ls())
###############
# Script Info #
###############
# PURPOSE: Simple animation with ggplot2
# AUTHOR: Scott Large 2014
# REVIEWED BY:
# VERSION: 0.1
############
# PACKAGES #
#############
library(ggplot2)
library(animation)
library(gridExtra)
###############
# CREATE DATA #
###############
ts.length <- 50
ts.dat <- data.frame(x = seq(1, ts.length),
                     y = 1)

test <- function(ts.length) {
  df <- data.frame()
    for(i in 1:ts.length) {
      dat <- data.frame(x = sample(10, 20, replace = TRUE),
                       y = sample(10, 20, replace = TRUE))
      dat$t <- i
      df <- rbind(dat, df)
    } # close i loop
    return(df)
} # close test() function

tt <- test(ts.length)

gganimate <- function(j){
  m <- ggplot(tt[tt$t == j,], aes(xmin = x, xmax = x + 1, ymin = y, ymax = y + 1)) +
       geom_rect() +
       scale_x_continuous(limits = c(1,10)) +
       scale_y_continuous(limits = c(1,10)) +
       theme_bw()
  ts <- ggplot(ts.dat, aes(x = x, y = y)) +
               geom_line() +
               geom_vline(xintercept = j, colour = "RED") +
               labs(x = "TIME", title = paste0("Time stop ", j)) +
               theme_bw() +
               theme(axis.text.y = element_blank(),
                     axis.title.y = element_blank(),
                     axis.ticks.y = element_blank()) 
  grid.arrange(m, ts, nrow = 2, heights = c(3,1))
}               
gganimate(25)

# oopt <- animation::ani.options(interval = 0.1)
FUN2 <- function() {
  lapply(seq(1, ts.length), function(i) {
    return(gganimate(i))
    animation::ani.pause()
  })
}
```

```{r animated ggplot, fig.show='animate', interval=.2, fig.height=5}
FUN2()
```
```{r}
#saveHTML(FUN2(), 
#         autoplay = FALSE, 
#         loop = FALSE, 
#         verbose = FALSE, 
#         outdir = getwd(),
#         single.opts = "'controls': ['first', 'previous', 'play', 'next', 'last', 'loop', 'speed'], 'delayMin': 0")
```