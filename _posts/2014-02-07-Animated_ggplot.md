---
title: Animated ggplot
layout: post
type: post
---
Working with time series data can be challenging to comprehend. Some colleagues are going to be plotting model output across spatial regions. I suggested an animation might be helpful to see potential changes as the model progresses through time. ```animation ``` is a relatively simple and robust program that basically creates a stop-motion animation from slightly modified images. Here is a .gif of a sample:

<img src="/assets/2014-02-07-animated_ggplot.gif" style="width: 400px; height: 400px; border:1px solid black; padding:0px;"/>

Before running the following code, make sure that <a href="http://www.imagemagick.org" target="_blank">ImageMagick</a> and <a href="http://www.ffmpeg.org/" target="_blank">FFmpeg</a> are installed on your machine. I used some help from <a href="http://www.renevolution.com/how-to-install-ffmpeg-on-mac-os-x/" target="_blank">Renevolution</a> for a bit of guidance on my mac.


{% highlight r %}
############### Script Info #
############### 
############### PURPOSE: Simple animation with ggplot2 AUTHOR: Scott Large
############### 2014 REVIEWED BY: VERSION: 0.1
############### 
############### PACKAGES #
library(ggplot2)
library(animation)
library(gridExtra)
############### CREATE DATA #
ts.length <- 50
ts.dat <- data.frame(x = seq(1, ts.length), y = 1)

test <- function(ts.length) {
    df <- data.frame()
    for (i in 1:ts.length) {
        dat <- data.frame(x = sample(10, 20, replace = TRUE), y = sample(10, 
            20, replace = TRUE))
        dat$t <- i
        df <- rbind(dat, df)
    }  # close i loop
    return(df)
}  # close test() function

tt <- test(ts.length)

gganimate <- function(j) {
    m <- ggplot(tt[tt$t == j, ], aes(xmin = x, xmax = x + 1, ymin = y, ymax = y + 
        1)) + geom_rect() + scale_x_continuous(limits = c(1, 10)) + scale_y_continuous(limits = c(1, 
        10)) + theme_bw()
    ts <- ggplot(ts.dat, aes(x = x, y = y)) + geom_line() + geom_vline(xintercept = j, 
        colour = "RED") + labs(x = "TIME", title = paste0("Time stop ", j)) + 
        theme_bw() + theme(axis.text.y = element_blank(), axis.title.y = element_blank(), 
        axis.ticks.y = element_blank())
    gg <- arrangeGrob(m, ts, nrow = 2, heights = c(3, 1))
    ggsave(paste0("ggAnimatePlot_", sprintf("%03d", j), ".png"), gg, height = 6, 
        width = 6, unit = "cm", scale = 2)
}

# oopt <- animation::ani.options(interval = 0.1)
FUN2 <- function() {
    lapply(seq(1, ts.length), function(i) {
        # return(gganimate(i))
        gganimate(i)
        animation::ani.pause()
    })
}

# Create the .png files
FUN2()
# Convert into a .gif
system("convert -delay 50 *.png ~/git/slarge.github.io/assets/2014-02-07-animated_ggplot.gif")
# remove the .png files
file.remove(list.files(pattern = ".png"))

# saveHTML(FUN2(), autoplay = FALSE, loop = FALSE, verbose = FALSE, outdir
# = getwd(), single.opts = ''controls': ['first', 'previous', 'play',
# 'next', 'last', 'loop', 'speed'], 'delayMin': 0')
{% endhighlight %}

