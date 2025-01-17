# Abstract
This project leveraged advanced supervised learning techniques combined with optimization algorithms to create a useful metric for evaluating player performance. Using these techniques, we developed a performance metric aimed at maximizing wins, which was subsequently used to optimize basketball team rosters through the CVXPY optimization library. Our approach provides a systematic way to evaluate players and ensures optimal team compositions based on various constraints such as budget and positional needs.

The methodologies we developed have significant business implications, particularly in enhancing team performance, making informed decisions during free agency, and optimizing draft picks. The flexibility of our model allows it to be applied across different sports, including the WNBA, once relevant data becomes accessible. This adaptability ensures that sports teams can consistently make data-driven decisions to maximize their chances of success.

Additionally, our optimization model is designed to be highly adaptable, capable of incorporating new data sources and evolving with the needs of the sports industry. By focusing on maximizing wins, our metric ensures that every player selection is geared towards achieving the highest possible performance on the court. The potential applications of this model extend beyond basketball, providing valuable insights and optimization strategies for any team sport where player performance and team composition are critical factors.

# Introduction
The landscape of professional sports is continuously evolving, with leagues frequently exploring opportunities for expansion to new markets. This growth necessitates the need for expansion teams to construct competitive rosters quickly. One of the critical challenges these teams face is evaluating a large pool of available players to identify those who can contribute most effectively to the team's success.

In this context, having a reliable and comprehensive metric for player evaluation becomes key. Traditional methods of player evaluation often rely on subjective assessments and basic statistics, which can lead to inconsistent and suboptimal decision-making. Advanced metrics that provide a more nuanced and holistic view of player performance are crucial for teams looking to build a competitive edge. By developing a comprehensive metric that quantifies player performance based on their contribution to wins, teams can make more informed and strategic decisions.

Our project addresses this need by combining supervised learning techniques with optimization algorithms to develop a performance evaluation metric. This metric is designed to provide a comprehensive view of a player's impact on the game, factoring in various performance indicators and their relative importance in contributing to wins.

Teams that adopt such advanced metrics can achieve several advantages such as:
- Enhanced decision-making: By using data-driven insights, teams can identify undervalued players who may not be recognized through traditional scouting methods.
- Optimized rosters: The optimization model ensures that teams are composed of players who collectively maximize the chances of winning while also adhering to constraints like salary caps and positional requirements.
- Strategic planning: With a reliable performance metric, teams can plan more effectively for free agency, drafts, and trades, making strategic moves that align with their long-term goals.
- Increased competitiveness: Expansion teams can quickly become competitive, reducing the typical lag period associated with new teams in a league.

Moreover, the ability to simulate various scenarios and constraints allows teams to explore different strategies and choose the one that best fits their objectives and resources. This strategic flexibility is invaluable in a dynamic and fast-paced sports environment.

While this project focused on basketball, the principles and methodologies developed are highly adaptable. The core idea of using performance metrics to drive optimization can be applied to any sport, making this approach universally valuable. As the WNBA and other sports leagues increasingly embrace data analytics, our model stands ready to provide actionable insights and competitive advantages.

Optimization models in sports are not just about selecting the best players; it also involves managing team finances, predicting player development, and planning for future seasons. By integrating these factors, our model offers a comprehensive solution that can increase the success of how teams are built and managed.

The integration of supervised learning and optimization techniques in evaluating and selecting players represents a significant advancement in sports management. By adopting these methods, teams can ensure that their rosters are not only competitive but also strategically aligned to achieve maximum success. The future of sports analytics lies in leveraging such sophisticated models to make data-driven decisions, ultimately leading to better performance and greater success on the field or court.

# Methods
Our primary objective is to identify the metrics that most significantly contribute to wins. To achieve this, we developed a model to predict a team's total wins and then mapped these metrics back to individual player performance to determine how players contribute to team success.

We began our process by obtaining two identical datasets of NBA box scores per 100 possessions from the 2003-2024 seasons: one dataset for each team’s home statistics and their opponent’s statistics against them at home, and the other for each team’s away statistics and their opponent’s statistics against them while playing on the road. Each row contains a unique identifier, "TeamSeason," which is a string concatenating the season and the team name (e.g., '2022Lakers'), and that team’s statistics per 100 possessions for that season. Since each team plays 41 games at home and 41 away, there are 41 games worth of statistics aggregated into each row (Refer to Appendix 7.1). Box score data was scraped from Game Summaries from nba.com, using the hoopR R package.

Our initial step in data cleaning was to separate the 'TeamSeason' column into two distinct columns for 'team' and 'season.' We then merged the 'home' and 'away' datasets on 'team' and 'season' and aggregated each team’s away and home statistics so that each row corresponds to a full 82-game season's worth of performance. RAPM data was collected from the nbashotcharts.com, and every team’s O-RAPM, D-RAPM, and Overall RAPM, were the product of a players minutes and their RAPM, done for entire rosters and summed together. We then modified the 'team' column to match the format of our master dataframe, and then joined it with our master dataframe on 'team' and 'season' to ensure that each team had impact data corresponding to each season’s statistics. After merging, our master dataset included only the seasons from 2011-2022 (Refer to Appendix 7.2).

