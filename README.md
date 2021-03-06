Data Visualization Examples
================

-   [General Tips and Tricks](#general-tips-and-tricks)
-   [Setup for plotting in R](#setup-for-plotting-in-r)
-   [Plot Types](#plot-types)
    -   [Bar chart](#bar-chart)
    -   [Box plot](#box-plot)
    -   [Line chart](#line-chart)

If the repository does not contain the plot type you need, please
consider submitting a pull request to add it.

# General Tips and Tricks

1.  Prefer horizontal layout over vertical layouts. Differences in width
    are easier to perceive than differences in height.
2.  Show the variability in your results. Results are rarely perfectly
    consistent.
3.  [Label your axis](https://xkcd.com/833/).
4.  include the unit in the axis label (e.g., `[s]` for seconds).
5.  Avoid large numbers on axis labels (e.g., use `1 MB`, not
    `1000 KB`).
6.  Make sure your axes starts at 0 to fairly represent the differences
    in values.
7.  Use the right amount of grid lines. Choose to use major and/or minor
    grid lines depending on your plot. Too many grid lines clutters the
    plot. Too few grid lines makes the plot hard to read.

# Setup for plotting in R

The code snippets shown on this page require one or more of the
libraries imported below.

``` r
library(tidyverse)      # import ggplot, dplyr, readr, etc.
library(data.table)     # data analysis
library(zoo)            # for working with time series data

library(coronavirus)    # dataset for example plots

theme_set(theme_bw())   # set black and white theme for plots.
library(gghighlight)    # for highlighting in plots
library(cowplot)        # convenience functions for plotting

library(knitr)          # literate programming library

# saves plot and removes surrounding whitespace
saveplot <- function(filename, ...) {
  ggsave2(filename, ...)
  knitr::plot_crop(filename)
}
```

# Plot Types

## Bar chart

``` r
 # load data set
iris %>%
  # group by species for summary on next line
  group_by(Species) %>%
  # calculate the arithmetic mean and the 5th and 95th percentile
  summarize(
    length = mean(Sepal.Length),
    p5 = quantile(Sepal.Length,0.05),
    p95 = quantile(Sepal.Length, 0.95)) %>%
  # plot data. 'fill=Species' adds the colors
  ggplot(aes(y=length, x=Species, fill=Species)) +
  # create bar chart where data contains value to plot (mean, 5th percentile, 95th percentile)
  geom_col() +
  # add whiskers with 5th percentile and 95th percentile
  geom_errorbar(aes(ymin=p5, ymax=p95), width=.4) +
  # make sure plot starts at 0 cm
  ylim(0,NA) +
  # add pretty axis labels
  labs(x= "", y="sepal length [cm]") +
  # modify plot theme
  theme_half_open() +
  # add major grid lines on the horizontal axis
  background_grid(major = "x") +
  # remove legend
  theme(legend.position = "none") +
  # make bars horizontal, not vertical
  coord_flip()
```

![](README_files/figure-gfm/bar-chart-1.svg)<!-- -->

Bar charts are useful for comparing scalar values such as the makespan
of a workload when using different schedulers, or the latency of request
for different network configurations. However, when there is variability
in your results (i.e., rerunning your experiments results in different
values), consider using a [box plot](#box-plot).

We plot the sepal length for all flowers in our data. The vertical axis
shows the different flower types, and the horizontal axis shows the
average sepal length, as well as the 5th and 95th percentiles.

## Box plot

``` r
# load data set
iris %>%
  # plot data. 'fill=Species' adds the different colors
  ggplot(aes(y=Sepal.Length, x=Species, fill=Species)) +
  # create box plot
  geom_boxplot() +
  # calculate the arithmetic mean, for each species. Show in plot using a white circle with black border
  stat_summary(fun=mean, geom="point", shape=21, size=2, color="black", fill="white") +
  # make sure the horizontal axis starts at 0
  ylim(0,NA) +
  # add pretty labels
  labs(x= "", y="sepal length [cm]") +
  # set plot theme
  theme_half_open() +
  # add major grid lines for horizontal axis
  background_grid(major = "x") +
  # remove legend
  theme(legend.position = "none") +
  # make boxes horizontal, not vertical
  coord_flip()
```

![](README_files/figure-gfm/box-plot-1.svg)<!-- -->

Whenever your scalar results show variability, it is good practice to
consider using a box plot. The box plot is generally better at showing
variability than a bar chart with whiskers because a box plot shows more
(well known) statistical properties. The black line inside each box
shows the median value, while the left and right side of the box show
the 25th and 75th percentile respectively. Just as with the bar chart,
the meaning of the whiskers in a box plot can vary. Typically, the
whiskers extend to values that are within 1.5 times the *inter-quartile
range* (IQR), i.e., within 1.5 times the width of the box. Values
outside this range are outliers and are typically depicted individually
as dots (in black). Alternative uses of the whiskers include, but are
not limited to, the 5th and 95th percentile, and the minimum and maximum
value.

Although the box plot shows several useful statistical properties, it
does not show the commonly reported arithmetic mean (i.e., *average*) of
a distribution. Unfortunately, there is no standardized way of showing
the mean in a box plot. We find that a simple marker, distinctly
different from the type of marker used for outliers, typically works
well. In the example above we use a white dot with a black border to
indicate the mean.

## Line chart

Line charts are useful for depicting events over time. The examples
below show the number of COVID-19 cases reported over time in several
places across the globe.

### Simple line chart

``` r
coronavirus %>%
  # filter data
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  # create new column
  mutate(place = ifelse(country == "US", "US", "other")) %>%
  # order items in legend: first "US", then "other"
  mutate(place = factor(place, levels = c("US", "other"))) %>%
  # group and sum cases in US and elsewhere
  group_by(date, place) %>%
  summarise(cases = sum(cases), .groups='drop') %>%
  ggplot(aes(x=date, y=cases, color=place)) +
  geom_line() +
  theme_half_open() +
  scale_y_continuous(labels = function(x) floor(x/1000)) +
  ylab(bquote("\u00D7"*10^3~"number of cases")) +  background_grid(major = "y") +
  theme(legend.position = "bottom")
```

![](README_files/figure-gfm/line-chart-simple-1.svg)<!-- -->

A very simple line chart showing events over time.

### Plotting cumulative values

``` r
coronavirus %>%
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  mutate(place = ifelse(country == "US", "US", "other")) %>%
  # order items in legend: first "US", then "other"
  mutate(place = factor(place, levels = c("US", "other"))) %>%
  group_by(date, place) %>%
  summarise(cases = sum(cases), .groups='drop') %>%
  group_by(place) %>%
  # create column with cumulative value
  mutate(ccases = cumsum(cases)) %>%
  ggplot(aes(x=date, y=ccases, color=place)) +
  geom_line()+
  theme_half_open() +
  scale_y_continuous(labels = function(x) floor(x/1000000)) +
  ylab(bquote("\u00D7"*10^6~"cumulative number of cases")) +  background_grid(major = "y") +
  theme(legend.position = "bottom")
```

![](README_files/figure-gfm/line-chart-cumulative-1.svg)<!-- -->

A line chart can become difficult to read when the curves are highly
jittery. In that case, it may help to plot cumulative values. However,
this will make it more difficult to observe interesting events (e.g.,
spikes) in the data.

### Using a sliding window average

``` r
coronavirus %>%
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  mutate(place = ifelse(country == "US", "US", "other")) %>%
  # order items in legend: first "US", then "other"
  mutate(place = factor(place, levels = c("US", "other"))) %>%
  group_by(date, place) %>%
  summarise(cases = sum(cases), .groups="drop") %>%
  group_by(place) %>%
  # add column with sliding window average, window size = 7
  mutate(meancases = rollmean(cases, 7, fill=NA)) %>%
  filter(!is.na(meancases)) %>%
  ggplot(aes(x=date, y=meancases, color=place)) +
  geom_line()+
  theme_half_open() +
  scale_y_continuous(labels = function(x) floor(x/1000)) +
  ylab(bquote("\u00D7"*10^3~"number of cases")) +
  background_grid(major = "y") +
  theme(legend.position = "bottom")
```

![](README_files/figure-gfm/line-chart-sliding-window-1.svg)<!-- -->

Another way of dealing with jittery curves is to use a sliding window
average. This smooths the curves, but partially hides spikes. For
example, compare the spike just before (i.e., to the left of) 2021-01,
and compare it with the spike [without using a sliding window
average](#simple-line-chart).

### Highlighting values

``` r
labels <- c("US", "India")
coronavirus %>%
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  group_by(country) %>%
  # Add US and India to the start of country list, to get the correct legend colors
  mutate(country = factor(country, levels = c(labels, sort(setdiff(country, labels))))) %>%
  mutate(meancases = rollmean(cases, 7, fill = NA)) %>%
  filter(!is.na(meancases)) %>%
  ggplot(aes(x=date, y=meancases, color=country)) +
  geom_line() +
  # highlight countries whose rolling average number of cases exceeds 200,000
  # if necessary, connect label to curve with a black line segment
  gghighlight(max(meancases) > 200000, label_key=country, label_params = list(segment.color="black")) +
  theme_half_open() +
  scale_y_continuous(labels = function(x) floor(x/1000)) +
  ylab(bquote("\u00D7"*10^3~"number of cases")) +
  background_grid(major = "y")
```

![](README_files/figure-gfm/line-chart-highlight-1.svg)<!-- -->

If your line chart contains many curves, it can be useful to highlight
the most important ones. This allows you to show all data without
overloading the reader.

### Improving accessibility

``` r
# create data for line chart
d <- coronavirus %>%
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  mutate(place = ifelse(country == "US", "US", "other")) %>%
  # order items in legend: first "US", then "other"
  mutate(place = factor(place, levels = c("US", "other"))) %>%
  group_by(date, place) %>%
  summarise(cases = sum(cases), .groups = "drop") %>%
  group_by(place) %>%
  mutate(meancases = rollmean(cases, 7, fill=NA)) %>%
  filter(!is.na(meancases))

# create data for markers
dm <- d %>%
  # select dates to place markers
  mutate(markers = if_else(mday(date) == 1, meancases, NaN)) %>%
  filter(!is.na(markers))

# plot
ggplot(data=d, aes(x=date, y=meancases, color=place)) +
geom_line() +
# plot markers on top of geom_line
geom_point(data=dm, aes(y=markers, shape=place, color=place), size=3) +
gghighlight(label_params = list(segment.color="black"), label_key = place) +
theme_half_open() +
scale_y_continuous(labels = function(x) floor(x/1000)) +
ylab(bquote("\u00D7"*10^3~"number of cases")) +
background_grid(major = "y") +
theme(legend.position = "none")
```

![](README_files/figure-gfm/line-chart-markers-1.svg)<!-- -->

Line charts can be tricky to read when printed in gray scale or when
your reader has a color vision deficiency. A simple way to improve
readability is to add markers to your chart.
