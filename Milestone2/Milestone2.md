Milestone2
================
pattiey

# Pattie Ye’s Mini Data Analysis: Milestone 2

First load the libraries

``` r
library(datateachr)
library(tidyverse)
# I need the lubridate package to efficiently extract the date
library(lubridate)
# I need the ggridges package
library(ggridges)
```

# Task 1: Process and summarize data

## 1.1: My research questions

Research questions are changed from milestone 1 to be more specific.

1.  How has the frequency of trees planted by neighbourhood changed over
    time? And on a broader scale, how has it changed by decade? This
    could reflect the rate of development of certain neighbourhoods over
    time.

2.  Is there a difference in the frequency of certain species of trees
    between neighbourhoods? What are the most popular species of tree
    within Vancouver and which neighbourhoods are these trees mostly
    found?

3.  How does the number of trees planted change depending on the season?

4.  Is there a relationship between the height and width of trees? Are
    these larger trees found in geographically similar areas or are they
    scattered throughout the city?

## 1.2 Summarizing and graphing

### Research Question 1

*How has the frequency of trees planted by neighbourhood changed over
time? And on a broader scale, how has it changed by decade? This could
reflect the rate of development of certain neighbourhoods over time.*

From Milestone 1, I created a variable called `year_planted` that
contains the year in which a tree was planted.

``` r
vancouver_trees <- vancouver_trees %>% 
  # use year() from lubridate to extract year from date
  mutate(year_planted = year(date_planted))
```

I just want the decade in which these trees were planted in order to get
a better sense of overall trends over time. Create a categorical
variable based on `year_planted`.

``` r
vancouver_trees <- vancouver_trees %>% 
  mutate(decade_planted = cut(year_planted, breaks = c(1990, 2000, 2010, 2020), labels = c("90's", "00's", "10's"))) 
```

I now want to see the count of trees planted per decade by
neighbourhood.

``` r
vancouver_trees %>% 
  group_by(neighbourhood_name, decade_planted) %>% 
  summarise(count = n())
```

    ## `summarise()` has grouped output by 'neighbourhood_name'. You can override using the `.groups` argument.

    ## # A tibble: 88 × 3
    ## # Groups:   neighbourhood_name [22]
    ##    neighbourhood_name decade_planted count
    ##    <chr>              <fct>          <int>
    ##  1 ARBUTUS-RIDGE      90's             826
    ##  2 ARBUTUS-RIDGE      00's            1176
    ##  3 ARBUTUS-RIDGE      10's             535
    ##  4 ARBUTUS-RIDGE      <NA>            2632
    ##  5 DOWNTOWN           90's             569
    ##  6 DOWNTOWN           00's            1179
    ##  7 DOWNTOWN           10's             464
    ##  8 DOWNTOWN           <NA>            2947
    ##  9 DUNBAR-SOUTHLANDS  90's            1111
    ## 10 DUNBAR-SOUTHLANDS  00's            1631
    ## # … with 78 more rows

Most of the trees do not contain information on `date_planted`. From the
trees with date information, we see that most neighbourhoods saw the
greatest number of trees planted between 2000-2010. This may suggest
that the decade of 2000-2010 saw the greatest increase in development,
but with large number of entries without date information, it is
difficult to tell.

I would like to see the distribution of the rate of trees being planted
(number of trees planted in a year) in a given neighbourhood and how
that distribution changes depending on the decade.

First, plot a histogram with `binwidth = 10`

``` r
vancouver_trees %>% 
  group_by(year_planted, neighbourhood_name, decade_planted) %>% 
  summarise(annual_trees_planted = n()) %>% 
  # omit trees that do not have date planted information 
  filter(!is.na(decade_planted)) %>%  
  ggplot(aes(annual_trees_planted, fill = decade_planted, color = decade_planted)) +
  geom_histogram(position = "dodge", alpha = 0.2, aes(y = ..density..), binwidth = 10) +
  labs(fill = "Decade Planted", color = "Decade Planted", title = "Density distribution of annual number of trees planted, bin = 10", x = "Number of trees planted in a given year")
```

    ## `summarise()` has grouped output by 'year_planted', 'neighbourhood_name'. You can override using the `.groups` argument.

