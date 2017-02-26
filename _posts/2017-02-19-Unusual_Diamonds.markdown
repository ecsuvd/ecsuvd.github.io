---
layout: post
title:  "Unusual Diamonds"
date:   2017-02-19  18:41:20
categories: blog
---

One of my recent favorite book is ["R for Data Science"][r4ds-url] by Wickham and Grolemund. 
It helps me greatly to improve my skills in R. I did most of its exercises and 
posted some of the solutions on my [github][github] account. 

The book shows analyses on some datasets which are either part of a package or a dataset-package.
Most of the packages are released by the authors and are great for analysing data. 
One of the datasets is called [Diamonds][diamond-link] which is a part of [ggplot2][ggplot-link] plotting package. 
The package has data of diamond prices, carats and the other main characteristics.

The [modelling part][model-link] of the book shows linear model 
predictions of the diamond prices from the dataset. While a vast majority of the diamond 
prices fall within some margins from the prediction, a few diamonds have unexplainably different 
prices. 

The author firstly selects diamonds under 2.5 ct which are 99.7 % of all and then creates two 
different linear models in which one of them considers carat as price factor and the other one 
carat, color, cut and, clarity. After that filters cases with unusually high 
residuals: more than twice or less than half the prediction. In one of the exercises, the reader is asked to explore whether there is anything unusual about those diamonds or not. The book steps are shown below:
    {% highlight R %}

# Loading packages.
libr    ary(tidyverse)
library(modelr)
library(gridExtra)
# Filtering diamonds under 2.5 ct. 
diamonds    2 <- diamonds %>% 
  filter(carat <= 2.5) %>% 
  mutate(lprice = log2(price), lcarat = log2(carat))
# Model 1: Price vs carat.
mod_diamond     <- lm(lprice ~ lcarat, data = diamonds2)
diamonds2 <- diamonds2 %>% 
  add_residuals(mod_diamond, "lresid")
# Model 2: Price vs carat, color, cut, and clarity. 
mod_diamond2 <-     lm(lprice ~ lcarat + color + cut + clarity, data = diamonds2)
diamonds2 <- diamonds2 %>% 
  add_residuals(mod_diamond2, "lresid2")
# Filtering unusual cases. 
unusual <- diamonds2 %>% 
  filter(abs(lresid2) > 1) %>% 
  add_predictions(mod_diamond2) %>% 
  mutate(pred = round(2 ^ pred)) %>% 
  select(price, pred, carat:table, x:z) %>% 
  arrange(price)
{% endhighlight %}
That shows me following table:
{% highlight R %}
> colnames(unusual)[colnames(unusual) == 'pred'] <- 'predicted'
> unusual   
# A tibble: 16 <U+00D7> 11
   price predicted carat       cut color clarity depth table     x     y     z
      <int> <dbl> <dbl>     <ord> <ord>   <ord> <dbl> <dbl> <dbl> <dbl> <dbl>
      1   1013   264  0.25      Fair     F     SI2  54.4    64  4.30  4.23  2.32
      2   1186   284  0.25   Premium     G     SI2  59.0    60  5.33  5.28  3.12
      3   1186   284  0.25   Premium     G     SI2  58.8    60  5.33  5.28  3.12
      4   1262  2644  1.03      Fair     E      I1  78.2    54  5.72  5.59  4.42
      5   1415   639  0.35      Fair     G     VS2  65.9    54  5.57  5.53  3.66
      6   1415   639  0.35      Fair     G     VS2  65.9    54  5.57  5.53  3.66
      7   1715   576  0.32      Fair     F     VS2  59.6    60  4.42  4.34  2.61
      8   1776   412  0.29      Fair     F     SI1  55.8    60  4.48  4.41  2.48
      9   2160   314  0.34      Fair     F      I1  55.8    62  4.72  4.60  2.60
      10  2366   774  0.30 Very Good     D    VVS2  60.6    58  4.33  4.35  2.63
      11  3360  1373  0.51   Premium     F     SI1  62.7    62  5.09  4.96  3.15
      12  3807  1540  0.61      Good     F     SI2  62.5    65  5.36  5.29  3.33
      13  3920  1705  0.51      Fair     F    VVS2  65.4    60  4.98  4.90  3.23
      14  4368  1705  0.51      Fair     F    VVS2  60.7    66  5.21  5.11  3.13
      15 10011  4048  1.01      Fair     D     SI2  64.6    58  6.25  6.20  4.02
      16 10470 23622  2.46   Premium     E     SI2  59.7    59  8.82  8.76  5.25
{% endhighlight %}
There are 16 cases totally. Most of them seem to be small diamonds with high prices. 
First, I will check for anything unusual regarding their x, y, z dimensions in case their carat 
sizes are wrongly registered in the database.
Average x, y, z sizes of per carat category will be calculated and our unusual cases will be compared.
{% highlight     R %}
# round() is used to get rid of numerical errors by computer. 
sample_carat = round(seq(0.2, 2.5, 0.01), digits = 3) 
xmean <- 0
ymean <- 0
zmean <- 0
for(i in seq_along(sample_carat)) {
  diamond_by_carat <- filter(diamonds2, carat == sample_carat[i]) 
  xmean[i] <- mean(diamond_by_carat$x)
  ymean[i] <- mean(diamond_by_carat$y)
  zmean[i] <- mean(diamond_by_carat$z)
}
average_size <- tibble(
  carat = sample_carat, 
  x_ave = xmean, 
  y_ave = ymean, 
  z_ave = zmean
)
pdf("plot1.pdf", width = 6, height = 2)
p1 <- ggplot() + 
  geom_point(data = unusual, aes(carat,x)) + 
  geom_smooth(data = average_size, aes(carat, x_ave))