We discovered a time series component in our data, as a team’s total wins in the previous season (t-1) is a significant predictor for their wins in the current season (t) (Refer to Appendix 7.12). To address this autocorrelation, we introduced a fixed effects model by creating dummy variables for each team and each season, which significantly increased the dimensionality of our dataset, and included two lags of total wins. Before running any models, our dataset contained 384 rows and 146 columns.

To avoid the autocorrelation issues that a random train-test split can cause with time series data, we assigned all seasons before 2020 to the training set and tested our models on the 2020-2022 seasons. The first model we ran was a standard linear regression without any variable selection, which yielded poor results as expected due to the high dimensionality and complexity of our dataset (Refer to Appendix 7.3). Next, we applied regularized regression techniques in hopes that the regularization parameter would perform variable selection and improve overall model performance. After running Ridge, LASSO, and ElasticNet regression models and performing five-fold cross-validation, LASSO performed the best on the test set with an MSE of 15.25 and an R² of 88.04%. Ridge was the second best, with an MSE of 21.59 and an R² of 80.08%, followed by ElasticNet with an MSE of 34.59 and an R² of 72.89% (Refer to Appendix 7.4). The relative similarity in our results across the three regularized regression models confirmed the predictive abilities of our metrics and positioned us to map the coefficients of our models to individual player statistics. However, with more rows of data, we could achieve better performance and more similarity across our different regression models.

Next, we aimed to map our model coefficients to individual player statistics to identify which players possess the most winning skill sets. We obtained a dataset of 6,563 rows containing individual player statistics from the 2010-2022 seasons (Refer to Appendix 7.5). To map the coefficients from our models to the player dataset, we needed to ensure the dimensions of the player dataset matched those of the dataset used in our models. We normalized the data using StandardScaler to ensure that the player statistics were on the same scale as the data in our models.

Non-predictive columns were dropped, and players who played fewer than 30 games in a season were filtered out. Additionally, we needed to impute data that was present in our team dataset but unavailable in our player dataset. For columns that existed in the master dataset but not in the player dataset, we imputed zeroes. Since home and away data was not available for individual player statistics, we assumed that a player's home statistics were equal to their performance on the road and imputed the data accordingly (Refer to Appendix 7.6).

Finally, we computed the coefficients for players by taking the dot product of the array of coefficients from our models with the new player dataset. This resulted in each player having a coefficient value for each of the three models. A positive coefficient indicated that the player had a positive impact on wins, while a negative coefficient indicated a detrimental effect. To ensure our coefficients were interpretable and standardized, we scaled them so that the average coefficient was 100 (Refer to Appendix 7.7).

# Optimization
With our player coefficients calculated, the next step was to construct optimal rosters using CVXPY. The objective was to maximize the sum of player coefficients while adhering to constraints like positional requirements and salary caps.

The optimization problem can be formulated as follows:
- **Objective**: Maximize the sum of player coefficients.
- **Constraints**:
  - Positional requirements: Ensure that the roster includes players for each required position (e.g., at least 2 guards, 2 forwards, and 1 center).
  - Salary cap: Ensure that the total salary of the selected players does not exceed the budget constraints.

We ran the optimization model for the 2023-24 season using statistics from the 2022-2023 season and the salaries for that season. The resulting optimal rosters are shown in Appendices 7.8-7.10, revealing practical 10-man NBA rosters composed of the players who provide the highest contributions to wins according to our metric. Notably, the optimized rosters included many top players, validating the effectiveness of our metric. When we introduced salary constraints, the resulting rosters still comprised many high-performing players, demonstrating the practical applicability of our approach.

# Results
Our LASSO regression model was the best-performing model and served as the foundation for our player coefficient calculations and subsequent optimization steps. Our model demonstrated that winning skill sets are correlated with players who can shoot efficiently, defend well, and contribute positively to team chemistry.

The optimization model generated rosters that included many well-known top players, confirming the validity of our approach. Additionally, the model provided insights into undervalued players who might be overlooked using traditional evaluation methods. These insights can be particularly valuable for teams with limited budgets, enabling them to identify players who can provide significant contributions at a lower cost.

# Discussion
The results from our project provide a robust framework for evaluating player performance and optimizing team rosters. Our approach combines the predictive power of supervised learning with the strategic benefits of optimization, resulting in a metric that is both comprehensive and actionable.

Our metric allows teams to:
- Identify key contributors to wins and focus on acquiring or retaining these players.
- Optimize rosters under various constraints, ensuring that teams are both competitive and financially sustainable.
- Make data-driven decisions during free agency, drafts, and trades, leading to better long-term outcomes.

Future work could involve integrating more granular financial data, such as contract structures and incentives, to provide even more precise optimization recommendations. Additionally, incorporating injury data and other context-specific factors could further enhance the model's predictive power and practical applicability.

# Conclusion
The integration of supervised learning and optimization techniques represents a significant advancement in sports management. By adopting these methods, teams can ensure that their rosters are not only competitive but also strategically aligned to achieve maximum success. The future of sports analytics lies in leveraging such sophisticated models to make data-driven decisions, ultimately leading to better performance and greater success on the field or court.

---

