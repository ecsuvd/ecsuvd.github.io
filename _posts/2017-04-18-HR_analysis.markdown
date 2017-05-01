---
layout: post
title: "Employee Turnover Analysis (part I)"
date: 2017-04-18  18:41:20
categories: blog
---

Recently I was surfing on [kaggle][kaggle-url] and got some interesting datasets. 
One of them was [Human Resources Analytics][hr-url]. The key question was "Why are our best 
and most experienced employees leaving prematurely?". I was curious and wanted to find 
out the answer to that question in my own way. Even though there was a footnote that 
stated the data was simulated I still thought it was interesting regardless 
if it was real or simulated.     

I loaded the data in R and made a little bit polishing for a better presentation.  
{% highlight R %}
# Loading library:
library(tidyverse)
library(corrplot)
library(GGally)
library(gridExtra)
library(rpart)
library(rpart.plot)

# Loading and polishing:
raw_data <- read_csv('Downloads/HR_comma_sep.csv')
hr_data <- raw_data
colnames(hr_data)[colnames(hr_data) == 'Work_accident'] <- 'work_accident'
colnames(hr_data)[colnames(hr_data) == 'average_montly_hours'] <- 'average_monthly_hours'
hr_data <- hr_data %>% 
  mutate(
    left = factor(left, labels = c("No", "Yes")),
    work_accident = factor(work_accident, labels = c("No", "Yes")),
    promotion_last_5years = factor(promotion_last_5years, labels = c("No", "Yes"))
  )
{% endhighlight %}
The data contained 10 columns of information of about 15000 employees: employee's satisfaction, 
last evaluation (I guess employer's evaluation), number of projects involved, 
average monthly hours, time spend in the company, work accident, promotion in 
last 5 years, type of position, salary category, and the employee left or not status.
{% highlight R %}
> head(hr_data)
# A tibble: 14,999 <U+00D7> 10
   satisfaction_level last_evaluation number_project average_monthly_hours
                   <dbl>           <dbl>          <int>                 <int>
                   1                0.38            0.53              2                   157
                   2                0.80            0.86              5                   262
                   3                0.11            0.88              7                   272
                   4                0.72            0.87              5                   223
                   5                0.37            0.52              2                   159
                   6                0.41            0.50              2                   153
                   7                0.10            0.77              6                   247
                   8                0.92            0.85              5                   259
                   9                0.89            1.00              5                   224
                   10               0.42            0.53              2                   142
# ... with 14,989 more rows, and 6 more variables: time_spend_company <int>,
#   work_accident <fctr>, left <fctr>, promotion_last_5years <fctr>, sales <chr>,
#   salary <chr>
{% endhighlight %}

# Quick scan analysis

Firstly, I checked whether there were any correlations between numerical parameters.
{% highlight R %} 
x <- cor(select(raw_data, -sales, -salary)) 
corrplot(x, type = "upper", order = "hclust", tl.srt = 45)
{% endhighlight %}

![]({{ site.url }}/assets/hr_plot1.jpg){:height="500px" width="500px"}

This correlation plot showed some obvious but not so strong tendencies: 
1. Low satisfaction leads employee to leave. 
2. Being involved in more number of projects leads to more hours of work.
3. An employee who works on more number of projects and works more hours is 
considered "good" workers by the employer or vice versa. 

Overall, the plot did not give any strong clues. Then as a next step, I made pair plots 
between the discrete or continuous variables. 
{% highlight R %}
ggpairs(hr_data, columns = 1:5, aes(color = left, alpha = 0.01))
{% endhighlight %}

![]({{ site.url }}/assets/hr_plot2.jpg)

My first revealings came - There were clustered patterns of leaving: 
1. A group of unsatisfied, but highly evaluated and over worked ones. 
I would call them as burnt-out workers.  
2. A group of medium-low satisfied, medium-low evaluated ones who worked 
less than average. I would call them unmotivated workers. 
The work environment was not the best suit for them.  
3. A group of highly satisfied and highly evaluated ones who spent more 
than average amount of hours at work. It seems like a motivated, 
high performing group who chose to leave. It was stated as the main question 
of the dataset. Let me call them mysterious leavers.

One more observation was that the ones who left had worked under 6 years. 
Moreover, the ones, who worked over approximately 280 hours, all left. 

To complete my first order check a few more categorical or binary data 
explorations followed for the remaining variables. 
{% highlight R %}
# Saving plots as:
pdf("plot3.pdf", width = 6, height = 2)
# Calculating the percentage of leaving:
p1.pct <- hr_data %>% 
  group_by(work_accident) %>%
  summarise(
    total = n(), 
    pct = sum(left == "Yes") / n()
  )
p2.pct <- hr_data %>% 
  group_by(promotion_last_5years) %>%
  summarise(
    total = n(), 
    pct = sum(left == "Yes") / n()
  )
# Plotting:  
p1 <- ggplot(hr_data, aes(x = work_accident)) + 
  geom_bar(aes(fill = left)) +
  labs(x = "Work Accident", y = "", fill = "Left") + 
  geom_text(data = p1.pct, 
            aes(y = total * pct / 2, label = paste0(round(pct * 100, 1), "%")), 
            size = 4)
p2 <- ggplot(hr_data, aes(x = promotion_last_5years)) + 
  geom_bar(aes(fill = left)) +
  labs(x = "Promotion in Last 5 Years", y = "", fill = "Left") +
  geom_text(data = p2.pct, 
            aes(y = total * pct / 2, label = paste0(round(pct * 100, 1), "%")), 
            size = 4)
grid.arrange(p1, p2, ncol = 2)
dev.off()
{% endhighlight %} 

![]({{ site.url }}/assets/hr_plot3.jpg){:height="200px" width="600px"}

Surprisingly, work accident seemed not to be the reason of leaving. 
On the other hand, there were lesser probability of leaving if employee was promoted.

{% highlight R %}
p3.pct <- hr_data %>% 
  group_by(sales) %>%
  summarise(
    total = n(), 
    pct = sum(left == "Yes") / n()
  )
ggplot(hr_data, aes(x = sales)) +
  geom_bar(aes(fill = left)) +
  labs(x = "", y = "", fill = "Left") +
  geom_text(data = p3.pct, 
            aes(y = total * pct / 2, label = paste0(round(pct * 100, 1), "%")), size = 2) +
  coord_flip() 
{% endhighlight %}

![]({{ site.url }}/assets/hr_plot4.jpg){:height="500px" width="500px"}

No strong dependency of quitting by their job positions could be said. 
HR had the highest rate of quitting whereas management had the lowest.

{% highlight R %} 
p4.pct <- hr_data %>% 
  group_by(salary) %>%
  summarise(
    total = n(), 
    pct = sum(left == "Yes") / n()
  )
ggplot(hr_data, aes(x = salary)) +
  geom_bar(aes(fill = left)) +
  labs(x = "", y = "", fill = "Left") +
  geom_text(data = p4.pct, 
            aes(y = total * pct / 2, label = paste0(round(pct * 100, 1), "%")), size = 4)
{% endhighlight %}

![]({{ site.url }}/assets/hr_plot5.jpg){:height="200px" width="300px"}

Another obvious fact: Lower the salary higher the rate of quitting. 

As a summary of these bar plots, I could say that promotions and high salaries 
could keep employees in the company whereas work accidents and type of jobs 
did not really matter.   

# Classification tree 

Noticing the pattern of clustering in my pair plots I thought a classification analysis
may give some deeper understanding of the data. The clustering looked somewhat simple 
patterned and therefore, tree analysis seemed suitable.

I splitted the data into training and testing sets and built a tree model.

{% highlight R %}
# Splitting into training and testing sets:  
rows <- nrow(hr_data)
ind <- sample(rows, rows * .67) 
train <- hr_data[ind, ]
test <- hr_data[-ind, ]
# Training:
tree <- rpart(left ~ ., data = train)
predicted <- predict(tree, test, type = "class")
{% endhighlight %}

{% highlight R %}
> # Confusion matrix 
> table(pred, test$left)
     
     pred    No  Yes
       No  3733  103
       Yes   41 1073
{% endhighlight %}
 
Not a bad confusion matrix. However at this moment I prioritized understanding 
the reason of quitting rather than the best prediction model which would yet 
come. I continued my analysis and plotted the tree.  

{% highlight R %}
rpart.plot(tree, type = 2, extra = 2)
{% endhighlight %}

![]({{ site.url }}/assets/hr_plot6.jpg){:height="600px" width="600px"}

My previously identified groups who left appeared clearer: 
1. Burnt-out group: satisfaction level < 0.11, number of projects > 2.5
2. Unmotivated group: satisfaction level < 0.46, projects < 2.5, last evaluation (0.45, 0.57) 
3. Mysterious group: satisfaction level >0.46, time spend in the company (4.5, 6.5), monthly hour > 214, last evaluation > 0.81 

# Mysterious group analysis
As the reason of quitting for the mysterious was unclear I decided to focus more analysis on this group.  
I filtered the data of the group and looked into the statistical summary.

{% highlight R %}
mysterious <- filter(hr_data, 
  satisfaction_level > 0.46, 
  time_spend_company >= 4.5, 
  last_evaluation >= 0.81,
  average_monthly_hours >= 214,
  time_spend_company < 6.5
  )
{% endhighlight %} 

{% highlight R %}
> summary(mysterious)
 satisfaction_level last_evaluation  number_project  average_monthly_hours
 Min.   :0.4800     Min.   :0.8100   Min.   :2.000   Min.   :215          
 1st Qu.:0.7600     1st Qu.:0.8700   1st Qu.:4.000   1st Qu.:233          
 Median :0.8200     Median :0.9200   Median :5.000   Median :246          
 Mean   :0.8106     Mean   :0.9226   Mean   :4.547   Mean   :247          
 3rd Qu.:0.8700     3rd Qu.:0.9800   3rd Qu.:5.000   3rd Qu.:260          
 Max.   :0.9800     Max.   :1.0000   Max.   :6.000   Max.   :307          
 time_spend_company work_accident  left     promotion_last_5years
 Min.   :5.000      No :890       No : 80   No :942              
 1st Qu.:5.000      Yes: 55       Yes:865   Yes:  3              
 Median :5.000                                                   
 Mean   :5.247                                                   
 3rd Qu.:5.000                                                   
 Max.   :6.000                                                   
 sales              salary         
 Length:945         Length:945        
 Class :character   Class :character  
 Mode  :character   Mode  :character
{% endhighlight %}

On average they worked on 4-5 projects and only 3 of them got promoted in last 5 years. 
Why were nearly anyone got no promotion should be investigated as it may give some clue.
 
Then I made distribution plots of their satisfaction, last evaluation, 
and average monthly hours.

{% highlight R %}

p3 <- ggplot(mysterious, aes(satisfaction_level)) +
  geom_density(aes(group = left, fill = left), alpha = 0.3) +
  labs(x = "Satisfaction Level", y = "", fill = "Left")
p4 <- ggplot(mysterious, aes(last_evaluation)) +
  geom_density(aes(group = left, fill = left), alpha = 0.3) +
  labs(x = "Last Evaluation", y = "", fill = "Left")
p5 <- ggplot(mysterious, aes(average_monthly_hours)) +
  geom_density(aes(group = left, fill = left), alpha = 0.3) +
  labs(x = "Average Monthly Hours", y = "", fill = "Left")
grid.arrange(p1, p2, p3, ncol = 3)
{% endhighlight %}

![]({{ site.url }}/assets/hr_plot7.jpg)

The ones who left had satisfaction levels quite precisely in between 0.7 and 0.95, 
which seemed unreal to me. It could be that the data was simulated...   

Furthermore, the number of projects and the salary distribution were checked by me. 

{% highlight R %}
p6 <- ggplot(mysterious) +
  geom_bar(aes(number_project, fill = left)) +
  labs(x = "Number of Projects", y = "", color = "Left")
p7 <- ggplot(mysterious) +
  geom_bar(aes(salary, fill = left)) +
  labs(x = "Salary", y = "", fill = "Left")
grid.arrange(p6, p7, ncol = 2)
{% endhighlight %}

![]({{ site.url }}/assets/hr_plot8.jpg){:height="300px" width="600px"}

Most employees in this group worked with more than 4 projects, but had either low 
or medium level salary. Is this normal? 

The suspicious condition lead me to run a few statistical check-ups. 
I made percentage distributions of the salaries for all employees, 
employees who spent the same length of time in the company  as the mysterious group,
and the same time length and the same number of projects:
{% highlight R %}
# Mysterious group
salary_myst <- mysterious %>% 
  group_by(salary) %>% 
  summarise(n = n()) %>% 
  mutate(percentage_myst = n / sum(n)) %>%
  select(-n)
# Company average
salary_avg <- hr_data %>% 
  group_by(salary) %>% 
  summarise(n = n()) %>% 
  mutate(percentage_avg = n / sum(n)) %>%
  select(-n)
# The same years spent in the company as the group
salary_time <- filter(hr_data, time_spend_company %in% c(5, 6)) %>%
  group_by(salary) %>% 
  summarise(n = n()) %>%
  mutate(percentage_time = n / sum(n)) %>%
  select(-n)
# The same years spent and works on the same number of projects
salary_time_proj <- 
  filter(hr_data, time_spend_company %in% c(5, 6), number_project %in% c(4, 5)) %>% 
  group_by(salary) %>% 
  summarise(n = n()) %>%
  mutate(percentage_time_proj = n / sum(n)) %>%
  select(-n)
{% endhighlight %}

{% highlight R %}
> right_join(right_join(right_join(salary_avg, salary_time), salary_time_proj), salary_myst)
# A tibble: 3 <U+00D7> 5
  salary percentage_avg percentage_time percentage_time_proj percentage_myst
     <chr>          <dbl>           <dbl>                <dbl>           <dbl>
     1   high     0.08247216      0.05522592           0.04810298      0.02010582
     2    low     0.48776585      0.51665906           0.53387534      0.58624339
     3 medium     0.42976198      0.42811502           0.41802168      0.39365079
{% endhighlight %}
The table showed that the mysterious group was underpaid comparing to the 
ones with the same years of experience in the company (we do not know 
employee's previous experience) and roughly the same amount of 
responsibility considering number of projects was a measure of certain responsibility. 
The group's salary distribution was even worse comparing to the companies average: 
Only 2 % of the group had high salary in contrast to 8 % as company's average.  

It was a bit hard to understand what exactly low, medium, and high salary meant and 
what the market salaries were for these people. 
However, it felt to me that such good employees could easily get a better 
salary promise from somewhere else. 
Moreover, I made a similar comparison in promotion facts and got the same results. 
Only, the promotion data was highly skewed, so may not be quite accurate to show in 
a table as I did for the salary.  
   
Anyway, my final check was the groups job title distribution. 

{% highlight R %}
ggplot(mysterious) +
  geom_bar(aes(sales, fill = left)) +
  labs(x = "", y = "", color = "Left") +
  coord_flip()
{% endhighlight %}

![]({{ site.url }}/assets/hr_plot9.jpg){:height="500px" width="500px"}

Not much I could really tell about the graph above. 
It seemed turnovers did not strongly depend on their departments.

I beleive this group of people deserved higher salary or a better condition. 
They could have got a better offer from somehwere else.

# Conclusions
There are three main groups of employees who quit: burnt-outs, wrong-fits (unmotivated), 
and possibly underpaid high-performers. The first group can be 
quickly noted by having very low satisfaction factor and many 
hours of working with high duties. The second group gets somewhat around 0.5 
by employer's evaluation, has less than 3 projects, and has low-medium satisfaction.
The third group employees are high performers who worked around 4-5 years and have 
satisfaction level between 0.7 and 0.95. 

Some conditions which turns off the employees are: 
- having spent less than 6 years in the company
- being responsible for more than 6 projects or working more than 280 hours a month
- being underpaid or not promoted 

I will show my predictions on their turnovers in my next post. 
   

[kaggle-url]: http://kaggle.com
[hr-url]: https://www.kaggle.com/ludobenistant/hr-analytics
