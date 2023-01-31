# Case study 3: Major League Baseball

**Data input**

    datapay_wide <- read.csv("/Users/ruolanli/Downloads/MLPayData_Total.csv")
    datapay_long <- read.csv("/Users/ruolanli/Downloads/baseball.csv")

## 4.1 EDA: Relationship between payroll changes and performance

### (a)

    library(dplyr)

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    library(ggplot2)
    library(ggpubr)
    #plot variable payroll directly
    p1 <- datapay_long %>%
      ggplot(aes(x = payroll)) +
      geom_density()

    #plot log transformed payroll
    p2 <- datapay_long %>%
      ggplot(aes(x = log(payroll))) +
      geom_density()

    ggarrange(p1,p2)

![](STAT571_hw1_case3_files/figure-markdown_strict/compare%20difference%20and%20log%20difference-1.png)

 As the density plot shown, the variable payroll is right skewed. After
the log transformation, the density plot is much less skewed. Therefore,
it is more appropriate to use log difference.

### (b)

    #log transform difference
    datapay_long = datapay_long %>%
      group_by(team) %>%
      mutate(diff_log = log(payroll) - log(lag(payroll)))
    head(datapay_long)

    ## # A tibble: 6 × 6
    ## # Groups:   team [1]
    ##   team                  year payroll win_num win_pct diff_log
    ##   <chr>                <int>   <dbl>   <int>   <dbl>    <dbl>
    ## 1 Arizona Diamondbacks  1998    31.6      65   0.401 NA      
    ## 2 Arizona Diamondbacks  1999    70.5     100   0.617  0.802  
    ## 3 Arizona Diamondbacks  2000    81.0      85   0.525  0.139  
    ## 4 Arizona Diamondbacks  2001    81.2      92   0.568  0.00220
    ## 5 Arizona Diamondbacks  2002   103.       98   0.605  0.236  
    ## 6 Arizona Diamondbacks  2003    80.6      84   0.519 -0.243

### (c)

    datapay_long = datapay_long %>%
      select(team, year, diff_log, win_pct)
    head(datapay_long)

    ## # A tibble: 6 × 4
    ## # Groups:   team [1]
    ##   team                  year diff_log win_pct
    ##   <chr>                <int>    <dbl>   <dbl>
    ## 1 Arizona Diamondbacks  1998 NA         0.401
    ## 2 Arizona Diamondbacks  1999  0.802     0.617
    ## 3 Arizona Diamondbacks  2000  0.139     0.525
    ## 4 Arizona Diamondbacks  2001  0.00220   0.568
    ## 5 Arizona Diamondbacks  2002  0.236     0.605
    ## 6 Arizona Diamondbacks  2003 -0.243     0.519

## 4.2 Exploratory questions

### (a)

    payroll <- datapay_long %>%
      group_by(team) %>%
      filter(year %in% c(2010:2014)) %>%
      summarise(diff_sum = sum(diff_log)) %>% #log(2014)-log(2010) = sum of log(2014)-log(2013)+log(2013)-log(2012)...
      arrange(desc(diff_sum))
    head(payroll)

    ## # A tibble: 6 × 2
    ##   team                 diff_sum
    ##   <chr>                   <dbl>
    ## 1 Los Angeles Dodgers     0.851
    ## 2 Washington Nationals    0.820
    ## 3 San Diego Padres        0.744
    ## 4 Texas Rangers           0.684
    ## 5 San Francisco Giants    0.629
    ## 6 Toronto Blue Jays       0.493

 Los Angeles Dodgers, Washington Nationals, San Diego Padres, Texas
Rangers and San Francisco Giants had ighest increase in their payroll
between years 2010 and 2014.