p2 <- ggplot() +
  geom_point(data = unusual, aes(carat,y)) +
  geom_smooth(data = average_size, aes(carat, y_ave))
p3 <- ggplot() + 
  geom_point(data = unusual, aes(carat,z)) + 
  geom_smooth(data = average_size, aes(carat, z_ave))
grid.arrange(p1, p2, p3, ncol = 3)
dev.off()
{% endhighlight %}
The code gives this plot:

![]({{ site.url }}/assets/plot1.jpg)

As I can see some diamonds have x, y, z sizes far from the average of the same carat diamonds. 
I will mark if any diamond is more than 10 % off from the average size and label them as 
having wrongly registered carats. I will also mark whether the depth and table of the diamonds lie 
outside recommended ranges checked by a Google search. 
{% highlight R %}
unusual <- unusual %>% 
  left_join(average_size, by = "carat") %>% 
  mutate(wrong_carat_label = ifelse(
    abs(x - x_ave) / x_ave > 0.1 |
    abs(y - y_ave) / y_ave > 0.1 |
    abs(z - z_ave) / z_ave > 0.1, "Yes", "No"
    )
  ) %>%
  mutate(non_st_depth = ifelse(depth > 64 | depth < 58, "Yes", "No")) %>%
  mutate(non_st_table = ifelse(table > 64 | table < 53, "Yes", "No"))
{% endhighlight %}
After this the cases look as below:
{% highlight R %}
> select(unusual, price, predicted, wrong_carat_label, non_st_depth, non_st_table)
    
# A tibble: 16 <U+00D7> 5
   p    rice predicted wrong_carat_label non_st_depth non_st_table
      <int>     <dbl>             <chr>        <chr>        <chr>
      1   1013       264                No          Yes           No
      2   1186       284               Yes           No           No
      3   1186       284               Yes           No           No
      4   1262      2644               Yes          Yes           No
      5   1415       639               Yes          Yes           No
      6   1415       639               Yes          Yes           No
      7   1715       576                No           No           No
      8   1776       412                No          Yes           No
      9   2160       314                No          Yes           No
      10  2366       774                No           No           No
      11  3360      1373                No           No           No
      12  3807      1540                No           No          Yes
      13  3920      1705                No          Yes           No
      14  4368      1705                No           No          Yes
      15 10011      4048                No          Yes           No
      16 10470     23622                No           No           No
{% endhighlight %}
From the table above, row 2 to 6 seem to be registered as having wrong carat sizes. Moreover, 
diamond in row 4 is very deep. Perhaps, it does not reflect light nicely in addition to its bad 
clarity. 
That is probably why the diamond has a significantly low price. Furthermore, some diamonds are 
either too shallow or deep or have too flat tables, but yet seem to have higher price than estimated.
It is hard to spot the reason within the given information. I am guessing that they may have 
some special characteristics. 

