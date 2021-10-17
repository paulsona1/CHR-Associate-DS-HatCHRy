# CHR-Associate-DS-HatCHRy

**Section 1. Data and notebook set up**

This analysis constructs classifiers for our inferential target, CurrentCondition - a factor variable indicating order status. In this most initial section, I import the provided data and inspect the levels of the target variable. I observe seven non-missing levels, which are highly unbalanced. In particular, approximately 60% of the orders have condition status "Accepted", compared to .05% of the orders having condition status "Waiting On Recommendation". 

**Section 2. Exploratory Data Analysis**

Now, I do some exploratory data analysis. We begin by assessing the possibility of duplication within the orders data. We find a single duplicate request_id, corresponding to request_id **f0293ccd87b445f5989c6c68726608dc**. On inspection of these duplicate request_id, they appear to be true duplicates, so I remove one of the duplicates, keeping the other. 

I proceed by identifying missingness in the features. I find four columns having incomplete cases: (1) order_distance, (2) order_origin_weight, (3) rate_norm and (4) est_cost_norm. On inspection of some samples of rows with these missing features, I assume that these data are missing at random. In any case, the missingness is rare and will probably not sway our analysis.

Next, we are interested in whether any features might be thought of as redundant to correlation. Among numeric variables, we produce a correlation matrix, which shows, unsurprisingly, that (1) fields rate_norm and est_cost_norm exhibit high correlation (rho=0.805) and (2) fields order_distance and miles exhibit high correlation (rho=0.982). These correlations are observable also in the two scatter plots provided. The combinations of these four variables are also correlated, though they are likely capturing different information related to the order process. For these reasons, we eliminate fields rate_norm and miles from the data. 

To shape expectations about the most important factors associated with our inferential target, I produced two other visualizations. These factor frequency bar plots help me understand the amounts of segmentation of CurrentCondition arising from two features: (1) color and (2) lead_days. It is striking that "rejections" are well segmented from other CurrentCondition levels by the level of color. This is likely by design, based on the data dictionary. Second, we may be able to re-bucket the lead_days field to produce more credible intervals and, therefore, better segmentation of CurrentCondition. In particular, there's likely no difference on the level of CurrentCondition based on whether the lead_days are 60 or 61; for this reason, there's a reasonable argument to group and bucket these into percentiles. When we look at the frequency bar plot, we see that CurrentCondition varies considerably based on the factor/percentile levels of lead_days. 

These observations lead us to believe that color and the re-bucketed lead_days variables will be important features.

**Section 3. Classification modeling**

We opt for simple methods of classification for a couple reasons.  I consider using SVM, but the large unbalance in CurrentCondition disqualifies it as a useful method for us at this moment. Designing weighting contrasts for SVM without much domain knowledge is probably outside of the scope of this project. Instead, I choose to classify CurrentCondition using naive Bayes and multinomial logistic regression. These approaches are better suited for this problem because, given my lack of domain expertise, I prefer simpler methods.

From these two methods, we visualize the results. For Naive Bayes, we see the class conditional probabilities as model output. The intuition of our exploratory data analysis is confirmed regarding the importance of features color and lead days. For visualization of the multinomial logistic regression, we simplify the problem by considering solely the main two inferential target classes - "Acceptance" and "Rejection". Then, we plot the fitted posterior probabilities of each class given the lead day factor level. We see that the model is picking up the analysis from the exploratory data work. In particular, for the middling two levels of lead day factor, the probability of "Rejection" is higher.

In total, we find that the multinomial regression model is outperformed by the naive Bayes model on test error rates. We do not find evidence of difference in training and testing error rates, indicating that our model likely does not overfit.

**Section 4. Follow-Up Questions and Conclusions**

After completing this work, I have 2 follow-up questions. First, what is the benchmark error rate within this classification project and, equivalently, what would constitute a sizable reduction in the error rate? Second, my analysis drastically simplified freight lanes by not considering origin and destination information. Does a mapping exist that can aggregate the ~800 destinations into smaller, more analyzable geographic chunks? Would such a mapping even be appropriate?

If I were an employee, these questions are hopefully specific enough whereby I could ask a manager, fellow data scientist or subject matter expert and get an answer.
