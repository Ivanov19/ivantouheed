# ML Fundamentals Series — Posts 2 through 8

---

## POST 2 — Your Data Is the Foundation. And It's Usually a Mess.
*Post 2 of 8 — ML Fundamentals*

Before a single model is trained, before a single parameter is tuned, there is a step that determines whether everything after it is meaningful or not. That step is understanding your data.

This sounds obvious. It rarely gets the attention it deserves.

---

**What a dataset actually is**

A dataset in ML is a table. Rows are observations — individual examples of the thing you're studying. Columns are features — the properties you've measured or recorded about each observation. One column is your target: the thing you want the model to predict.

A clean dataset looks like this: every row is complete, every column means one thing, units are consistent, and there are no duplicates. In practice, that almost never exists.

What you actually get is a table where some rows are missing several values, some columns have inconsistent formatting, units occasionally switch halfway through, and a handful of rows are clearly erroneous but not obviously wrong. This is real-world data, and it's normal.

---

**Missing values: the first thing to look at**

The first diagnostic to run on any dataset is a missingness map — a visual check of which columns have gaps and how much.

📊 *[Suggested image: a missingness heatmap — dark cells where data exists, white/yellow where it's absent. Search: "missingno heatmap machine learning missing data visualization"]*

Missing data is not automatically a problem. The question is: *why* is it missing?

There are three distinct situations:

*Missing completely at random* — the gap has no relationship to anything else in the data. A sensor glitched. A field was left blank by accident. This is the benign case.

*Missing at random (conditionally)* — the gap is related to another column you *do* have. Papers from certain time periods didn't report a particular measurement; newer papers did. The missingness is predictable from context.

*Missing not at random* — the gap is related to the value itself. Extreme results often go unreported. This is the hard case, and naive imputation can introduce systematic bias.

The approach you take to fill gaps — or whether to fill them at all — depends entirely on which situation you're in. Filling everything with a column average regardless of why it's missing is technically easy and analytically wrong in most real datasets.

---

**Skewed distributions and why they matter**

Plot your target variable before doing anything else. A histogram takes thirty seconds and can change your entire strategy.

If the distribution is heavily skewed — most values clustered at the low end with a long tail of high values — linear models and many distance-based methods will perform poorly. The math they rely on assumes the target behaves roughly like a bell curve. When it doesn't, the model will systematically underperform on the extreme values, which are often the ones you care most about.

The standard fix is a log transform: replace the raw value with its logarithm before training. This compresses the tail, makes the distribution more symmetric, and stabilizes model performance across the full range. You reverse the transform when interpreting predictions.

---

**Redundant features**

Two features that are perfectly correlated carry the same information twice. In a linear model, this creates numerical instability. In tree models, it dilutes feature importance. Either way, it's noise you can remove.

A correlation matrix — a grid showing how strongly every numeric column correlates with every other — tells you where this is happening. Any pair with a correlation above 0.9 warrants a decision: keep one, drop one, or combine them into a single engineered feature.

---

**The principle that carries through the rest of this series**

Every hour spent understanding your data before modeling is worth more than an hour spent tuning a model on data you don't understand. This isn't a philosophical point — it's an empirical one. The datasets that produce reliable models are almost always ones where someone sat with the data long enough to know where the problems are before training started.

---

*Next: Post 3 — Preprocessing: turning raw data into something a model can actually use.*

*#MachineLearning #DataScience #MLFundamentals #DataCleaning*

---
---

## POST 3 — Preprocessing: What Happens Before the Model Sees Your Data
*Post 3 of 8 — ML Fundamentals*

You have your data. You've looked at it, understood its shape, noted where the gaps are. Now you need to convert it into a form a model can actually process.

This stage is called preprocessing. It's unglamorous work. It's also where many projects either build a solid foundation or quietly introduce errors that corrupt everything downstream.

---

**Why models can't just take raw data**

Most ML algorithms operate on numbers. They compute distances, dot products, weighted sums. If your data contains text categories ("Hydrothermal", "Sol-gel", "Combustion"), the model cannot work with them directly — you have to encode them numerically. If your numeric columns span wildly different scales (one column ranges from 0 to 1, another from 0 to 50,000), models that rely on distance calculations will treat the large-scale column as far more important simply because its numbers are bigger, regardless of whether that's physically meaningful.

Preprocessing is the set of transformations that fixes these issues.

---

**Encoding categorical variables**

The most common approach is one-hot encoding: create a new binary column for each category, set it to 1 if the row belongs to that category, 0 otherwise.

If your synthesis_method column has 5 categories, it becomes 5 binary columns. The model sees numbers; you've preserved the categorical structure.

The tradeoff: one-hot encoding expands your dataset width. With high-cardinality columns — a composition formula with 200 unique values, for instance — one-hot encoding creates 200 columns, most of which are zero for any given row. This is called sparse representation, and it can harm model performance. Alternative approaches (target encoding, embedding) exist for these cases.

---

**Scaling numeric features**

When you scale numeric features, you're putting them on a comparable footing. Two common methods:

*Standardisation* — subtract the mean, divide by standard deviation. Each column now has mean 0 and std 1. Linear models, SVR, and neural networks benefit significantly from this.

*Min-max normalisation* — rescale each column to the range [0, 1]. Useful when you want to preserve the shape of the distribution exactly.

Tree-based models (Random Forest, XGBoost) do not need scaling — they make decisions based on rank ordering of values, not magnitude. Applying scaling to them won't hurt, but it won't help either.

---

**Imputing missing values**

Three approaches, in increasing sophistication:

*Drop the row* — if a row is missing too many values to be useful, remove it. Works when missingness is rare and random.

*Simple imputation* — replace missing values with the column mean (for numeric) or the most common value (for categorical). Fast, easy, and wrong when the missingness is systematic rather than random.

*Model-based imputation (IterativeImputer)* — train a regression model to predict the missing value from the other columns. Works column by column, iteratively, until predictions stabilize. More computation, far more defensible when missingness correlates with other features.

One addition that's worth the five lines of code: after imputing, add a binary "was this value missing?" indicator column. If the absence of a measurement is itself informative — and in literature-scraped data, it often is — the model can learn that signal too.

---

**Feature engineering**

Preprocessing isn't just cleaning what you have. It's sometimes creating new features from existing ones that better represent the underlying structure.

Simple examples: a ratio of two columns that together drive the target more than either does alone. A log transform of a skewed input feature. A binary flag that indicates whether a material belongs to a certain class.

The best feature engineering is grounded in domain knowledge — understanding *why* two quantities relate, not just that they correlate in your training set. A derived feature with a physical justification generalizes. One that happens to correlate in your particular dataset often doesn't.

---

**The critical rule: fit on training data only**

Every parameter of your preprocessing pipeline — the mean you subtract during scaling, the percentile boundaries you use for normalisation, the predicted values in IterativeImputer — must be calculated from the training data only, then applied to the test set.

If you compute the column mean from the entire dataset before splitting, your test set has already influenced the pipeline. The scores you get are not honest. This is a form of data leakage, and it's surprisingly common.

Sklearn's Pipeline object exists precisely to enforce this. If you're not using it, you should be.

---

*Next: Post 4 — The most important concept nobody teaches first: validation.*

*#MachineLearning #DataScience #MLFundamentals #Preprocessing*

---
---

## POST 4 — Validation: The Concept That Determines Whether Your Results Mean Anything
*Post 4 of 8 — ML Fundamentals*

This is the post that matters most. Everything in Posts 2 and 3 is about getting your data in shape. This post is about whether the number you get at the end — your model's reported accuracy — actually means what you think it means.

It often doesn't.

---

**Start with the basic idea**

You train a model on some data. Then you test it on data it hasn't seen. The performance on the unseen data is your estimate of how the model will behave in the real world.

The train/test split is how you operationalize this. You hold back a portion of your data before training — typically 20-30% — train on the rest, and evaluate on the held-back portion.

This seems straightforward. The complications start when you think carefully about what "unseen" means for your specific dataset.

---

**Why a single split is often not enough**

With a single 80/20 split, your performance estimate depends heavily on which 20% ended up in the test set. If those rows happen to be easier or harder than average, your score is biased in that direction. With small datasets, this variance can be large enough to make the number meaningless.

Cross-validation solves this by running the evaluation multiple times.

📊 *[Suggested image: k-fold cross-validation diagram — dataset split into 5 blocks, each one serving as test set in turn. Search: "k-fold cross validation diagram 5 fold illustration"]*

In k-fold cross-validation, you split the dataset into k equal-sized groups (folds). You train on k-1 folds and test on the remaining one. You repeat this k times, rotating which fold is the test set. You average the scores across all k runs.

The result is a more stable performance estimate with a measure of variability (the standard deviation across folds). You also use all your data for both training and testing, at different times — a meaningful advantage when data is limited.

Common choices: k=5 or k=10. With very small datasets, Leave-One-Out CV (where k equals the number of rows) is sometimes used — every single row gets a turn as the sole test point.

---

**The part most tutorials skip: structured data requires structured splitting**

Standard k-fold shuffles rows randomly before splitting. That's appropriate when rows are truly independent. When they're not, random splitting creates leakage.

Consider a dataset built from scientific papers, where multiple rows come from the same publication. Those rows share something beyond chemistry: the same lab setup, the same instrument, the same authors' reporting conventions. A random split will put some rows from Paper A in training and others in test. The model can learn paper-specific patterns and apply them at test time — it's not generalizing to new science, it's recognizing a familiar paper.

*GroupKFold* solves this: instead of splitting rows randomly, you split by group. Every row from the same paper stays entirely in training or entirely in test. The model is now evaluated on papers it has genuinely never seen.

📊 *[Suggested image: GroupKFold diagram showing papers as colored blocks kept intact across splits. Search: "GroupKFold cross validation grouped split by source diagram"]*

The same logic applies to any dataset with natural groupings: patients in clinical data, buildings in energy data, subjects in psychology experiments. If rows within a group share structure beyond what's captured in the features, your splitting strategy needs to respect those groups.

---

**The practical consequence**

The gap between random-split performance and group-split performance is a direct measure of how much leakage you had. In many real research datasets, this gap is substantial — sometimes the difference between a model that looks good and one that actually generalizes.

While the train-test split is simple and quick to implement, the performance estimate can be highly dependent on the specific split, leading to high variance in the results. Cross-validation with a k of 5 or 10 addresses the variance problem. GroupKFold addresses the leakage problem. You often need both.

---

**One more type worth knowing: Leave-One-Group-Out (LOGO)**

The strictest version of grouped CV: each fold holds out exactly one group (one paper, one patient, one experiment). If you have 55 groups, you run 55 training-test cycles. This is computationally expensive but gives the most honest estimate of generalization — especially useful when you want to ask "how well does this model predict behavior from an entirely new source?"

---

**The takeaway**

Choosing a validation strategy isn't a formality. It's a scientific decision about what question you're asking your model to answer.

"Can you predict rows similar to what you've seen?" → standard k-fold is fine.
"Can you predict genuinely new groups you've never encountered?" → GroupKFold or LOGO.

If the answer to the second question is what you need and you're measuring with the first, your reported performance is wrong — in a direction that flatters the model.

---

*Next: Post 5 — The model lineup: which algorithm for which job.*

*#MachineLearning #DataScience #MLFundamentals #CrossValidation #ModelValidation*

---
---

## POST 5 — The Model Lineup: What Each Algorithm Actually Does
*Post 5 of 8 — ML Fundamentals*

Once your data is clean and your validation strategy is locked, you choose a model. This is the step that gets the most attention in popular ML content, and it's probably the third or fourth most important decision you'll make — behind data quality, feature engineering, and validation design.

That said, knowing what each model actually does — and when to reach for which one — is genuinely useful.

---

**The standard regression lineup**

For predicting a continuous number (regression), five model families appear in almost every comparative study. Here they are in order of complexity, not performance.

---

**Linear and Ridge/Lasso Regression**

The simplest possible model: a weighted sum of the input features. Each feature gets a coefficient; the prediction is just those coefficients multiplied by the feature values and summed up.

Why use it: it's interpretable, fast, and often underestimated. If log-transforming skewed features has linearized the relationships in your data, a regularized linear model (Ridge adds a penalty on large coefficients; Lasso can drive some to zero entirely) can perform surprisingly well.

Why it fails: it can only learn linear relationships. If the true relationship between a feature and the target bends, curves, or reverses direction depending on the value of another feature, a linear model will miss it.

Ridge is your first baseline. If Ridge does well, you understand your problem clearly. If it struggles, you know the relationships are more complex.

---

**Support Vector Regression (SVR)**

SVR finds a function that stays within a tube of tolerance around the data points, rather than minimizing average error across all points. It's robust to outliers and works well in moderate-dimensional spaces.

With an RBF (radial basis function) kernel, SVR can fit nonlinear relationships. The kernel effectively transforms the feature space into a higher-dimensional one where linear separation becomes possible.

Two things to know: SVR requires feature scaling (standardize your numerics), and it's sensitive to its hyperparameters (C, gamma, epsilon). Default settings are rarely optimal.

---

**Random Forest**

An ensemble of decision trees. Each tree is trained on a random sample of rows and a random subset of features, then the predictions of all trees are averaged.

📊 *[Suggested image: random forest — multiple decision trees voting on a prediction. Search: "random forest ensemble decision trees illustration diagram"]*

This randomness is what makes it work. Individual trees overfit badly; their average doesn't, because the errors are uncorrelated. The result is a model that's robust, handles mixed data types well, doesn't need scaling, and produces reliable feature importance estimates.

For tabular scientific data with moderate sample sizes, Random Forest is usually the first serious model to try. It's the practical baseline for everything non-linear.

---

**XGBoost (and LightGBM)**

Where Random Forest builds trees in parallel and averages them, XGBoost builds them sequentially — each new tree focuses specifically on the errors the previous ones made. This is called gradient boosting.

The result is typically more accurate than Random Forest on the same data, but more sensitive to hyperparameters and more prone to overfitting if not tuned carefully. It also handles missing values natively, which is a genuine practical advantage.

In most benchmarks on small-to-medium tabular datasets, XGBoost and Random Forest are the top performers. Which one wins depends on the specific data.

---

**MLP / Neural Network**

A multi-layer perceptron is a series of layers of weighted connections with nonlinear activation functions. It can, in principle, learn arbitrarily complex functions.

In practice, for tabular data with fewer than a few thousand rows, neural networks rarely outperform well-tuned tree ensembles. They need more data to generalize and more engineering to work well (architecture choice, learning rate, batch size, regularization). They're included in every benchmark for completeness and sometimes win on specific problems, but they are not the default first choice for structured scientific datasets.

---

**How to approach model selection in practice**

Start with defaults across all candidates. Do not tune yet — tuning before comparing defaults introduces too many degrees of freedom. Evaluate under your locked validation strategy, compare the mean and standard deviation of scores across folds, and eliminate the clearly weak ones.

Then tune the top 2-3 candidates using nested cross-validation (an inner loop for hyperparameter search, an outer loop for unbiased performance estimation). The winner is the model with the best outer-loop score, with ties broken in favor of the simpler model.

The comparison itself — across model families — is a result. If a linear model and an ensemble converge to similar scores, that tells you something real about the complexity of the underlying relationships in your data.

---

*Next: Post 6 — Bias, variance, and why your model can be confidently wrong.*

*#MachineLearning #DataScience #MLFundamentals #RandomForest #XGBoost*

---
---

## POST 6 — Overfitting, Underfitting, and the Balance Between Them
*Post 6 of 8 — ML Fundamentals*

A model can fail in two completely different ways. It can be too simple — missing real structure in the data. Or it can be too complex — fitting the data so closely that it memorizes noise instead of learning signal. These two failure modes have names, and understanding them tells you what to do when your model isn't working.

---

**Underfitting: the model is too simple**

If your model performs poorly on training data *and* on test data, it's underfitting. It hasn't captured the actual patterns in the data — its assumptions are too restrictive for the problem at hand.

A classic example: fitting a straight line to data that has a clear curve. No matter how much data you show the model, the line will always miss the curve. The error isn't random — it's systematic. This is called high bias.

Fixes: use a more expressive model, add relevant features, engineer better representations of the data.

---

**Overfitting: the model is too complex**

If your model performs very well on training data but poorly on test data, it's overfitting. It's learned the specific rows it was trained on — including their noise, their measurement errors, their paper-specific quirks — rather than the general pattern.

📊 *[Suggested image: overfitting vs underfitting vs good fit curve through scatter data. Search: "overfitting underfitting good fit machine learning curve illustration"]*

This is called high variance: change the training set slightly and the model changes substantially. It's not learning the underlying relationship — it's learning the particular sample.

---

**The bias-variance tradeoff**

These two failure modes trade off against each other as a function of model complexity.

A simple model has high bias (systematic error) but low variance (stable across different training sets). A complex model has low bias but high variance — it fits any training set well but generalizes poorly.

📊 *[Suggested image: U-shaped total error curve vs model complexity, showing bias decreasing and variance increasing, with sweet spot in the middle. Search: "bias variance tradeoff U curve model complexity diagram"]*

The goal is to find the complexity that minimizes total error on unseen data — not just training error. Training error almost always decreases as complexity increases; test error eventually starts rising again. The gap between training error and test error is what tells you whether you're on the overfitting side.

---

**Regularization: the practical solution**

Rather than choosing a completely different model, you can constrain the complexity of the model you have. This is called regularization.

For linear models, Ridge regression adds a penalty proportional to the size of the coefficients, discouraging the model from placing too much weight on any single feature. Lasso does the same but can drive coefficients to exactly zero, effectively removing features. The regularization strength (usually denoted alpha or lambda) is a hyperparameter you tune.

For trees, depth limits and minimum samples per leaf serve the same purpose. An unconstrained tree can grow until every training row has its own leaf — perfect training accuracy, terrible generalization. Limiting depth forces the tree to find patterns that hold across many rows rather than memorizing individual ones.

For neural networks, dropout randomly zeroes out units during training, preventing the network from relying too heavily on any individual node.

---

**What a parity plot tells you**

When you plot predicted values against actual values, the dots should cluster around the 45° diagonal. Systematic deviation from this line is informative:

- Dots consistently below the line at high values: the model underestimates extremes. Likely underfitting or a missing feature that matters most at high values.
- Dots scattered widely: high variance, probably overfitting.
- Fan-shaped spread (tight at low values, wide at high): the model struggles where the target is largest. Often a sign that a log transform of the target would help.

---

**A number worth checking that most tutorials don't mention**

The standard deviation of your cross-validation scores. A mean R² of 0.6 with std 0.05 is a very different result from a mean R² of 0.6 with std 0.4. The first means the model consistently explains 60% of variance across held-out folds. The second means the model is highly unstable — doing well on some paper groups and failing on others. Both report the same mean. Only one of them tells you you have a problem.

---

*Next: Post 7 — Feature importance: what is the model actually using to make predictions?*

*#MachineLearning #DataScience #MLFundamentals #Overfitting #BiasVariance*

---
---

## POST 7 — Feature Importance: Opening the Black Box
*Post 7 of 8 — ML Fundamentals*

You've trained a model. It generalizes reasonably well. Now the question changes: *what* is it actually using to make predictions?

This matters for two reasons. First, practical — understanding which inputs drive the output tells you where to focus future data collection or experimental effort. Second, scientific — if the model's dominant features don't align with what the underlying domain says should matter, either the model is learning spurious correlations, or you've discovered something unexpected. Either way, you need to know.

---

**Coefficient magnitude (linear models)**

For a Ridge or Lasso model, the simplest importance measure is the coefficient magnitude. A larger coefficient means the model places more weight on that feature. Sign matters: positive coefficient means higher feature value → higher prediction; negative means the opposite.

One caveat: coefficients are only comparable across features if you've standardized the input features (zero mean, unit variance). Otherwise, a large coefficient might just reflect a feature with a small numerical scale, not genuine importance.

---

**Permutation importance**

Method: take your trained model, hold it fixed (no retraining), shuffle the values of one feature randomly across all test rows, and measure how much worse the model's predictions get. If shuffling feature X causes the score to drop substantially, the model was relying on X. If the score barely changes, the model wasn't.

This is model-agnostic — it works on any model, regardless of how complex. It measures what actually matters for predictions on held-out data, not what the model assigns weight to internally.

One important implementation detail: run permutation importance inside your grouped CV folds, not on the full dataset. You want importance on genuinely unseen data, not data the model has already been exposed to.

---

**SHAP values**

SHAP (SHapley Additive exPlanations) is more sophisticated. For each prediction, SHAP computes how much each feature contributed — how much did knowing feature X move this particular prediction up or down relative to the baseline?

📊 *[Suggested image: SHAP beeswarm plot — features on y-axis, SHAP values on x-axis, dots colored by feature value. Search: "SHAP beeswarm plot feature importance machine learning"]*

The beeswarm plot is the most information-dense summary: each dot is one data point, horizontal position shows the magnitude and direction of that feature's contribution, and color shows the feature's value. You can read off at a glance: "high values of feature A push predictions up; low values push them down."

Beeswarm plots reveal not just the relative importance of features, but their actual relationships with the predicted outcome. This is more than a ranking — it shows you the *direction* of each feature's effect, which is where the domain sanity check happens.

---

**The sanity check: direction and magnitude**

Before reporting any feature importance result, ask whether the directions make physical or domain sense.

If your model says that higher surface area increases the predicted output, and your domain knowledge confirms that more surface area should increase performance — that's evidence the model learned something real. If the direction is reversed, or a completely uninformative column ranks first, investigate before concluding anything.

This check is not optional. A model that explains 70% of variance but attributes it to an uninformative feature is not a good model — it's a model that found a spurious correlation that will fail on new data.

---

**When SHAP and permutation importance disagree**

They will, sometimes. SHAP reflects what the model weights internally across the full training data. Permutation importance measures what actually generalizes on held-out groups.

When a feature ranks high in SHAP but near zero in permutation importance, it means: the model learned to rely on that feature, but that reliance doesn't transfer to unseen data. The feature carries paper-specific or sample-specific signal that the model exploited but that doesn't generalize.

This disagreement is itself a result — it tells you precisely where the model is overfitting to your training distribution. Don't ignore it. Report it.

---

**The broader point**

Feature importance is not a way to explain the model after the fact. It's a validation step — evidence that the model has learned real structure rather than noise. If the importance rankings are physically sensible and consistent across multiple methods, you have meaningful evidence that the model is doing something real. If they're not, no amount of good R² saves you.

---

*Next: Post 8 — When the model hits a ceiling, and what that actually means.*

*#MachineLearning #DataScience #MLFundamentals #SHAP #FeatureImportance #Interpretability*

---
---

## POST 8 — When the Model Hits a Ceiling
*Post 8 of 8 — ML Fundamentals*

This is the post I wish I had read before starting. Not because it would have saved me the work — the work was necessary. But because understanding this earlier would have changed how I interpreted the results along the way.

---

**The plateau**

You run your baseline. You preprocess carefully, engineer features, apply a proper validation strategy. You train five models. You tune the top performers. And then — you notice something. The scores cluster. Linear, kernel, and ensemble methods all converge to roughly the same number, and no amount of tuning pushes past it.

Your first instinct is that something is wrong. You check the code. You try a different encoding. You engineer more features. The number barely moves.

At some point you have to ask: what if the ceiling isn't a bug? What if it's the answer?

---

**What convergence across model families actually means**

When a linear model, an SVR with an RBF kernel, and a Random Forest all land at approximately the same performance under grouped cross-validation — that's a signal. Not a failure of any individual model, but a statement about the data itself.

The models are different enough that if there were learnable nonlinear structure they were missing, they wouldn't agree. The fact that they agree means one of two things: either all the learnable signal has been captured, or the remaining variance is genuinely noise — between-source variability that no model can extract because it's not encoded in the features.

---

**The dataset ceiling vs the model ceiling**

There are two different kinds of limits, and conflating them leads to wasted effort.

A *model ceiling* is a limit of a specific algorithm — a linear model can't learn nonlinear relationships, so switching to a tree ensemble raises the ceiling. This is fixable by model selection.

A *dataset ceiling* is the maximum predictable variance given the information available in your features. If the remaining variance is driven by factors not in your dataset — measurement conditions you didn't record, lab-to-lab variability, publication biases in reported values — no model will capture it, because the information simply isn't there. This is not fixable by tuning. It's fixable only by collecting better data.

In literature-mined scientific datasets especially, dataset ceilings appear frequently. Different labs use different instruments. Papers report different combinations of measurements. The target variable is measured under conditions that vary in ways your features don't fully describe. The noise floor is real and it's high.

---

**R² is a relative measure**

R² measures the fraction of variance in the target that your model explains. An R² of 0.15 means the model explains 15% of the variance — which sounds low. Whether it's a good result or a bad one depends entirely on how much of the variance is *explainable in principle* given your features.

If 70% of your target's variance is between-source noise that your features don't capture, then an explainable ceiling of ~20% means your model is doing well. If 90% of the variance is signal and you're explaining 15%, you have a real problem.

The baseline — always predict the training mean, regardless of features — gives you an R² of 0. Any model that doesn't beat this isn't useful. But the distance between your model and the baseline matters less than the distance between your model and the *ceiling of what's explainable*, which you can only estimate by understanding what your data does and doesn't contain.

---

**What actually moves the ceiling**

Not more hyperparameter tuning. Not a fancier architecture. The things that actually raise a dataset ceiling:

*More complete data* — filling in the missing measurements that most drove your imputation flags. If surface area is missing in 40% of rows and it's your most important feature, getting those values raises the ceiling directly.

*Better features* — not more features, but features that capture variation currently absorbed into noise. Replacing a flat categorical label with a physically grounded descriptor encodes information that was there but not usable.

*More controlled data collection* — if between-source variability is the noise floor, studies designed to hold everything constant except the variable you care about are worth more than ten times as many rows from the scattered literature.

*A more specific question* — sometimes the question is too broad. "Predict performance across all synthesis methods and electrolytes" is harder than "predict performance for hydrothermal synthesis in KOH." Narrowing the scope can raise the explainable fraction substantially.

---

**The honest result**

A modest, stable, well-validated score is more useful than a high score from a leaky pipeline. The first tells you something real about what can be predicted from the information you have. The second tells you how well your pipeline can exploit correlations that don't exist in new data.

If you've done the work — proper grouping, no leakage in preprocessing, honest imputation, domain-sensible feature importance — and the model lands at a modest R², that number is the finding. Report it clearly, explain what it means, and state what would need to change to raise it. That's rigorous science.

The ceiling isn't failure. It's where honest analysis ends and better data collection begins.

---

*That's the end of this series. Eight posts from raw data to interpreting why your model plateaued. The pipeline doesn't change much across problems — your data and your domain do. Start there.*

*#MachineLearning #DataScience #MLFundamentals #ModelInterpretation #Research*