### (b)

    win <- datapay_wide %>%
      rename("team" = names(.)[1]) %>%
      select(team, paste0("X", c(2014:2010))) %>%
      group_by(team) %>%
      mutate(pct = sum(c_across(X2014:X2010)) / X2010) %>% #calculate percentage between 2010 and 2014
      arrange(desc(pct))
    head(win)

    ## # A tibble: 6 × 7
    ## # Groups:   team [6]
    ##   team                 X2014 X2013 X2012 X2011 X2010   pct
    ##   <chr>                <int> <int> <int> <int> <int> <dbl>
    ## 1 Pittsburgh Pirates      88    94    79    72    57  6.84
    ## 2 Washington Nationals    96    86    98    80    69  6.22
    ## 3 Baltimore Orioles       96    85    93    69    66  6.20
    ## 4 Arizona Diamondbacks    64    81    81    94    65  5.92
    ## 5 Seattle Mariners        87    71    75    67    61  5.92
    ## 6 Kansas City Royals      89    86    72    71    67  5.75

 Pittsburgh Pirates had the biggest percentage gain in wins between 2010
and 2014.

## 4.3 Do log increases in payroll imply better performance?

    #scatterplot
    datapay_agg <- payroll %>%
      inner_join(win, by = c("team" = "team")) %>%
      select(team,diff_sum,pct)
    datapay_agg

    ## # A tibble: 30 × 3
    ##    team                  diff_sum   pct
    ##    <chr>                    <dbl> <dbl>
    ##  1 Los Angeles Dodgers      0.851  5.42
    ##  2 Washington Nationals     0.820  6.22
    ##  3 San Diego Padres         0.744  4.33
    ##  4 Texas Rangers            0.684  4.86
    ##  5 San Francisco Giants     0.629  4.74
    ##  6 Toronto Blue Jays        0.493  4.66
    ##  7 Pittsburgh Pirates       0.472  6.84
    ##  8 Baltimore Orioles        0.470  6.20
    ##  9 Philadelphia Phillies    0.466  4.39
    ## 10 Cincinnati Reds          0.460  4.76
    ## # … with 20 more rows

    datapay_agg %>%
      ggplot(aes(x = diff_sum, y = pct)) +
      geom_point(color = "blue", size = 3) +
      geom_text(aes(label = team), size = 3) +
      labs(title = "Win Percentage vs. Log Increase in Payroll",
           x = "Log_increase_in_payroll",
           y = "Win_pct")

![](STAT571_hw1_case3_files/figure-markdown_strict/test%20higher%20increases%20in%20payroll%20lead%20to%20increased%20performance-1.png)

    #Least Squared Lines
    datapay_agg %>%
      ggplot(aes(x = diff_sum, y = pct))+
      geom_point(size = 3)+
      geom_smooth(method = "lm", formula = y~x, color = "blue")+
      labs(title = "Win Percentage vs. Log Increase in Payroll",
           x = "Log_increase_in_payroll",
           y = "Win_pct")+
      theme_bw()

![](STAT571_hw1_case3_files/figure-markdown_strict/unnamed-chunk-1-1.png)

    #linear relationship
    fit = lm(pct~diff_sum, data = datapay_agg)
    summary(fit)

    ## 
    ## Call:
    ## lm(formula = pct ~ diff_sum, data = datapay_agg)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -1.26091 -0.47951 -0.07277  0.36990  1.64956 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)   4.9257     0.1554  31.694   <2e-16 ***
    ## diff_sum      0.5658     0.3476   1.628    0.115    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.6916 on 28 degrees of freedom
    ## Multiple R-squared:  0.08645,    Adjusted R-squared:  0.05382 
    ## F-statistic:  2.65 on 1 and 28 DF,  p-value: 0.1148

    #correaltion
    cor(datapay_agg$diff_sum, datapay_agg$pct)

    ## [1] 0.2940234

 No, there is no strong evidence to support the hypothesis that higher
increases in payroll on the log scale lead to increased performance.

 In the scatter plot, the data points do not cluster tightly.

 R-squared in the linear model shows the linear regression model did not
fit the data well, indicating no strong linear relationship between log
increases in payroll and win percentage.

 The correlation coefficient is 0.294, indicating a low correlation.

## 4.4 Comparison
