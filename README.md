## Can We Predict the Winner of a Professional League of Legends Match at 20 Minutes?

**Author:** Leadon Liao  
**Course:** DSC 80, University of California San Diego  
**Dataset:** Oracle's Elixir 2025 League of Legends Esports Match Data

## Introduction

League of Legends is a competitive team-based game in which five players work together to defeat the opposing team. Each player has a specific role, and understanding how these roles contribute to team success can provide insight into the factors that influence match outcomes.

In this project, we first investigate the contributions of two important roles: Attack Damage Carry (ADC), also known as Bot, and Jungle. We examine their gameplay statistics to investigate whether their contributions to team victories differ.

We then explore whether a professional League of Legends team's eventual match outcome can be predicted using statistics available at the 20-minute mark. By analyzing early and middle-game performance, we aim to understand how much information about the final outcome is already reflected in a team's performance before the match ends.

## Data Cleaning and Exploratory Data Analysis

Our dataset comes from Oracle's Elixir and contains professional League of Legends match data from 2025.

The original dataset contains 120,528 rows and 165 columns. Each match includes observations for individual player positions as well as team-level statistics. The six position categories are Top, Jungle, Middle, Bot, Support, and Team, with 20,088 observations in each category.

For our role comparison, we focused on Bot and Jungle players. We examined statistics including kills, deaths, assists, damage dealt to champions, earned gold, damage share, gold share, and vision score.

Our exploratory analysis revealed differences between the two roles. On average, Bot players dealt more champion damage and earned more gold, while Jungle players had higher average assists and vision scores. These differences reflect the distinct responsibilities of the two positions.

For our prediction task, we focused on team-level observations. We retained the match identifier, team side, match result, and statistics available at the 20-minute mark.

## Assessment of Missingness

We investigated missing values in the `firstbloodassist` column.

Among Blue-side Bot observations, 820 out of 10,044 values were missing, representing approximately 8.16%.

We examined whether missingness was associated with match result using a permutation test. The resulting p-value was approximately 0.971. Since this p-value is large, we found insufficient evidence that missingness in this column was associated with match result.

However, missingness was strongly associated with the `datacompleteness` column. The total variation distance between the missingness distributions across completeness categories was 1, with a permutation p-value of approximately 0.001.

These findings suggest that missingness in `firstbloodassist` is related to data completeness. In particular, incomplete match records may explain why some observations lack this information.

## Hypothesis Testing

We investigated whether Bot and Jungle players differ in their contributions to team victories, using damage share as our comparison metric.

Specifically, we compared the difference between winning and losing teams' damage-share advantages for the two roles.

Our hypotheses were:

- **Null Hypothesis:** The difference in damage-share advantage between winning and losing teams is approximately the same for Bot and Jungle players.
- **Alternative Hypothesis:** The difference in damage-share advantage between winning and losing teams differs between Bot and Jungle players.

We used a permutation test to evaluate the observed difference. The observed statistic was approximately -0.0000245, with a p-value of approximately 0.9857.

Since the p-value is much larger than our significance level of 0.05, we fail to reject the null hypothesis. We do not find sufficient statistical evidence that Bot and Jungle differ in their winning-versus-losing damage-share advantage.

This does not prove that the two roles contribute identically to team success. Rather, our analysis did not identify a statistically significant difference using this particular metric.

## Framing a Prediction Problem

We framed our prediction task as a binary classification problem.

Our goal is to predict whether a professional League of Legends team will eventually win or lose using information available at the 20-minute mark.

The target variable is `result`, where 1 represents a win and 0 represents a loss.

We selected team-level statistics available at 20 minutes, including gold, experience, creep score, their corresponding differences against the opposing team, and kills, assists, and deaths.

To prevent data leakage, we excluded statistics that would only become available after the 20-minute prediction point or after the match had ended.

We split the dataset into training and testing sets using match IDs as groups. This ensures that the two teams from the same match cannot appear in both the training and testing sets, preventing information from the same match from leaking between the two sets.

## Baseline Model

Our baseline model is a Logistic Regression classifier.

We used 12 quantitative features describing each team's resources and performance at 20 minutes. These features include gold, experience, creep score, their corresponding opponent statistics and differences, as well as kills, assists, and deaths.

We standardized the quantitative features using `StandardScaler` within a scikit-learn Pipeline. Since all selected predictors were quantitative, no categorical encoding was necessary.

We evaluated the model on a held-out test set separated by match ID.

The baseline model achieved the following results:

- **Accuracy:** 79.92%
- **Precision:** 79.96%
- **Recall:** 79.87%
- **F1-score:** 79.91%

The dataset is balanced between wins and losses, meaning that a classifier always predicting the majority class would achieve approximately 50% accuracy. Our baseline model performs substantially better than this benchmark, demonstrating that 20-minute team statistics contain useful information for predicting eventual match outcomes.

## Final Model

Our final model uses a Support Vector Machine (SVM) with an RBF kernel.

We selected SVM because its RBF kernel can capture nonlinear relationships between gameplay statistics and match outcomes. This allows the model to learn more complex decision boundaries than a standard linear classifier.

We used `GridSearchCV` with grouped five-fold cross-validation to tune the model's hyperparameters. We tested multiple values of the regularization parameter C and gamma, selecting the combination that achieved the highest cross-validation accuracy.

The best hyperparameters were:

- **Kernel:** RBF
- **C:** 0.1
- **Gamma:** 0.001

On the held-out test set, the model achieved:

- **Accuracy:** 80.03%
- **Precision:** 80.20%
- **Recall:** 79.76%
- **F1-score:** 79.98%

Compared with our Logistic Regression baseline, the SVM improved test accuracy by approximately 0.11 percentage points. Although this improvement is modest, the result suggests that the nonlinear decision boundaries learned by the RBF kernel provide a small predictive advantage for this dataset.

## Fairness Analysis

We investigated whether our final model performs differently for Blue-side and Red-side teams.

We selected precision as our evaluation metric. Precision measures the proportion of teams predicted to win that actually won.

Our hypotheses were:

- **Null Hypothesis:** The model has approximately equal precision for Blue-side and Red-side teams, and any observed difference is due to random chance.
- **Alternative Hypothesis:** The model has lower precision for Blue-side teams than for Red-side teams.

The observed precision was:

- **Blue-side precision:** 81.18%
- **Red-side precision:** 79.06%
- **Difference (Blue minus Red):** 2.12 percentage points

We performed a one-sided permutation test with 1,000 repetitions by randomly shuffling the Blue-side and Red-side labels while keeping the model predictions and actual outcomes fixed.

The resulting p-value was approximately 0.8791. Since this value is greater than our significance level of 0.05, we fail to reject the null hypothesis.

Therefore, we do not find statistically significant evidence that our model has lower precision for Blue-side teams than for Red-side teams. Although the observed precision was higher for Blue-side teams, this difference could plausibly be due to random variation.

This analysis does not prove that the model is perfectly fair. It only indicates that we did not find sufficient evidence of the particular performance disparity tested.