Despite my opinion above, there is one interesting case: Diamond on row 16 or the last one is 
large and seem to be underpriced. I will spend some time to investigate. For that purpose I will
filter out large diamonds (> 2 ct) and explore their characteristics.
{% highlight R %}
big_diamonds <- filter(diamonds, carat > 2.0)
pdf("plot2.pdf", width = 4, height = 2.5)
ggplot(big_diamonds, aes(carat, price, color = clarity)) + geom_point() 
dev.off()
{% endhighlight %}

![]({{ site.url }}/assets/plot2.jpg)

Here, we see that clarity reduces prices of large diamonds significantly (I1 means bad and IF 
means good). However, the diamond being investigated has a rather low price for the same 
clarity and carat category. Now, I will see price distribution by color and cut, but filter out 
the ones with bad clarity.
{% highlight R %}
pdf("plot3.pdf", width = 8, height = 2.5)
p1 <- ggplot(filter(diamonds, clarity != "I1"), aes(carat, price, color = color)) + 
  geom_point()
p2 <- ggplot(filter(big_diamonds, clarity != "I1"), aes(carat, price, color = cut)) + 
  geom_point()
grid.arrange(p1, p2, ncol = 2)
dev.off()
{% endhighlight %}

![]({{ site.url }}/assets/plot3.jpg)
 
I can see in the left plot that our diamond is the only brown spot around 2.5 ct among 
purple and pink dots (purple, pink means nearly colorless category whereas brown means colorless). 
It seems underpriced for its color category too. But I can not say much about its cut. Obviously, 
premium should mean good. My suspicion is that the diamond is either underpriced or one of its characteristics are swrongly registered with or without intention.

I will add my concluded hypothesis to the table now. 
{% highlight R %}
unusual = mutate(unusual, 
  guessed_reasons = c("depth", "wrong carat", "wrong carat", "wrong carat+deep", "wrong carat", "wrong carat", "unknown", "depth", "depth", "unknown", "unknown", "table", "depth", "table", "depth", "unknown"))
{% endhighlight %}
That shows to me the following:
{% highlight R %}
> select(unusual, price, predicted, wrong_carat_label, non_st_depth, non_st_table, guessed_reasons)
 # A tibble: 16 <U+00D7> 6
    price predicted wrong_carat_label non_st_depth non_st_table  guessed_reasons
       <int>     <dbl>             <chr>        <chr>        <chr>            <chr>
       1   1013       264                No          Yes           No            depth
       2   1186       284               Yes           No           No      wrong carat
       3   1186       284               Yes           No           No      wrong carat
       4   1262      2644               Yes          Yes           No wrong carat+deep
       5   1415       639               Yes          Yes           No      wrong carat
       6   1415       639               Yes          Yes           No      wrong carat
       7   1715       576                No           No           No          unknown
       8   1776       412                No          Yes           No            depth
       9   2160       314                No          Yes           No            depth
       10  2366       774                No           No           No          unknown
       11  3360      1373                No           No           No          unknown
       12  3807      1540                No           No          Yes            table
       13  3920      1705                No          Yes           No            depth
       14  4368      1705                No           No          Yes            table
       15 10011      4048                No          Yes           No            depth
       16 10470     23622                No           No           No          unknown
{% endhighlight %}


[r4ds-url]: http://r4ds.had.co.nz 
[github]: https://github.com/ecsuvd/R-learning
[diamond-link]: http://docs.ggplot2.org/0.9.3.1/diamonds.html
[ggplot-link]: http://ggplot2.org/
[model-link]: http://r4ds.had.co.nz/model-building.html
