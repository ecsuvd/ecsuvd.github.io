---
layout: post
title: "Employee Turnover (part II - Salary and Promotion)"
date: 2018-04-20  10:41:20
categories: blog
---
It has been a long time since I posted my last post. 
The reason is that I have been working busy 
and did not really have time to catch up my blog. Anyhow, I 
think now is a good time restart my hobby again. 
It seems that I promised to write one more post on the HR analytics. 
To keep up my promise I am writing this post.  

In my previous [post][pre-post] I showed some investigative analysis 
on employees turnover dataset which I got from [kaggle][hr-url]. 
Besides my main findings there were a few 
questions that still remained unanswered to me. 

My next step was to perform some more analysis which could give me answers. 
I was particularly interested in:
- Who gets promoted? 
- Who gets a high salary?  
    
# Promotion
I reloaded and continued on my previous investigation in R. 
{% highlight R %}
> hr_data %>%
+ group_by(promotion_last_5years) %>%
+ summarise(n())
# A tibble: 2 <U+00D7> 2
  promotion_last_5years `n()`
                   <fctr> <int>
1                    No 14680
2                   Yes   319
{% endhighlight %}


[pre-post]: http://data.altai.se
[hr-url]: https://www.kaggle.com/ludobenistant/hr-analytics 

Apparently, promotion is kind of rare in this company and
the data is expectedly skewed. 

I tried the tree classification, the ggpair plots and a few more bar 
plots similar to what I did for the turnover 
analysis in my previous post 
to get a little bit more insight. However, the findings were not really fruitful. 
Not sure whether the analysis are worth to share here. 
In short, the tree method showed that the ones who belonged to the 
management department, worked between 6 and 8 years and also responsible 
for less than 4 projects were surely promoted. These people weighed about 
7 percent of the total promotions. The other promotions according to ggpair 
plots showed that were pretty random, and could not really be revealed by the dataset.    

In conclusion, it remained unclear who gets promoted except the upper level 
managers. What does even promotion mean here - to become a manager? 

# Salary 
{% highlight R %} 
> high_salary <- hr_data %>% 
+   mutate(
+     high_sal = ifelse(salary == 'high', 'yes', 'no')
+   )
> high_salary %>% 
+   group_by(high_sal) %>%
+   summarise(n())  
# A tibble: 2 x 2
  high_sal `n()`
  <chr>    <int>
1 no       13762
2 yes       1237
> 
> ggpairs(high_salary, columns = 1:5, aes(color = high_sal, alpha = 0.01))
{% endhighlight %}  

 
{:refdef: style="text-align: center;"}
![]({{ site.url }}/assets/hr_plot10.jpg){:height="500px" width="500px"}
{: refdef}


Again, pretty random distributions except one tendency - folk who 
worked longer than 6 year were more likely to be paid high than the rest, 
though not necessarily be favored by their employer according to their last 
evaluation.

{% highlight R %}
ggplot(hr_data) +
  geom_bar(aes(x = factor(time_spend_company), 
               fill = factor(salary, levels = c('low', 'medium', 'high'))), 
           position = "fill") + 
  labs(x = "Time spent, years", y = "Relative ratio", fill = "Salary") 
{% endhighlight %}

{:refdef: style="text-align: center;"}
![]({{ site.url }}/assets/hr_plot11.jpg){:height="250px" width="500px"}
{: refdef}


The bar plot showed that in fact the chance of being low-paid was 
about 20% less and the chance of being high-paid was 10% more after 
working more than 6 years in the company. 
However, this does not seem to be the key of being high-paid. The plots 
made me conclude that the salary is probably based on the individual's 
negotation skill rather than the employer's satisfaction or anything else. 
By working longer in the company increases the total number of 
salary talks and each discussion adds a chance of raising one's salary. 

# Conclusions
My brief analysis did not bring me far. A few obvious conclusions were drawn.
- certain people who holds managerial position gets promoted. 
For others, the promotion occurs in an uncertain way. 
- working more than 6 years increases the chance of being better paid. 

Honestly, I lost my interest in this dataset as it has been a 
long time since I got the dataset.
I will try to write a more interesting post the next time. 

 









