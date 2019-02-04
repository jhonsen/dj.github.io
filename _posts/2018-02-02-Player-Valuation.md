---
layout: post
title: NFL-Players Valuation
---



# Predicting NFL Player Salary Based on Early Career Perfomance    

#### Introduction

This project (codename: *Luther*) was completed during my third week at Metis [data science bootcamp](https://www.thisismetis.com/data-science-bootcamps). Based on the curriculum, the focus of this project was on web-scraping and building linear regression models. Unlike the first project, this one was completed individually, i.e., I had the opportunity to determine  the scope of the project and to design the best way to collect-, clean-, and analyze the data. To this end, I decided to focus my work on the world of NFL analytics. 

Having recently moved to Seattle (for the bootcamp), it was the perfect time for me to see the lives of the Seattelites through the eyes of their beloved football team, the Seahawks. This team has demonstrated their prowess in recent years, as they were led to Super Bowl victory (in 2013) by their quarterback, Russell Wilson. Wilson's *rise-to-fame* story is a perfect example of the [*moneyball*](https://en.wikipedia.org/wiki/Moneyball_(film)) success. As the 75th-pick overall (3rd round), Wilson was drafted by the Seahawks and signed to a 4-year contract deal that was worth [$2.9 million](https://www.spotrac.com/nfl/seattle-seahawks/russell-wilson-9885/) . With less than 1 million annual salary, he took the Seahawks to the Super Bowl twice, brought home the Lombardi once, and broke several franchise [records](https://en.wikipedia.org/wiki/Russell_Wilson). During this period, many (including myself) contended that Russell Wilson deserved more pay. However, was there any data that would support his fans intuition about a salary hike? In other words,  *was Russell Wilson really underpaid during his early NFL career?* 

In my attempt to answer this question, I wanted to take a data-driven approach of player valuation, with respect to their early NFL career performance

#### Project Description and Approach

Each football player that gets drafted into the NFL typically gets a 4-year contract deal (1st-round players get a [5th year option](https://www.sbnation.com/nfl/2018/4/30/17171726/nfl-rookie-wage-scale-draft)). Under this contract, a player is unable to renegotiate his salary, even if he excels beyond the franchise's expectations. Given a set of performance measures, what is the *"true"* value ($$$) of a player, e.g., on their 4th year?

#### Data Acquisition

The scope of my project was limited to 3 types of offensive positions: quarterbacks (QB), running backs (RB), and wide receivers (WR). My approach to assess players' early career was to collect data that describe their (1) draft worthiness, (2) their first four years of performance, (3) and their salary on the 4th year. These were all webscraped using `beautifulsoup` and `selenium` from [pro-football-reference](http://pro-football-reference.com/) and [spotrac](https://www.spotrac.com/nfl/rankings/2003/base/). The gruesome details of webscraping and data preprocessing are described elsewhere, and the codes are available in my [repo](https://github.com/jhonsen/NFLplayersValuation). Briefly,

1. *Draft worthiness* - contains players' information and physical test results in the NFL combine. Specifically, this dataset includes players' name, draft status, draft round, weight, height, and 40-yard dash record (other results were excluded due to  missing values). Pro-football-reference.com only keeps combine records for rookies entering the league in year 2000 and later. So my analysis was limited to only those players. 

2. *Four years of performance measures*  - represent position-specific yardage (Yds) and touchdowns (TD), accumulated by each player within the first 4 years of his career.  Web-scraping these datasets turned out to be quite involved, because the metrics are position-dependent. For instance, the total passing Yds and -TD for the year 2012 (1st-), 2013 (2nd-), 2014 (3rd-), and 2015 (4th-year) were collected for a rookie QB that entered the NFL in 2012 (*see* **Figure 1**). This acquisition process was applied to each of the 3 different positions. Namely, passing, rushing and receiving Yds (or TD) were extracted for QB, RB, and WR, respectively. 

   ![Fig.1]({{site.url}}/images/concat1.png)

   **Figure 1**. Workflow of data acquisition. Inset illustrates data collection required for a single class of rookie QB entering the NFL in 2012. 

3. The *4th year* base salary of each player - was used as the *target* for my model (*see* **Figure 1**). I chose to use this 4th-year information, so that I could limit my analysis to "active" players (or still playing in the league at their 4th-year) like Russell Wilson. So, I collected players' salary over the last 15 years (2003-2018). Additionally, I adjusted each year's salary with the annual inflation [rate](https://www.usinflationcalculator.com/inflation/historical-inflation-rates/), to reflect a value that is equivalent to 2018.

   

#### Data Wrangling and Feature Engineering

With the workflow described above, I ended up collecting 45 dataframes (i.e., 15 per position), representing 15 years of performance data. At this point, I couldn't immediately combine all of these dataframes into one, as these stats correspond to different positions. For instance, rushing Yards are not identical to passing Yds nor receiving Yds. So, I decided to create a metric that is relevant to different positions. In this case, I chose to use a standard Yahoo fantasy football point system to reflect every player's annual performance:

- For RB,  1 point for every 10 rushing yards, and 6 points for every TD
- For WR, 1 point for every 10 receiving yards, and 6 points for every TD
- For QB, 1 point for every 10 passing yards, and 4 points for every TD

After transforming players' Yds & TD into points, I joined the 45 dataframes into a single workable table. Then, categorical features (such as *position* and *draft status*) were one-hot encoded, and one of each of these category-columns was dropped to avoid the dummy variable trap. Some features were also tranformed using the log1p or box-cox or Yeo-Johnson method. Many of these features were largely skewed to the right, as rookie players often make little impact in their first 4 years of NFL career.



#### Regression Models and Analysis

After cleaning my dataset and filtering out "non-active" players (i.e., those who don't get paid on the 4th year), I ended up with 356 rows of players and 11 features. (Btw, it's quite remarkable to see that almost 70% of new players entering the NFL don't last 4 years). The 11 features I included in the model are draft status, draft round, weight, height, 40-yard dash, 4 years of performance points (i.e., Year1, Year2, Year3, and Year4), and positions (RB or WR) (**Figure 2**). 

![Fig.2]({{site.url}}/images/dataframesample.png)

**Figure 2.** A sample observation from the dataframe containing 11 features. Player names are excluded in the regression models. 

Prior to building a regression model, I split the dataframe into training (80%) and testing (20%) sets. With the training set, I built a pipeline that performs feature scaling and 10-fold CV on  `LinearRegression`, `Ridge`, `Lasso`, `ElasticNet`. (Regularization parameters for Ridge, Lasso and ElasticNet were obtained using  `LassoCV`, `RidgeCV`, and `ElasticNetCV`). As a first stab of model selection, I also included tree-based regressors:`RandomForestRegressor`, `DecisionTreeRegressor`, and`BaggingRegressor`, *out-of the-box* (or without any hyperparametric tuning).

The result of 10-fold CV on each of the regressors was not as optimal as I had anticipated. Linear regression models show comparable results (**Figure 3**) with *root-mean-squared error* (RMSE) of ~0.61, which is lower than tree-based methods (RMSE~0.64-0.85). Also, the r-squared values for linear regressors were ~0.21, compared to the ~0.12 r-squared of tree-based regressors, suggesting better predictive power.   

![Fig.3]({{site.url}}/images/Alg_comparison_RMSE.png)

**Figure 3**. Model-performance on the training set, based on 10-fold CV. Jittered points reflect errors on each fold.

Despite the less-than-optimal performance, I decided to use the linear regression model, without regularizations, to predict players' salary on the test set. I chose this simple model for its interpretability, which would be most useful for NFL managers as they assess their players.  

The linear regression model is fairly effective for predicting players' salaries (**Figure 4**). The prediction (orange line) seems to capture the bulk distribution of players' salary ranging from 0 to ~2 million USD. However, it shows weaker predictive power for higher-income players receiving ~4-8 million USD. The model's tendency to predict lower salary is most likely influenced by the skewed distributions observed in the *features*. Even though various transformations were applied, many of those *features*  had outliers at or near zeros (data not shown). Overall, the curent model has an (RMSE) error of ~1 million USD, which is fairly substantial for (rookie) players.  

![Fig.4]({{site.url}}/images/ytest_ypred_histo_kde.png) 

**Figure 4**. Predicted and true target values in the test set. Predicted- and true salary are illustrated using kernel density estimation, which estimates the probability density function. The horizontal (x-axis) represents salary distribution in the test set, whereas vertical (y-axis) shows the probability density of a given salary.      

Once I trained my linear regressor on the entire training set, I then insert Russell Wilson's stats into my model, to obtain his predicted salary. The result suggests that an NFL player with his performances stats should receive ~2.6 million USD. This value is way higher than Wilson's actual pay (<1 million/year). Even when the prediction error (~1 million) is considered, Wilson's value should be around ~1.6 million USD. So, I would contend that (indeed) the Seahawks got a very good deal by signing Wilson with a 4-year contract for only ~3 million USD. In other words, given his performance, *Russell Wilson could (should) have been paid more than he actually got*.   

 

#### Summary and Future Work

As it turns out, this classic "moneyball" problem (though successful in MLB) may not be trivial for NFL. At the moment, my model only considers features that are related directly to the individuals. However, I believe there are other factors that would affect the value of an NFL player and, consequently, provide higher predictive power. To this end, much feature engineering work could be done to account for those variables. For example, I would have wanted to include variables that describe a player's opportunity to play in a game (e.g., playing time per game, injured players in the depth chart, etc.). 

In addition, I would have wanted to use a bigger dataset to train my model. At the moment, my primary source of data is pro-football-reference.com, which only keeps NFL-Combine stats for players entering the NFL from year 2000 and later. Combining RB, WR, and QB players into a single table only results in 356 total observations. (This concatenation not be the best approach, as each of these players have different roles and contibutions within a given team). Ideally, it may be best to create regression models that are position-dependent. Nevertheless, the current *less-than-ideal* scenario is presented, because the 4-year-survival rate of QB, RB, and WR in the NFL is pretty low (~30-40%). In other words, not very many players entering the NFL actually last 4 years and/or longer, and the challenge of building a robust model from limited number of observations remains difficult to complete.         

I feel like I have learned so much from doing a data science project that dives into the world of NFL analytics. Through this process, I learned how to web-scrape data from multiple sources, build simple predictive models, and extract informative insights from them. I can't wait to work on my next project, so stay tuned for my next installment. 

Thanks for reading. A more detailed explanation of my codes is available on my [repo](https://github.com/jhonsen/NFLplayersValuation). Any helpful comments and suggestions are appreciated.  



**Attribution**

This project was inspired by the work of [Ka Hou Sio](https://medium.com/@kahousio/project-luther-predicting-nba-player-salary-from-their-performance-b8209323c72d) and [Jason SA]( https://github.com/jason-sa/baseball_lin_regression), who investigated  the valuation of NBA and MLB players at METIS.

