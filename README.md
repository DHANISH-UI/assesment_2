1. Data Preparation

Weekly seasonality & trend

Converted week column to a datetime and derived week number, month, quarter, and year.

These temporal features allow the model to capture seasonal patterns and long-term growth/decline.

Lagged outcomes

Added revenue_lag1 and revenue_lag2 to model carryover effects and reduce leakage.

Added followers_lag1 and followers_growth to track momentum in organic social signals.

Transformations & scaling

Applied log1p transform to spend and revenue variables to handle skew, reduce heteroscedasticity, and normalize variance.

Scaled continuous features (average_price, followers_growth) with StandardScaler for comparability.

Zero-spend periods

The log transform (log1p) ensures zero spend is handled gracefully (log(0+1)=0).

Promotion features

Encoded promotions as categorical (promotions_cat).

If intensity levels exist, added a numeric promotions_intensity.

2. Modeling Approach

Two-stage pipeline (mediator-aware)

Stage 1: Predict Google Spend as a function of other paid social channels (Facebook, TikTok, Instagram, Snapchat).

Stage 2: Predict Revenue (log) using predicted Google spend, promotions, price, email/SMS activity, and follower dynamics.

Model choice: Random Forest Regressor

Captures nonlinearities and interactions without strong parametric assumptions.

Handles noisy marketing signals and avoids overfitting small seasonal fluctuations.

Hyperparameters

Stage 1: default n_estimators=100 for stability.

Stage 2: tuned depth, min samples split, and leaf size to regularize and prevent overfitting.

Feature selection

Top drivers identified via feature importances and partial dependence plots (PDPs).

Validation plan

Used time-based split (80/20): training on earlier weeks, testing on later weeks.

This respects temporal ordering and avoids data leakage.

3. Causal Framing

Mediator assumption

Structured the pipeline so Google Spend is predicted first, preventing direct leakage of actual Google spend into the revenue model.

This respects the causal pathway: Social → Google → Revenue.

Back-door paths

By separating Stage 1 and Stage 2, the model blocks confounding from direct inclusion of correlated spend variables.

Lagged outcomes (e.g., revenue_lag) further mitigate temporal confounding.

Leakage control

All transformations and splits respect time ordering.

No information from the future is used to predict the past.

4. Diagnostics

Out-of-sample performance

Reported RMSE and R² for both Stage 1 (Google Spend) and Stage 2 (Revenue).

Stage 2 tested on unseen weeks to measure generalization.

Stability checks

The 80/20 split simulates a rolling forecast.

(Future extension: blocked/rolling CV across multiple time windows).

Residual analysis

Scatter plots of predicted vs actual highlight fit quality.

Deviations indicate weeks where promotions or price shocks dominate.

Sensitivity analysis

PDPs illustrate marginal effects of Average Price, Promotions, and Google Spend on revenue.

Helps assess robustness of causal interpretation.

5. Insights & Recommendations

Key drivers

Social media spend (via Google mediation) strongly influences revenue.

Promotions and Average Price are significant modifiers.

Organic signals (followers growth) add incremental lift.

Risks & caveats

Collinearity between channels may inflate importances.

Mediated effects are assumed to flow only through Google Spend; in reality, direct effects may exist.

Overfitting risk minimized via shallow trees and validation, but longer-term rolling CV is recommended.

Business recommendation

Balance paid social investments with Google Ads efficiency since much of social’s effect is mediated.

Monitor price sensitivity and promotion strategies since both show strong nonlinear impacts.

Continue tracking organic growth, as it compounds paid efficiency.