![](Milestone2_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

Now with `binwidth = 25`

``` r
vancouver_trees %>% 
  group_by(year_planted, neighbourhood_name, decade_planted) %>% 
  summarise(annual_trees_planted = n()) %>% 
  # omit trees that do not have date planted information 
  filter(!is.na(decade_planted)) %>%  
  ggplot(aes(annual_trees_planted, fill = decade_planted, color = decade_planted)) +
  geom_histogram(position = "dodge", alpha = 0.2, aes(y = ..density..), binwidth = 25) +
  labs(fill = "Decade Planted", color = "Decade Planted", title = "Density distribution of annual number of trees planted, bin = 25", x = "Number of trees planted in a given year")
```

    ## `summarise()` has grouped output by 'year_planted', 'neighbourhood_name'. You can override using the `.groups` argument.

![](Milestone2_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

Finally, plot a histogram with `binwidth = 50`

``` r
vancouver_trees %>% 
  group_by(year_planted, neighbourhood_name, decade_planted) %>% 
  summarise(annual_trees_planted = n()) %>% 
  # omit trees that do not have date planted information 
  filter(!is.na(decade_planted)) %>%  
  ggplot(aes(annual_trees_planted, fill = decade_planted, color = decade_planted)) +
  geom_histogram(position = "dodge", alpha = 0.2, aes(y = ..density..), binwidth = 50) +
  labs(fill = "Decade Planted", color = "Decade Planted", title = "Density distribution of annual number of trees planted, bin = 50", x = "Number of trees planted in a given year")
```

    ## `summarise()` has grouped output by 'year_planted', 'neighbourhood_name'. You can override using the `.groups` argument.

![](Milestone2_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

The histogram with `binwidth = 10` produces a graph where it is hard to
see the differences between the decades and is difficult to extract
anything meaningful from it. `binwidth = 50` does not give a great
overview of the differences in distribution since the bins are quite
wide. `binwidth = 25` provides a happy medium where you can get a sense
of the distribution of the data without it being too difficult to read.
Overlaying `geom_density()` gives an even better sense of the
differences in distributions between the three decades.

``` r
vancouver_trees %>% 
  group_by(year_planted, neighbourhood_name, decade_planted) %>% 
  summarise(annual_trees_planted = n()) %>% 
  # omit trees that do not have date planted information 
  filter(!is.na(decade_planted)) %>%  
  ggplot(aes(annual_trees_planted, fill = decade_planted, color = decade_planted)) +
  geom_histogram(position = "dodge", alpha = 0.2, aes(y = ..density..), binwidth = 25) +
  geom_density(alpha = 0.3) + 
  labs(fill = "Decade Planted", color = "Decade Planted", title = "Density distribution of annual number of trees planted in a neighbourhood", x = "Number of trees planted in a given year")
```

    ## `summarise()` has grouped output by 'year_planted', 'neighbourhood_name'. You can override using the `.groups` argument.

![](Milestone2_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

From this we see that the average rate of trees being planted per
neighbourhood during the 2010’s is lower than in the 1990’s and 2000’s.
The 2000’s saw the greatest average number of trees planted per year per
neighbourhood.

### Research Question 2

*Is there a difference in the frequency of certain species of trees
between neighbourhoods? What are the most popular species of tree within
Vancouver and which neighbourhoods are these trees mostly found?*

First, we shall look at summary statistics of the number of trees per
type of tree in each neighbourhood. For each neighbourhood, find the
mean, the median, min, and max number of trees per type, the most
popular type of tree, and the total number of trees.

``` r
vancouver_trees %>% 
  group_by(common_name, neighbourhood_name) %>% 
  # get a count of trees of each type per neighbourhood
  summarise(spec_count = n()) %>% 
  group_by(neighbourhood_name) %>% 
  arrange(desc(spec_count)) %>% 
  # names aren't as descriptive as I'd like in order to fit in the document
  summarise(mean = mean(spec_count), 
            median = median(spec_count), 
            min = min(spec_count), 
            max = max(spec_count),
            pop_tree = first(common_name), 
            total = sum(spec_count))
```

    ## `summarise()` has grouped output by 'common_name'. You can override using the `.groups` argument.

    ## # A tibble: 22 × 7
    ##    neighbourhood_name        mean median   min   max pop_tree              total
    ##    <chr>                    <dbl>  <dbl> <int> <int> <chr>                 <int>
    ##  1 ARBUTUS-RIDGE             19.9    5.5     1   696 PISSARD PLUM           5169
    ##  2 DOWNTOWN                  30.7    7       1   486 RED MAPLE              5159
    ##  3 DUNBAR-SOUTHLANDS         25.9    5       1   754 KWANZAN FLOWERING CH…  9415
    ##  4 FAIRVIEW                  17.9    6       1   251 RED MAPLE              4002
    ##  5 GRANDVIEW-WOODLAND        21.6    5       1   452 KWANZAN FLOWERING CH…  6703
    ##  6 HASTINGS-SUNRISE          30.4    9       1   657 PISSARD PLUM          10547
    ##  7 KENSINGTON-CEDAR COTTAGE  34.2    9       1   739 KWANZAN FLOWERING CH… 11042
    ##  8 KERRISDALE                24.4    7       1   668 NORWAY MAPLE           6936
    ##  9 KILLARNEY                 25.9    7       1   676 CRIMEAN LINDEN         6148
    ## 10 KITSILANO                 24.2    5.5     1   874 NORWAY MAPLE           8115
    ## # … with 12 more rows

From this summary, we can get a general sense of which trees are most
popular overall in the city (Kwanzan Flowering Cherry, Red Maple, Norway
Maple). We can also tell that the distributions of numbers of trees per
type is generally quite right-skewed (mean is greater than the median)
which suggests that for most types of trees in each neighbourhood there
are only a couple trees planted, while the popular trees are great in
numbers.

Now we look at the ten most popular types of trees in the city overall
and see how the number of each of those trees per neighbourhood varies
using a jitter plot overlaid with a violin plot.

``` r
# first get the ten most popular trees in Vancouver
(most_pop_trees <- vancouver_trees %>% 
  group_by(common_name) %>% 
  summarise(spec_count = n()) %>% 
  arrange(desc(spec_count)) %>% 
  pull(common_name) %>% 
  head(10))
```

    ##  [1] "KWANZAN FLOWERING CHERRY"    "PISSARD PLUM"               
    ##  [3] "NORWAY MAPLE"                "CRIMEAN LINDEN"             
    ##  [5] "PYRAMIDAL EUROPEAN HORNBEAM" "NIGHT PURPLE LEAF PLUM"     
    ##  [7] "RED MAPLE"                   "KOBUS MAGNOLIA"             
    ##  [9] "BOWHALL RED MAPLE"           "AKEBONO FLOWERING CHERRY"

``` r
vancouver_trees %>% 
  # filter to keep only the most popular trees
  filter(common_name %in% most_pop_trees) %>% 
  group_by(common_name, neighbourhood_name) %>% 
  summarise(spec_count = n()) %>% 
  # Have tree names as a factor so that the graph is in order of popularity
  ggplot(aes(factor(common_name, levels = most_pop_trees), spec_count, color = common_name, fill = common_name)) +
  # jitter plot, we want to plot the actual number of trees, so eliminate vertical jitter
  geom_jitter(height = 0) +
  geom_violin(alpha = 0.2) +
  # Do not include legend since it is redundant
  theme(legend.position = "none") +
  labs(title = "Per neighbourhood count of most \npopular trees across Vancouver", 
       x = "Common name of tree", 
       y = "Count by neighbourhood") +
  coord_flip()
```

    ## `summarise()` has grouped output by 'common_name'. You can override using the `.groups` argument.

![](Milestone2_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

From this graph, we see that some types of trees such as the Kwanzan
Flowering Cherry are quite popular across a number of neighbourhoods,
while others such as the Red Maple are generally not as popular, but are
prominent in a few neighbourhoods.

### Research Question 3

*How does the number of trees planted change depending on the season?*

We need to create a categorical variable `season_planted`. The value of
the variable will be assigned as so:

| `season_planted` | Months                         |
|------------------|--------------------------------|
| `winter`         | Dec. (12), Jan. (1), Feb. (2)  |
| `spring`         | Mar. (3), Apr. (4), May (5)    |
| `summer`         | Jun. (6), Jul. (7), Aug. (8)   |
| `autumn`         | Sep. (9), Oct. (10), Nov. (11) |

``` r
vancouver_trees <- vancouver_trees %>% 
  mutate(season_planted = case_when(
    month(date_planted) %in% 3:5 ~ "Spring",
    month(date_planted) %in% 6:8 ~ "Summer",
    month(date_planted) %in% 9:11 ~ "Autumn",
    is.na(month(date_planted)) ~ "NA",
    # if the month is not in any of those other ranges, it must be Winter
    TRUE ~ "Winter"
  ))

vancouver_trees %>% 
  group_by(season_planted) %>% 
  summarise(count = n())
```

    ## # A tibble: 5 × 2
    ##   season_planted count
    ##   <chr>          <int>
    ## 1 Autumn         14999
    ## 2 NA             76548
    ## 3 Spring         20663
    ## 4 Summer           773
    ## 5 Winter         33628

Most of the trees do not have date information. From the trees that do
have `date_planted` information, it is shown that Winter is the most
popular season for trees to be planted, followed by Spring, Autumn, and
lastly (by a lot) Summer.

Here, we plot the distribution of trees planted by date coloured by
their season.

Start with `binwidth = 1`, each bin is one day.

``` r
vancouver_trees %>% 
  # Filter out trees without date information
  filter(!is.na(date_planted)) %>% 
  # Extract just the day of the year from date_planted
  mutate(day = yday(date_planted)) %>% 
  ggplot(aes(day, fill = season_planted)) + 
  geom_histogram(binwidth = 1) + 
  labs(x = "Day of the year", 
       fill = "Season planted", 
       title = "Distribution of trees planted \nby day of the year, binwidth = 1")
```

![](Milestone2_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

From this, we see that there is one day in Autumn that appears to be an
outlier with close to 800 trees planted in a single day. We also see
that towards the Summer, as the weather starts getting warmer, tree
planting slows down.

Now plot with `binwidth = 7`. Each bin is a week.

``` r
vancouver_trees %>% 
  # Filter out trees without date information
  filter(!is.na(date_planted)) %>% 
  # Extract just the day of the year from date_planted
  mutate(day = yday(date_planted)) %>% 
  ggplot(aes(day, fill = season_planted)) + 
  geom_histogram(binwidth = 7) + 
  labs(x = "Day of the year", 
       fill = "Season planted", 
       title = "Distribution of trees planted \nby day of the year, binwidth = 7")
```

![](Milestone2_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

From this histogram we see that the busiest weeks of the year for tree
planting are between Winter and Spring, and late Autumn. It is also
interesting to see that during the week surrounding the holiday season
in late December/early January, we see a dramatic decrease in the number
of trees planted. This histogram is a bit smoother and easier to read.

Now plot with `bins = 12`. Each bin roughly represents one month.

``` r
vancouver_trees %>% 
  # Filter out trees without date information
  filter(!is.na(date_planted)) %>% 
  # Extract just the day of the year from date_planted
  mutate(day = yday(date_planted)) %>% 
  ggplot(aes(day, fill = season_planted)) + 
  geom_histogram(bins = 12) + 
  labs(x = "Day of the year", 
       fill = "Season planted", 
       title = "Distribution of trees planted \nby day of the year, bins = 12")
```

![](Milestone2_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

From this histogram, we see that February, March/April, and
November/December are the busiest months for tree planting. Although,
this graph does not cleanly divide up the months, if we are looking for
the trees by month, it would be better to directly plot a histogram
using the month.

``` r
vancouver_trees %>% 
  # Filter out trees without date information
  filter(!is.na(date_planted)) %>% 
  # Extract just the day of the year from date_planted
  mutate(month = month(date_planted)) %>% 
  ggplot(aes(month, fill = season_planted)) + 
  geom_histogram(binwidth = 1) + 
  labs(x = "Month planted", 
       fill = "Season planted", 
       title = "Distribution of trees planted \nby month")
```

![](Milestone2_files/figure-gfm/unnamed-chunk-15-1.png)<!-- -->

Here, see that the busiest tree planting months are actually January,
February, March, and November, which is difficult to tell from the
previous histogram.

The most informative histogram in this case would be that one with
`binwidth = 7` since it is smoother than the first histogram, eliminates
the effect of the outlying day, while also being able to show
interesting trends in planting such as the decrease surrounding the
holiday season and the steady decrease and increase in planting
surrounding the Summer.

### Research Question 4

*Is there a relationship between the height and width of trees? Are
these larger trees found in geographically similar areas or are they
scattered throughout the city?*

Since `height_range_id` is an ID variable, it can be viewed as a
categorical variable. Let’s look at the summary statistics of `diameter`
across the different `height_range_id` values.

For each `height_range_id` find the range, mean, median, first and third
quartile values, as well as what type of tree the thinnest and thickest
tree are.

``` r
vancouver_trees %>% 
  group_by(height_range_id) %>% 
  summarise(min = min(diameter), 
            first_q = quantile(diameter, probs = 0.25), 
            median = median(diameter), 
            mean = mean(diameter), 
            third_q = quantile(diameter, probs = 0.75), 
            max = max(diameter))
```

    ## # A tibble: 11 × 7
    ##    height_range_id   min first_q median  mean third_q   max
    ##              <dbl> <dbl>   <dbl>  <dbl> <dbl>   <dbl> <dbl>
    ##  1               0   0       0      3    5.41     6      55
    ##  2               1   0       3      3    3.92     4      86
    ##  3               2   0       4      7    8.38    11     435
    ##  4               3   0       9     13   14.6     19     141
    ##  5               4   1      12     15.5 16.6     19.5   317
    ##  6               5   1      18     22.5 22.8     27      99
    ##  7               6   0      23     27   27.3     31      99
    ##  8               7   2.5    26     30   30.8     35      75
    ##  9               8   3      28.3   33   33.3     38      99
    ## 10               9   2      30     35   34.8     41      67
    ## 11              10   3      32.5   39   34.1     42.8    50

``` r
# For visual purposes, thinnest and thickest types are separate
vancouver_trees %>% 
  group_by(height_range_id) %>%
  arrange(diameter) %>% 
  summarise(thinnest_type = first(common_name), thickest_type = last(common_name))
```

    ## # A tibble: 11 × 3
    ##    height_range_id thinnest_type            thickest_type                 
    ##              <dbl> <chr>                    <chr>                         
    ##  1               0 QUEEN ELIZABETH MAPLE    PERSIAN IRONWOOD              
    ##  2               1 COMMON LILAC             PYRAMIDAL EUROPEAN HORNBEAM   
    ##  3               2 WORPLESDON SWEETGUM      JAPANESE SNOWBELL             
    ##  4               3 MAGNOLIA 'GALAXY'        KWANZAN FLOWERING CHERRY      
    ##  5               4 NORWAY MAPLE             MAPLE SPECIES                 
    ##  6               5 DAWYCK'S BEECH           LAWSON CYPRESS/PORT ORFORD CED
    ##  7               6 NORWAY MAPLE             WESTERN RED CEDAR             
    ##  8               7 CHANTICLEER PEAR         GIANT SEQUOIA                 
    ##  9               8 AUTUMN BLAZE RED MAPLE   AMERICAN CHESTNUT             
    ## 10               9 INGES RUBY VASE IRONWOOD WESTERN RED CEDAR             
    ## 11              10 GLOBE OR MOPHEAD ACACIA  RED OAK

Based on the median and mean values, it appears that a higher
`height_range_id` corresponds to thicker trees. But it is interesting to
note that the maximum tree diameters belong in `height_range_id` 3, 4,
and 5. There doesn’t seem to be a clear explanation for these outliers
since they are much greater than the other diameter values. Perhaps
there was a recording error.

Here, we plot the diameter against `height_range_id` using a jitter
plot.

``` r
vancouver_trees %>% ggplot(aes(factor(height_range_id, levels = 0:10), diameter)) + geom_jitter(height = 0, alpha = 0.1) + labs(y = "Diameter", x = "Height range ID", title = "Tree diameter vs. Height range ID")
```

![](Milestone2_files/figure-gfm/unnamed-chunk-17-1.png)<!-- -->

This is extremely difficult to read. Use a
*l**o**g*<sub>10</sub>
transform on the y-axis.

``` r
vancouver_trees %>% ggplot(aes(factor(height_range_id, levels = 0:10), diameter)) + geom_jitter(height = 0, alpha = 0.2) + scale_y_continuous(trans = 'log10') + labs(y = "Diameter", x = "Height range ID", title = "Tree diameter vs. Height range ID")
```

    ## Warning: Transformation introduced infinite values in continuous y-axis

    ## Warning: Removed 92 rows containing missing values (geom_point).

![](Milestone2_files/figure-gfm/unnamed-chunk-18-1.png)<!-- --> This
still is not great, but is better. You can see that, generally, as the
`height_range_id` increases, the number of smaller diameter trees
decreases. Perhaps using a box plot will give something more
interpretable.

``` r
vancouver_trees %>% 
  ggplot(aes(factor(height_range_id, levels = 0:10), diameter)) + 
  geom_boxplot(alpha = 0.2) + 
  scale_y_continuous(trans = 'log10') + 
  labs(y = "Diameter", 
       x = "Height range ID", 
       title = "Box plot of tree diameter vs. height range ID")
```

    ## Warning: Transformation introduced infinite values in continuous y-axis

    ## Warning: Removed 92 rows containing non-finite values (stat_boxplot).

![](Milestone2_files/figure-gfm/unnamed-chunk-19-1.png)<!-- -->

Indeed it does. From this, it is clear that in general, as the height
increases, the average tree diameter also increases.

For fun, I want to see if the presence of a root barrier has any
relationship with diameter.

``` r
vancouver_trees %>% 
  ggplot(aes(factor(height_range_id, levels = 0:10), diameter, color = root_barrier)) + 
  geom_boxplot(alpha = 0.2) + 
  scale_y_continuous(trans = 'log10') + 
  labs(y = "Diameter", 
       x = "Height range ID", 
       title = "Box plot of tree diameter vs. height range ID",
       color = "Root barrier")
```

    ## Warning: Transformation introduced infinite values in continuous y-axis

    ## Warning: Removed 92 rows containing non-finite values (stat_boxplot).

![](Milestone2_files/figure-gfm/unnamed-chunk-20-1.png)<!-- -->

In general, it appears that the presence of a root barrier corresponds
to a lower tree diameter.

## 1.3 Assess research questions

### Research Question 1

*How has the frequency of trees planted by neighbourhood changed over
time? And on a broader scale, how has it changed by decade? This could
reflect the rate of development of certain neighbourhoods over time.*

I am closer to answering this question in that I have identified the
different distributions of number of trees planted per year in each
neighbourhood. From this we see that the 2000’s had the highest average
number of neighbourhood trees planted annually. What remains unclear is
which neighbourhoods saw the greatest amounts of growth in each decade
and which neighbourhoods had the biggest variation of trees planted
between decades. I’d like to refine my research question to be: which
neighbourhoods saw the greatest rate of trees planted per decade? Are
there any neighbourhoods that have only recently seen a high rate of
tree planting (last 5 years?) and conversely, are there any
neighbourhoods that have drastically slowed down in their rate of tree
planting?

### Research Question 2

*Is there a difference in the frequency of certain species of trees
between neighbourhoods? What are the most popular species of tree within
Vancouver and which neighbourhoods are these trees mostly found?*

I have identified the most popular types of trees within each
neighbourhood and throughout Vancouver as a whole. What remains to be
answered is the specific neighbourhoods where the most popular trees are
mainly found. It would also be interesting to see if all the popular
trees are concentrated in one area in the neighbourhood or are scattered
around.

### Research Question 3

*How does the number of trees planted change depending on the season?*

I think this question has been sufficiently answered. The number of
trees planted decreases as the weather gets warmer in the Summer months,
as well as in late December/early January surrounding the holiday
season. It may be interesting to create a time series model for the
number of trees planted through time.

### Research Question 4

*Is there a relationship between the height and width of trees? Are
these larger trees found in geographically similar areas or are they
scattered throughout the city?*

From the data exploration, we found that on average, a larger
`height_range_id` corresponds to a larger diameter, which answers the
first part of the question. The second part of the research remains
unanswered. It would be interesting to explore the geographic
relationship between the trees that are taller (ex. `height_range_id`
&gt; 5) and wider in diameter (ex. `diameter > 20`). It would also be
interesting to see if the age of a tree (based on `date_planted`) has
any relationship with the height or diameter of a tree.

# Task 2: Tidy my data

## 2.1 Is my data tidy?

I believe my data is tidy. Each row represents one tree (one
observation), each column represents a different variable describing the
tree, and each cell represents the value corresponding to that tree. To
be sure, let’s go through the 8 variables:

-   `tree_id`
-   `std_street`
-   `common_name`
-   `height_range_id`
-   `diameter`
-   `date_planted`
-   `longitude`
-   `latitude`

``` r
vars_of_interest <- c("tree_id", "std_street", "common_name", "height_range_id", "diameter", "date_planted", "longitude", "latitude")

vancouver_trees %>% select(vars_of_interest)
```

    ## Note: Using an external vector in selections is ambiguous.
    ## ℹ Use `all_of(vars_of_interest)` instead of `vars_of_interest` to silence this message.
    ## ℹ See <https://tidyselect.r-lib.org/reference/faq-external-vector.html>.
    ## This message is displayed once per session.

    ## # A tibble: 146,611 × 8
    ##    tree_id std_street    common_name           height_range_id diameter date_planted
    ##      <dbl> <chr>         <chr>                           <dbl>    <dbl> <date>      
    ##  1  149556 W 58TH AV     BRANDON ELM                         2     10   1999-01-13  
    ##  2  149563 W 58TH AV     JAPANESE ZELKOVA                    4     10   1996-05-31  
    ##  3  149579 WINDSOR ST    JAPANESE SNOWBELL                   3      4   1993-11-22  
    ##  4  149590 E 39TH AV     AUTUMN APPLAUSE ASH                 4     18   1996-04-29  
    ##  5  149604 WINDSOR ST    HEDGE MAPLE                         2      9   1993-12-17  
    ##  6  149616 W 61ST AV     CHANTICLEER PEAR                    2      5   NA          
    ##  7  149617 SHERBROOKE ST COLUMNAR NORWAY MAPLE               3     15   1993-12-16  
    ##  8  149618 SHERBROOKE ST COLUMNAR NORWAY MAPLE               3     14   1993-12-16  
    ##  9  149619 SHERBROOKE ST COLUMNAR NORWAY MAPLE               2     16   1993-12-16  
    ## 10  149625 E 39TH AV     AUTUMN APPLAUSE ASH                 2      7.5 1993-12-03  
    ## # … with 146,601 more rows, and 2 more variables: longitude <dbl>,
    ## #   latitude <dbl>

From we can assess what the value in each variable is representing.

| Variable          | Value                                             |
|-------------------|---------------------------------------------------|
| `tree_id`         | The ID of a given tree                            |
| `std_street`      | The street where a given tree is located          |
| `common_name`     | The common name of the species/cultivar of a tree |
| `height_range_id` | Categorized height of a given tree                |
| `diameter`        | Diameter across the trunk of a given tree         |
| `date_planted`    | The date when a given tree was planted            |
| `longitude`       | The longitude of a given tree                     |
| `latitude`        | The latitude of a given tree                      |

## 2.2 Untidying my data

I can untidy my data by using `pivot_longer` to create a new variable
called `location_metric` that will either be `longitude` or `latitude`
and the value will be the longitudinal or latitudinal coordinate of the
tree. This will untidy the data since there will be two rows per tree

``` r
(untidy_vancouver_trees <- vancouver_trees %>% 
   select(vars_of_interest) %>% 
   pivot_longer(cols = c(longitude, latitude),
                names_to = "location_metric", 
                values_to = "coordinate")) %>% 
  select(location_metric, coordinate, everything())
```

    ## # A tibble: 293,222 × 8
    ##    location_metric coordinate tree_id std_street common_name         height_range_id
    ##    <chr>                <dbl>   <dbl> <chr>      <chr>                         <dbl>
    ##  1 longitude           -123.   149556 W 58TH AV  BRANDON ELM                       2
    ##  2 latitude              49.2  149556 W 58TH AV  BRANDON ELM                       2
    ##  3 longitude           -123.   149563 W 58TH AV  JAPANESE ZELKOVA                  4
    ##  4 latitude              49.2  149563 W 58TH AV  JAPANESE ZELKOVA                  4
    ##  5 longitude           -123.   149579 WINDSOR ST JAPANESE SNOWBELL                 3
    ##  6 latitude              49.2  149579 WINDSOR ST JAPANESE SNOWBELL                 3
    ##  7 longitude           -123.   149590 E 39TH AV  AUTUMN APPLAUSE ASH               4
    ##  8 latitude              49.2  149590 E 39TH AV  AUTUMN APPLAUSE ASH               4
    ##  9 longitude           -123.   149604 WINDSOR ST HEDGE MAPLE                       2
    ## 10 latitude              49.2  149604 WINDSOR ST HEDGE MAPLE                       2
    ## # … with 293,212 more rows, and 2 more variables: diameter <dbl>,
    ## #   date_planted <date>

Tidy this back up with `pivot_wider` to get one row per tree again.

``` r
untidy_vancouver_trees %>% 
  pivot_wider(names_from = location_metric, values_from = coordinate) %>% 
  select(longitude, latitude, everything())
```

    ## # A tibble: 146,611 × 8
    ##    longitude latitude tree_id std_street  common_name   height_range_id diameter
    ##        <dbl>    <dbl>   <dbl> <chr>       <chr>                   <dbl>    <dbl>
    ##  1     -123.     49.2  149556 W 58TH AV   BRANDON ELM                 2     10  
    ##  2     -123.     49.2  149563 W 58TH AV   JAPANESE ZEL…               4     10  
    ##  3     -123.     49.2  149579 WINDSOR ST  JAPANESE SNO…               3      4  
    ##  4     -123.     49.2  149590 E 39TH AV   AUTUMN APPLA…               4     18  
    ##  5     -123.     49.2  149604 WINDSOR ST  HEDGE MAPLE                 2      9  
    ##  6     -123.     49.2  149616 W 61ST AV   CHANTICLEER …               2      5  
    ##  7     -123.     49.2  149617 SHERBROOKE… COLUMNAR NOR…               3     15  
    ##  8     -123.     49.2  149618 SHERBROOKE… COLUMNAR NOR…               3     14  
    ##  9     -123.     49.2  149619 SHERBROOKE… COLUMNAR NOR…               2     16  
    ## 10     -123.     49.2  149625 E 39TH AV   AUTUMN APPLA…               2      7.5
    ## # … with 146,601 more rows, and 1 more variable: date_planted <date>

## 2.3 Picking 2 research questions

I would like to continue exploring research questions 1 and 4.

### Research Question 1

*How has the frequency of trees planted by neighbourhood changed over
time? Which neighbourhoods saw the greatest rate of trees planted per
decade? Are there any neighbourhoods that have only recently seen a high
rate of tree planting (last 5 years?) and conversely, are there any
neighbourhoods that have drastically slowed down in their rate of tree
planting?*

This question mainly interests me because of the potential it has to
serve as a signal of development in neighbourhoods.There are many
different angles from which I can approach this question.

I want to explore the refined version of this question, looking at the
relative rate of trees planted per decade in each neighbourhood.

To answer this question, I need to first filter out observations that do
not have `date_planted` information.

``` r
# keep modified dataset to answer question 1 as q1_data
(q1_data <- vancouver_trees %>% 
  filter(!is.na(date_planted)))
```

    ## # A tibble: 70,063 × 23
    ##    tree_id civic_number std_street    genus_name species_name cultivar_name  
    ##      <dbl>        <dbl> <chr>         <chr>      <chr>        <chr>          
    ##  1  149556          494 W 58TH AV     ULMUS      AMERICANA    BRANDON        
    ##  2  149563          450 W 58TH AV     ZELKOVA    SERRATA      <NA>           
    ##  3  149579         4994 WINDSOR ST    STYRAX     JAPONICA     <NA>           
    ##  4  149590          858 E 39TH AV     FRAXINUS   AMERICANA    AUTUMN APPLAUSE
    ##  5  149604         5032 WINDSOR ST    ACER       CAMPESTRE    <NA>           
    ##  6  149617         4909 SHERBROOKE ST ACER       PLATANOIDES  COLUMNARE      
    ##  7  149618         4925 SHERBROOKE ST ACER       PLATANOIDES  COLUMNARE      
    ##  8  149619         4969 SHERBROOKE ST ACER       PLATANOIDES  COLUMNARE      
    ##  9  149625          720 E 39TH AV     FRAXINUS   AMERICANA    AUTUMN APPLAUSE
    ## 10  149626          736 E 39TH AV     TILIA      EUCHLORA   X <NA>           
    ## # … with 70,053 more rows, and 17 more variables: common_name <chr>,
    ## #   assigned <chr>, root_barrier <chr>, plant_area <chr>,
    ## #   on_street_block <dbl>, on_street <chr>, neighbourhood_name <chr>,
    ## #   street_side_name <chr>, height_range_id <dbl>, diameter <dbl>, curb <chr>,
    ## #   date_planted <date>, longitude <dbl>, latitude <dbl>, year_planted <dbl>,
    ## #   decade_planted <fct>, season_planted <chr>

I only really need neighbourhood and date information in order to answer
this question, so I can just select the relevant variables.

``` r
(q1_data <- q1_data %>% 
  select(neighbourhood_name, date_planted))
```

    ## # A tibble: 70,063 × 2
    ##    neighbourhood_name       date_planted
    ##    <chr>                    <date>      
    ##  1 MARPOLE                  1999-01-13  
    ##  2 MARPOLE                  1996-05-31  
    ##  3 KENSINGTON-CEDAR COTTAGE 1993-11-22  
    ##  4 KENSINGTON-CEDAR COTTAGE 1996-04-29  
    ##  5 KENSINGTON-CEDAR COTTAGE 1993-12-17  
    ##  6 KENSINGTON-CEDAR COTTAGE 1993-12-16  
    ##  7 KENSINGTON-CEDAR COTTAGE 1993-12-16  
    ##  8 KENSINGTON-CEDAR COTTAGE 1993-12-16  
    ##  9 KENSINGTON-CEDAR COTTAGE 1993-12-03  
    ## 10 KENSINGTON-CEDAR COTTAGE 1993-12-03  
    ## # … with 70,053 more rows

Since I’m interested in looking at the annual rate of planting, I’ll
create a `year_planted` variable.

``` r
(q1_data <- q1_data %>% 
  mutate(year_planted = lubridate::year(date_planted)))
```

    ## # A tibble: 70,063 × 3
    ##    neighbourhood_name       date_planted year_planted
    ##    <chr>                    <date>              <dbl>
    ##  1 MARPOLE                  1999-01-13           1999
    ##  2 MARPOLE                  1996-05-31           1996
    ##  3 KENSINGTON-CEDAR COTTAGE 1993-11-22           1993
    ##  4 KENSINGTON-CEDAR COTTAGE 1996-04-29           1996
    ##  5 KENSINGTON-CEDAR COTTAGE 1993-12-17           1993
    ##  6 KENSINGTON-CEDAR COTTAGE 1993-12-16           1993
    ##  7 KENSINGTON-CEDAR COTTAGE 1993-12-16           1993
    ##  8 KENSINGTON-CEDAR COTTAGE 1993-12-16           1993
    ##  9 KENSINGTON-CEDAR COTTAGE 1993-12-03           1993
    ## 10 KENSINGTON-CEDAR COTTAGE 1993-12-03           1993
    ## # … with 70,053 more rows

Rather than looking at the data from a tree-by-tree basis, I’d rather
explore by neighbourhood.

``` r
(q1_data <- q1_data %>% 
  group_by(neighbourhood_name, year_planted) %>% 
  summarise(trees_planted = n()))
```

    ## `summarise()` has grouped output by 'neighbourhood_name'. You can override using the `.groups` argument.

    ## # A tibble: 671 × 3
    ## # Groups:   neighbourhood_name [22]
    ##    neighbourhood_name year_planted trees_planted
    ##    <chr>                     <dbl>         <int>
    ##  1 ARBUTUS-RIDGE              1989            41
    ##  2 ARBUTUS-RIDGE              1990            76
    ##  3 ARBUTUS-RIDGE              1991            16
    ##  4 ARBUTUS-RIDGE              1992            81
    ##  5 ARBUTUS-RIDGE              1993            18
    ##  6 ARBUTUS-RIDGE              1994            58
    ##  7 ARBUTUS-RIDGE              1995           151
    ##  8 ARBUTUS-RIDGE              1996            95
    ##  9 ARBUTUS-RIDGE              1997            61
    ## 10 ARBUTUS-RIDGE              1998            59
    ## # … with 661 more rows

I would like 1 row per neighbourhood, with each annual number of trees
planted as a variable.

``` r
(q1_data <- q1_data %>% 
  pivot_wider(names_from = year_planted, values_from = trees_planted))
```

    ## # A tibble: 22 × 32
    ## # Groups:   neighbourhood_name [22]
    ##    neighbourhood_name    `1989` `1990` `1991` `1992` `1993` `1994` `1995` `1996`
    ##    <chr>                  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>
    ##  1 ARBUTUS-RIDGE             41     76     16     81     18     58    151     95
    ##  2 DOWNTOWN                   6     32     21     16     13     63     95     18
    ##  3 DUNBAR-SOUTHLANDS         65     27     24     71     36     37    141    202
    ##  4 FAIRVIEW                  NA     60     14      1      4      3     97     64
    ##  5 GRANDVIEW-WOODLAND        26     99     13     29     61    108     60     54
    ##  6 HASTINGS-SUNRISE          NA    389     56    299    411    316    224    231
    ##  7 KENSINGTON-CEDAR COT…      7     41     85    131    302    251    206    368
    ##  8 KERRISDALE                 2     31      7     33     87      5    119    127
    ##  9 KILLARNEY                 30     20     34     43     90    161    159    150
    ## 10 KITSILANO                 NA     55      9     10     62     99    101    124
    ## # … with 12 more rows, and 23 more variables: 1997 <int>, 1998 <int>,
    ## #   1999 <int>, 2000 <int>, 2001 <int>, 2002 <int>, 2003 <int>, 2004 <int>,
    ## #   2005 <int>, 2006 <int>, 2007 <int>, 2008 <int>, 2009 <int>, 2010 <int>,
    ## #   2011 <int>, 2012 <int>, 2013 <int>, 2014 <int>, 2015 <int>, 2016 <int>,
    ## #   2017 <int>, 2018 <int>, 2019 <int>

There are a few NA values, these are years where no trees were recorded,
replace the NA values with 0

``` r
(q1_data <- q1_data %>% 
  replace(is.na(.), 0))
```

    ## # A tibble: 22 × 32
    ## # Groups:   neighbourhood_name [22]
    ##    neighbourhood_name    `1989` `1990` `1991` `1992` `1993` `1994` `1995` `1996`
    ##    <chr>                  <int>  <int>  <int>  <int>  <int>  <int>  <int>  <int>
    ##  1 ARBUTUS-RIDGE             41     76     16     81     18     58    151     95
    ##  2 DOWNTOWN                   6     32     21     16     13     63     95     18
    ##  3 DUNBAR-SOUTHLANDS         65     27     24     71     36     37    141    202
    ##  4 FAIRVIEW                   0     60     14      1      4      3     97     64
    ##  5 GRANDVIEW-WOODLAND        26     99     13     29     61    108     60     54
    ##  6 HASTINGS-SUNRISE           0    389     56    299    411    316    224    231
    ##  7 KENSINGTON-CEDAR COT…      7     41     85    131    302    251    206    368
    ##  8 KERRISDALE                 2     31      7     33     87      5    119    127
    ##  9 KILLARNEY                 30     20     34     43     90    161    159    150
    ## 10 KITSILANO                  0     55      9     10     62     99    101    124
    ## # … with 12 more rows, and 23 more variables: 1997 <int>, 1998 <int>,
    ## #   1999 <int>, 2000 <int>, 2001 <int>, 2002 <int>, 2003 <int>, 2004 <int>,
    ## #   2005 <int>, 2006 <int>, 2007 <int>, 2008 <int>, 2009 <int>, 2010 <int>,
    ## #   2011 <int>, 2012 <int>, 2013 <int>, 2014 <int>, 2015 <int>, 2016 <int>,
    ## #   2017 <int>, 2018 <int>, 2019 <int>

### Research Question 4 (Now referred to as question 2)

*Is there a relationship between the height and width of trees? Are
these larger trees found in geographically similar areas or are they
scattered throughout the city?*

I’d like to continue exploring this questions because I have not yet
answered the second part of it. I’d also like to add a part to this
question that explores the relationship between the age of a tree (based
on `date_planted`) and the size of the tree.

I would like to only keep the relevant variables to the question, which
are `height_range_id`, `diameter`, `latitude`, `longitude`,
`date_planted`. It may be useful to have `on_street_block`, `on_street`,
`root_barrier`, `common_name`, and `neighbourhood_name` as well for the
exploration.

``` r
(q2_data <- vancouver_trees %>% 
  select(height_range_id, diameter, latitude, longitude, date_planted, on_street_block, on_street, root_barrier, common_name, neighbourhood_name))
```

    ## # A tibble: 146,611 × 10
    ##    height_range_id diameter latitude longitude date_planted on_street_block
    ##              <dbl>    <dbl>    <dbl>     <dbl> <date>                 <dbl>
    ##  1               2     10       49.2     -123. 1999-01-13               400
    ##  2               4     10       49.2     -123. 1996-05-31               400
    ##  3               3      4       49.2     -123. 1993-11-22              4900
    ##  4               4     18       49.2     -123. 1996-04-29               800
    ##  5               2      9       49.2     -123. 1993-12-17              5000
    ##  6               2      5       49.2     -123. NA                       500
    ##  7               3     15       49.2     -123. 1993-12-16              4900
    ##  8               3     14       49.2     -123. 1993-12-16              4900
    ##  9               2     16       49.2     -123. 1993-12-16              4900
    ## 10               2      7.5     49.2     -123. 1993-12-03               700
    ## # … with 146,601 more rows, and 4 more variables: on_street <chr>,
    ## #   root_barrier <chr>, common_name <chr>, neighbourhood_name <chr>

Height range ID is currently a double type, since it is an ID value, it
would make more sense to cast it to a factor.

``` r
(q2_data <- q2_data %>% 
   mutate(height_range_id = factor(height_range_id, 0:10)))
```

    ## # A tibble: 146,611 × 10
    ##    height_range_id diameter latitude longitude date_planted on_street_block
    ##    <fct>              <dbl>    <dbl>     <dbl> <date>                 <dbl>
    ##  1 2                   10       49.2     -123. 1999-01-13               400
    ##  2 4                   10       49.2     -123. 1996-05-31               400
    ##  3 3                    4       49.2     -123. 1993-11-22              4900
    ##  4 4                   18       49.2     -123. 1996-04-29               800
    ##  5 2                    9       49.2     -123. 1993-12-17              5000
    ##  6 2                    5       49.2     -123. NA                       500
    ##  7 3                   15       49.2     -123. 1993-12-16              4900
    ##  8 3                   14       49.2     -123. 1993-12-16              4900
    ##  9 2                   16       49.2     -123. 1993-12-16              4900
    ## 10 2                    7.5     49.2     -123. 1993-12-03               700
    ## # … with 146,601 more rows, and 4 more variables: on_street <chr>,
    ## #   root_barrier <chr>, common_name <chr>, neighbourhood_name <chr>

I’d also like to sort my data in order of increasing `height_range_id`.

``` r
(q2_data <- q2_data %>% 
   arrange(height_range_id))
```

    ## # A tibble: 146,611 × 10
    ##    height_range_id diameter latitude longitude date_planted on_street_block
    ##    <fct>              <dbl>    <dbl>     <dbl> <date>                 <dbl>
    ##  1 0                    4       NA         NA  NA                      4100
    ##  2 0                    6       49.2     -123. NA                      3600
    ##  3 0                   28.5     49.3     -123. NA                      3400
    ##  4 0                    0       NA         NA  NA                      3800
    ##  5 0                    0       NA         NA  NA                      4500
    ##  6 0                    3       49.3     -123. 2010-10-20               300
    ##  7 0                    3       49.3     -123. 2010-10-20               300
    ##  8 0                    0       NA         NA  NA                      6000
    ##  9 0                    6.5     49.2     -123. 2010-03-08              2500
    ## 10 0                    8       49.2     -123. 2010-03-08              2500
    ## # … with 146,601 more rows, and 4 more variables: on_street <chr>,
    ## #   root_barrier <chr>, common_name <chr>, neighbourhood_name <chr>

There are some trees that do not have location information or date
planted information, filter those out.

``` r
(q2_data <- q2_data %>% 
   filter(!is.na(longitude) | !is.na(latitude) | !is.na(date_planted)))
```

    ## # A tibble: 135,158 × 10
    ##    height_range_id diameter latitude longitude date_planted on_street_block
    ##    <fct>              <dbl>    <dbl>     <dbl> <date>                 <dbl>
    ##  1 0                    6       49.2     -123. NA                      3600
    ##  2 0                   28.5     49.3     -123. NA                      3400
    ##  3 0                    3       49.3     -123. 2010-10-20               300
    ##  4 0                    3       49.3     -123. 2010-10-20               300
    ##  5 0                    6.5     49.2     -123. 2010-03-08              2500
    ##  6 0                    8       49.2     -123. 2010-03-08              2500
    ##  7 0                    3       NA         NA  2010-03-11              1300
    ##  8 0                    3       NA         NA  2010-03-11              1300
    ##  9 0                    3       NA         NA  2010-03-11              1300
    ## 10 0                   21       49.2     -123. NA                      6200
    ## # … with 135,148 more rows, and 4 more variables: on_street <chr>,
    ## #   root_barrier <chr>, common_name <chr>, neighbourhood_name <chr>

I may create some categorical variables based on diameter and
date\_planted, but will leave it for now.
