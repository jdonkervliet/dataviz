---
title: "A Practical Guide to Visualizing Your Results"
output:
  html_notebook:
    toc: true
    toc_depth: 1
  github_document:
    toc: true
    toc_depth: 1
    dev: svg
---

# Overview

This repository contains data visualization scripts for your HP/BSc/MSc/PhD Thesis + articles!
If the repository does not contain a script that suits your needs, please consider submitting a pull request to add it.

```{r setup, message = FALSE}

library(tidyverse)
theme_set(theme_bw())
library(knitr)
library(data.table)
library(cowplot)
library(coronavirus)
library(gghighlight)
library(zoo)

saveplot <- function(filename, ...) {
  ggsave2(filename, ...)
  knitr::plot_crop(filename)
}
```
R comes with built-in datasets, such as the `iris` dataset. You can list the available datasets using `data()`.



# Bar chart

This section walks you through creating a bar chart. You can find the final result at the [end of this section](#showing-variability-in-a-bar-chart).

Bar charts are useful for comparing scalar values such as the makespan of a workload when using different schedulers,
or the latency of request for different network configurations.
However, when there is variability in your results (i.e., rerunning your experiments results in different values), consider using a [box plot](#box-plot).

In our example we will work with the `iris` dataset included in R.
We plot the arithmetic mean (i.e., average) sepal length for all flowers in the dataset.
The resulting plot is simple but clear.
On the vertical axis we print a label (length) and the unit (centimeters),
we print major grid lines for improved readability, but leave out the minor grid lines.
Adding minor grid lines would be overkill: we are not comparing anything that requires fine-grained comparison.
Lastly, we start the vertical axis at zero.
The importance of doing so will become apparent in our next plot.


```{r}
iris %>%
  select(Sepal.Length) %>%
  summarize(value = mean(Sepal.Length)) %>%
  ggplot(aes(x="sepal", y=value, fill=value)) +
  geom_col() +
  ylim(0,NA) +
  labs(x= "", y="length [cm]") +
  theme_half_open() +
  background_grid(major = "y") +
  theme(legend.position = "none")
saveplot("bar1.pdf", height = 3, width = 1)
```

The next step is to depict multiple bars, allowing us to compare values.
In our example these values are lengths, but in your case these values are more likely to be duration, cost, energy, etc.
The plot shows that there is an observable, although not extremely large, difference between the mean sepal lengths of three types of iris species.

To allow a reader to get a good understanding of the magnitude of the differences between values quickly and easily, it is important to start the vertical axis at zero,
even if the "interesting part" is between 4cm and 7cm.
Starting the vertical axis at zero ensures that the ratio between the height of the bars is the same as the ratio between the depicted values.
For example, starting the vertical axis at 4cm would make the blue bar twice as high as the red bar, incorrectly suggesting that the mean sepal length of virginica is twice that of setosa.
Because your favorite plotting framework may not automatically force the vertical axis at zero,
it is good practice to include such a command explicitly (e.g., `ylim(0,NA)`).


```{r}
iris %>%
  group_by(Species) %>%
  summarize(Sepal.Length = mean(Sepal.Length)) %>%
  ggplot(aes(y=Sepal.Length, x=Species, fill=Species)) +
  geom_col() +
  ylim(0,NA) +
  labs(x= "", y="sepal length [cm]") +
  theme_half_open() +
  background_grid(major = "y") +
  theme(legend.position = "none") +
  theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust=1))
saveplot("bar1-compare.pdf", height = 3, width = 1.2)

iris %>%
  group_by(Species) %>%
  summarize(Sepal.Length = mean(Sepal.Length)) %>%
  ggplot(aes(y=Sepal.Length, x=Species, fill=Species)) +
  geom_col() +
  coord_cartesian(ylim = c(4, NA)) +
  labs(x= "", y="sepal length [cm]") +
  theme_half_open() +
  background_grid(major = "y") +
  theme(legend.position = "none") +
  theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust=1))
saveplot("bar1-compare-misleading.pdf", height = 3, width = 1.2)
```

```{r}
iris %>%
  select(Sepal.Length) %>%
  summarize(value = mean(Sepal.Length)) %>%
  ggplot(aes(x="sepal", y=value, fill=value)) +
  geom_col() +
  ylim(0,NA) +
  labs(x= "", y="length [cm]") +
  theme_half_open() +
  background_grid(major = "x") +
  theme(legend.position = "none") +
  coord_flip()
saveplot("bar2.pdf", height = 1, width = 3)
saveplot("bar2.png", height = 1, width = 3, dpi = 100)
```


```{r}
iris %>%
  group_by(Species) %>%
  summarize(Sepal.Length = mean(Sepal.Length)) %>%
  ggplot(aes(y=Sepal.Length, x=Species, fill=Species)) +
  geom_col() +
  ylim(0,NA) +
  labs(x= "", y="sepal length [cm]") +
  theme_half_open() +
  background_grid(major = "x") +
  theme(legend.position = "none") +
  coord_flip()
saveplot("bar3.pdf", height = 1.2, width = 3)
```

## Showing variability in a bar chart

If there is any variability in your results, it is typically recommended to use a box plot.
However, you can also show variability in your bar chart.
To do so, first choose which statistical property to visualize with the top (i.e., right-most edge) of the bar, such as the arithmetic mean or the median.
Then indicate the range of the variability using whiskers that extend up to a certain percentile of the distribution.
For example, starting from the 5th percentile up to the 95th percentile.
An example of this approach is shown in the figure below.

```{r}
iris %>%
  group_by(Species) %>%
  summarize(length = mean(Sepal.Length), p5 = quantile(Sepal.Length, 0.05), p95 = quantile(Sepal.Length, 0.95)) %>%
  ggplot(aes(y=length, x=Species, fill=Species)) +
  geom_col() +
  geom_errorbar(aes(ymin=p5, ymax=p95), width=.4) +
  ylim(0,NA) +
  labs(x= "", y="sepal length [cm]") +
  theme_half_open() +
  background_grid(major = "x") +
  theme(legend.position = "none") +
  coord_flip()
saveplot("bar4.pdf", height = 1.2, width = 3)
```

# Box plot

Whenever your scalar results show variability it is good practice to consider using a box plot.
The box plot is generally better at showing variance than a bar chart with whiskers because a box plot shows more (well known) statistical properties.
The line inside the box shows the median value, while the left and right side of the box show the 25th and 75th percentile respectively.
Just as with the bar chart, the meaning of the whiskers in a box plot can vary. Typically the whiskers extend up to 1.5 times the _inter-quartile range_ (IQR), i.e., 1.5 times the width of the box.
Alternative uses of the whiskers include, but are not limited to, the 5th and 95th percentile, and the minimum and maximum value.
Values outside this range are outliers and are typically depicted individually as dots.

```{r}
iris %>%
  group_by(Species) %>%
  ggplot(aes(y=Sepal.Length, x=Species, fill=Species)) +
  geom_boxplot() +
  ylim(0,NA) +
  labs(x= "", y="sepal length [cm]") +
  theme_half_open() +
  background_grid(major = "x") +
  theme(legend.position = "none") +
  coord_flip()
saveplot("box1.pdf", height = 1.2, width = 3)
```
## Adding the arithmetic mean to a box plot

Although the box plot shows several useful statistical properties, it does not show the commonly reported arithmetic mean (i.e., _average_) of a distribution.
Unfortunately, there is no standardized way of showing the mean in a box plot.
We find that a simple marker, distinctly different from the type of marker used for outliers, typically works well.
In the example below we use a white dot with a black border to indicate the mean.

```{r}
iris %>%
  group_by(Species) %>%
  ggplot(aes(y=Sepal.Length, x=Species, fill=Species)) +
  geom_boxplot() +
  stat_summary(fun.y=mean, geom="point", shape=21, size=1, color="black", fill="white") +
  ylim(0,NA) +
  labs(x= "", y="sepal length [cm]") +
  theme_half_open() +
  background_grid(major = "x") +
  theme(legend.position = "none") +
  coord_flip()
saveplot("box2.pdf", height = 1.2, width = 3)
```

```{r}
iris %>%
  group_by(Species) %>%
  ggplot(aes(y=Sepal.Length, x=Species, fill=Species)) +
  geom_violin() +
  stat_summary(fun.y=median, geom="point", shape=4, size=2, color="black") +
  stat_summary(fun.y=mean, geom="point", shape=21, size=1, color="black", fill="white") +
  ylim(0,NA) +
  labs(x= "", y="sepal length [cm]") +
  theme_half_open() +
  background_grid(major = "x") +
  theme(legend.position = "none") +
  coord_flip()
saveplot("violin1.pdf", height = 1.2, width = 3)
```
```{r}
iris %>%
  gather(key = "attribute", value = "measurement", -Species) %>%
  ggplot(aes(y=measurement, x=Species, fill=attribute)) +
  geom_boxplot() +
  stat_summary(fun.y=mean, geom="point", aes(group=attribute), position=position_dodge(0.75), shape=21, size=1.5, color="black", fill="white") +
  ylim(0,NA) +
  labs(x= "", y="sepal length [cm]") +
  scale_fill_discrete(labels=c("Sepal.Width" = "sepal width", "Sepal.Length" = "sepal length", "Petal.Width" = "petal width", "Petal.Length" = "petal length")) +
  theme_half_open() +
  background_grid(major = "x") +
  theme(legend.position = "bottom") +
  coord_flip()
saveplot("box-group1.pdf", height = 4, width = 6)
```
```{r}
iris %>%
  gather(key = "attribute", value = "measurement", -Species) %>%
  ggplot(aes(y=measurement, x=attribute, fill=Species)) +
  geom_boxplot() +
  stat_summary(fun.y=mean, geom="point", aes(group=Species), position=position_dodge(0.75), shape=21, size=1.5, color="black", fill="white") +
  ylim(0,NA) +
  labs(x="", y="sepal length [cm]") +
  scale_x_discrete(labels=c("Sepal.Width" = "sepal width", "Sepal.Length" = "sepal length", "Petal.Width" = "petal width", "Petal.Length" = "petal length")) +
  theme_half_open() +
  background_grid(major = "x") +
  theme(legend.position = "bottom") +
  coord_flip()
saveplot("box-group2.pdf", height = 4, width = 6)
```

```{r}
iris %>%
  gather(key = "attribute", value = "measurement", -Species) %>%
  ggplot(aes(y=measurement, x=Species, fill=Species)) +
  geom_boxplot() +
  facet_wrap(~attribute, ncol=1, labeller = labeller(attribute = c("Sepal.Width" = "sepal width", "Sepal.Length" = "sepal length", "Petal.Width" = "petal width", "Petal.Length" = "petal length"))) +
  stat_summary(fun.y=mean, geom="point", aes(group=Species), position=position_dodge(0.75), shape=21, size=2, color="black", fill="white") +
  ylim(0,NA) +
  labs(x= "", y="length [cm]") +
  theme_half_open() +
  background_grid(major = "x") +
  theme(legend.position = "none") +
  theme(strip.background=element_rect(fill='white', color="black")) +
  coord_flip()
saveplot("box-group3.pdf", height = 4, width = 6)
```

```{r}
iris %>%
  ggplot(aes(x=Sepal.Length, fill="")) +
  geom_histogram(binwidth = 0.1) +
  labs(x= "sepal length [cm]", y="count") +
  theme_half_open() +
  xlim(0, NA) +
  background_grid(major = "xy") +
  theme(legend.position = "none")
saveplot("histogram1.pdf", height = 2, width = 3)

iris %>%
  ggplot(aes(x=Sepal.Length, fill="")) +
  geom_density() +
  labs(x= "sepal length [cm]", y="") +
  theme_half_open() +
  xlim(0, NA) +
  background_grid(major = "xy") +
  theme(legend.position = "none")
saveplot("pdf1.pdf", height = 2, width = 3)
```

```{r}
iris %>%
  mutate(a = 1) %>%
  ggplot(aes(x=Sepal.Length, fill=a)) +
  stat_ecdf(geom = "step") +
  labs(x= "length [cm]", y="fraction") +
  theme_half_open() +
  xlim(0, NA) +
  background_grid(major = "xy") +
  theme(legend.position = "none")
saveplot("cdf1.pdf", height = 2, width = 3)
```

```{r}
df <- data.frame(value = rnorm(10000, 6, 1))

df %>%
  ggplot(aes(x=value)) +
  geom_histogram(binwidth = .2, color="black", fill = "white") +
  theme_half_open() +
  xlim(0, NA) +
  background_grid(major = "y") +
  theme(legend.position = "none")
saveplot("pdf2.pdf", height = 2, width = 3)


df %>%
  ggplot(aes(x=value)) +
  stat_ecdf(geom = "step") +
  theme_half_open() +
  xlim(0, NA) +
  background_grid(major = "xy", minor= "xy") +
  theme(legend.position = "none")
saveplot("cdf2.pdf", height = 2, width = 3)
```

```{r}
iris %>%
  ggplot(aes(y=Sepal.Width, x=Sepal.Length)) +
  geom_point() +
  ylim(0, NA) +
  xlim(0, NA) +
  labs(y="sepal width [cm]", x="sepal length [cm]") +
  theme_half_open() +
  background_grid(major = "xy") +
  theme(legend.position = "none") +
  coord_flip()
saveplot("scatter1.pdf", height = 4, width = 6)
```



```{r}
iris %>%
  group_by(Species) %>%
  ggplot(aes(y=Sepal.Width, x=Sepal.Length, color=Species)) +
  geom_point() +
  ylim(0, NA) +
  xlim(0, NA) +
  labs(y="sepal width [cm]", x="sepal length [cm]") +
  theme_half_open() +
  background_grid(major = "xy") +
  theme(legend.position = "bottom") +
  coord_flip()
saveplot("scatter2.pdf", height = 4, width = 6)
```
```{r}
iris %>%
  group_by(Species) %>%
  ggplot(aes(y=Sepal.Width, x=Sepal.Length, color=Species, shape=Species)) +
  geom_point(size = 3) +
  ylim(0, NA) +
  xlim(0, NA) +
  labs(y="sepal width [cm]", x="sepal length [cm]") +
  theme_half_open() +
  background_grid(major = "xy") +
  theme(legend.position = "bottom") +
  coord_flip()
saveplot("scatter3.pdf", height = 4, width = 6)
```

```{r}
coronavirus %>%
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  filter(month(date) == 12) %>%
  ggplot(aes(x=date, y=cases, fill=country)) +
  geom_col() +
  theme_half_open() +
  scale_y_continuous(labels = function(x) floor(x/100000)) +
  ylab(bquote("\u00D7"*10^5~"number of cases")) +
  background_grid(major = "y") +
  theme(legend.position = "none")
saveplot("stacked-bar1.pdf", height = 3, width = 4.5)

coronavirus %>%
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  filter(month(date) == 12) %>%
  ggplot(aes(x=date, y=cases, fill=country)) +
  geom_col() +
  gghighlight(max(cases) > 50000) +
  theme_half_open() +
  scale_y_continuous(labels = function(x) floor(x/100000)) +
  ylab(bquote("\u00D7"*10^5~"number of cases")) +
  background_grid(major = "y") +
  theme(legend.position = "bottom")
saveplot("stacked-bar1-a.pdf", height = 4, width = 6)
```


```{r}
coronavirus %>%
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  filter(month(date) == 12) %>%
  mutate(place = ifelse(country == "US", "US", "other")) %>%
  group_by(date, place) %>%
  summarise(cases = sum(cases)) %>%
  ggplot(aes(x=date, y=cases, fill=place)) +
  geom_col() +
  theme_half_open() +
  scale_y_continuous(labels = function(x) floor(x/100000)) +
  ylab(bquote("\u00D7"*10^5~"number of cases")) +
  background_grid(major = "y") +
  theme(legend.position = "bottom")
saveplot("stacked-bar2.pdf", height = 3, width = 4.5)
```


```{r}
coronavirus %>%
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  filter(month(date) == 12) %>%
  mutate(place = ifelse(country == "US", "US", "other")) %>%
  group_by(date, place) %>%
  summarise(cases = sum(cases)) %>%
  ggplot(aes(x=date, y=cases, fill=place)) +
  geom_col(position="fill") +
  theme_half_open() +
  ylab("fraction of cases") +
  background_grid(major = "y") +
  theme(legend.position = "bottom")
saveplot("stacked-bar3.pdf", height = 3, width = 4.5)
```

```{r}
coronavirus %>%
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  mutate(place = ifelse(country == "US", "US", "other")) %>%
  group_by(date, place) %>%
  summarise(cases = sum(cases)) %>%
  ggplot(aes(x=date, y=cases, fill=place)) +
  geom_col() +
  theme_half_open() +
  scale_y_continuous(labels = function(x) floor(x/100000)) +
  ylab(bquote("\u00D7"*10^5~"number of cases")) +
  background_grid(major = "y") +
  theme(legend.position = "bottom")
saveplot("stacked-bar4.pdf", height = 3, width = 4.5)

coronavirus %>%
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  mutate(place = ifelse(country == "US", "US", "other")) %>%
  group_by(date, place) %>%
  summarise(cases = sum(cases)) %>%
  ggplot(aes(x=date, y=cases, fill=place)) +
  geom_col(position="fill") +
  theme_half_open() +
  scale_y_continuous(labels = function(x) floor(x/100000)) +
  ylab(bquote("\u00D7"*10^5~"number of cases")) +
  background_grid(major = "y") +
  theme(legend.position = "bottom")
saveplot("stacked-bar5.pdf", height = 3, width = 4.5)
```

```{r}
coronavirus %>%
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  mutate(place = ifelse(country == "US", "US", "other")) %>%
  group_by(date, place) %>%
  summarise(cases = sum(cases)) %>%
  ggplot(aes(x=date, y=cases, fill=place)) +
  geom_area(color="black") +
  theme_half_open() +
  scale_y_continuous(labels = function(x) floor(x/100000)) +
  ylab(bquote("\u00D7"*10^5~"number of cases")) +
  background_grid(major = "y") +
  theme(legend.position = "bottom")
saveplot("stacked-area1.pdf", height = 3, width = 4.5)
```

```{r}
coronavirus %>%
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  mutate(place = ifelse(country == "US", "US", "other")) %>%
  group_by(date, place) %>%
  summarise(cases = sum(cases)) %>%
  ggplot(aes(x=date, y=cases, fill=place)) +
  geom_area(position="fill", color="black") +
  theme_half_open() +
  ylab("fraction of cases") +
  background_grid(major = "y") +
  theme(legend.position = "bottom")
saveplot("stacked-area2.pdf", height = 3, width = 4.5)
```

```{r}
coronavirus %>%
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  mutate(place = ifelse(country == "US", "US", "other")) %>%
  group_by(date, place) %>%
  summarise(cases = sum(cases)) %>%
  ggplot(aes(x=date, y=cases, color=place)) +
  geom_line()+
  theme_half_open() +
  scale_y_continuous(labels = function(x) floor(x/100000)) +
  ylab(bquote("\u00D7"*10^5~"number of cases")) +  background_grid(major = "y") +
  theme(legend.position = "bottom")
saveplot("line1.pdf", height = 3, width = 4.5)

coronavirus %>%
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  mutate(place = ifelse(country == "US", "US", "other")) %>%
  group_by(date, place) %>%
  summarise(cases = sum(cases)) %>%
  group_by(place) %>%
  mutate(ccases = cumsum(cases)) %>%
  ggplot(aes(x=date, y=ccases, color=place)) +
  geom_line()+
  theme_half_open() +
  scale_y_continuous(labels = function(x) floor(x/10000000)) +
  ylab(bquote("\u00D7"*10^7~"cumulative number of cases")) +  background_grid(major = "y") +
  theme(legend.position = "bottom")
saveplot("line1-cumulative.pdf", height = 4, width = 6)
```


```{r}
coronavirus %>%
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  mutate(place = ifelse(country == "US", "US", "other")) %>%
  group_by(date, place) %>%
  summarise(cases = sum(cases)) %>%
  group_by(place) %>%
  mutate(meancases = rollmean(cases, 7, fill=NA)) %>%
  ggplot(aes(x=date, y=meancases, color=place)) +
  geom_line()+
  theme_half_open() +
  scale_y_continuous(labels = function(x) floor(x/100000)) +
  ylab(bquote("\u00D7"*10^5~"number of cases")) +
  background_grid(major = "y") +
  theme(legend.position = "bottom")
saveplot("line2.pdf", height = 3, width = 4.5)
```

```{r}
coronavirus %>%
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  group_by(country) %>%
  mutate(meancases = rollmean(cases, 7, fill = NA)) %>%
  filter(!is.na(meancases)) %>%
  ggplot(aes(x=date, y=meancases, color=country)) +
  geom_line() +
  gghighlight(max(meancases) > 50000, label_params = list(segment.color="black")) +
  theme_half_open() +
  scale_y_continuous(labels = function(x) floor(x/10000)) +
  ylab(bquote("\u00D7"*10^4~"number of cases")) +
  background_grid(major = "y")
saveplot("line3.pdf", height = 3, width = 4.5)
```

```{r}
coronavirus %>%
  filter(type == "confirmed") %>%
  filter(cases > 0) %>%
  mutate(place = ifelse(country == "US", "US", "other")) %>%
  group_by(date, place) %>%
  summarise(cases = sum(cases)) %>%
  group_by(place) %>%
  mutate(meancases = rollmean(cases, 7, fill=NA)) %>%
  mutate(markers = ifelse(mday(date) == 1, meancases, NA)) %>%
  filter(!is.na(meancases)) %>%
  ggplot(aes(x=date, y=meancases, color=place)) +
  geom_line()+
  geom_point(aes(y=markers, shape=place), size=3) +
  gghighlight(label_params = list(segment.color="black")) +
  theme_half_open() +
  scale_y_continuous(labels = function(x) floor(x/100000)) +
  ylab(bquote("\u00D7"*10^5~"number of cases")) +
  background_grid(major = "y")
saveplot("line4.pdf", height = 3, width = 4.5)
```
