## Introduction

Machine learning models can often encapsulate unintentional bias that results in unfairness. With Fairlearn and Azure Machine Learning, you can detect and mitigate unfairness in your models. In this article, we will discuss about how to evaluate machine learning models for fairness and mitigate predictive disparity in a machine learning model.

## 1. Definition of model fairness

When we consider the concept of fairness concerning predictions made by machine learning models, it helps to be clear about what we mean by "fair".

The example in tutorial is really usefull to get better understanding of model fairness.

Suppose a classification model is used to predict the probability of successful loan repayment and therefore influences whether or not the loan is approved. The model will likely be trained using features that reflect the characteristics of the applicant, such as: Age, Employment status, Income, Savings and Current debt. These features are used to train a binary classification model that predicts whether an applicant will repay a loan. the model predicts that around 45% of applicants will successfully repay their loans. However, on reviewing loan approval records, you begin to suspect that fewer loans are approved for applicants aged 25 or younger than for applicants who are over 25. How can you be sure the model is fair to applicants in both age groups?

### 1.1 Meansuing disparity in predictions

One way to start evaluating the fairness of a model is to compare predictions for each group within a sensitive feature. For the loan approval model, Age is a sensitive feature that we care about, so we could split the data into subsets for each age group and compare the selection rate (the proportion of positive predictions) for each group.

Let's say we find that the model predicts that 36% of applicants aged 25 or younger will repay a loan, but it predicts successful repayments for 54% of applicants aged over 25. There's a disparity in predictions of 18%.

Let's say we find that the model predicts that 36% of applicants aged 25 or younger will repay a loan, but it predicts successful repayments for 54% of applicants aged over 25. There's a disparity in predictions of 18%.

At first glance, this comparison seems to confirm that there's bias in the model that discriminates against younger applicants. However, when you consider the population as a whole, it may be that younger people generally earn less than people more established in their careers, have lower levels of savings and assets, and have a higher rate of defaulting on loans.

The important point to consider here is that just because we want to ensure fairness regarding age, it doesn't necessarily follow that age is not a factor in loan repayment probability. It's possible that in general, younger people are less likely to repay a loan than older people. To get the full picture, we need to look a little deeper into the predictive performance of the model for each subset of the population.

## 1.2 Measuing disparity in prediction performance

When you train a machine learning model using a supervised technique, like regression or classification, you use metrics achieved against hold-out validation data to evaluate the overall predictive performance of the model. For example, you might evaluate a classification model based on accuracy, precision, or recall.

To evaluate the fairness of a model, you can apply the same predictive performance metric to subsets of the data, based on the sensitive features on which your population is grouped, and measure the disparity in those metrics across the subgroups.

For example, suppose the loan approval model exhibits an overall recall metric of 0.67 - in other words, it correctly identifies 67% of cases where the applicant repaid the loan. The question is whether or not the model provides a similar rate of correct predictions for different age groups.

To find out, we group the data based on the sensitive feature (Age) and measure the predictive performance metric (recall) for those groups. Then we can compare the metric scores to determine the disparity between them.

Let's say that we find that the recall for validation cases where the applicant is 25 or younger is 0.50, and recall for cases where the applicant is over 25 is 0.83. In other words, the model correctly identified 50% of the people in the 25 or younger age group who successfully repaid a loan (and therefore misclassified 50% of them as loan defaulters), but found 83% of loan repayers in the older age group (misclassifying only 17% of them). The disparity in prediction performance between the groups is 33%, with the model predicting significantly more false negatives for the younger age group.

## 1.3 Potential cause of disparity

When you find a disparity between prediction rates or prediction performance metrics across sensitive feature groups, it's worth considering potential causes. These might include:

1. Data imbalance. Some groups may be overrepresented in the training data, or the data may be skewed so that cases within a specific group aren't representative of the overall population.

2. Indirect correlation. The sensitive feature itself may not be predictive of the label, but there may be a hidden correlation between the sensitive feature and some other feature that influences the prediction. For example, there's likely a correlation between age and credit history, and there's likely a correlation between credit history and loan defaults. If the credit history feature is not included in the training data, the training algorithm may assign a predictive weight to age without accounting for credit history, which might make a difference to loan repayment probability.

3. Societal biases. Subconscious biases in the data collection, preparation, or modeling process may have influenced feature selection or other aspects of model design.

## 1.4 Mitigating bias

Optimizing for fairness in a machine learning model is a sociotechnical challenge. In other words, it's not always something you can achieve purely by applying technical corrections to a training algorithm. However, there are some strategies you can adopt to mitigate bias, including:

1. Balance training and validation data. You can apply over-sampling or under-sampling techniques to balance data and use stratified splitting algorithms to maintain representative proportions for training and validation.

2. Perform extensive feature selection and engineering analysis. Make sure you fully explore the interconnected correlations in your data to try to differentiate features that are directly predictive from features that encapsulate more complex, nuanced relationships. You can use the model interpretability support in Azure Machine Learning to understand how individual features influence predictions.

3. Evaluate models for disparity based on significant features. You can't easily address the bias in a model if you can't quantify it.

4. Trade-off overall predictive performance for the lower disparity in predictive performance between sensitive feature groups. A model that is 99.5% accurate with comparable performance across all groups is often more desirable than a model that is 99.9% accurate but discriminates against a particular subset of cases.

## 2. Analyzing model fairness with Fairlearn

Fairlearn is a Python package that you can use to analyze models and evaluate disparity between predictions and prediction performance for one or more sensitive features.

It works by calculating group metrics for the sensitive features you specify. The metrics themselves are based on standard scikit-learn model evaluation metrics, such as accuracy, precision, or recall for classification models.

The Fairlearn API is extensive, offering multiple ways to explore disparity in metrics across sensitive feature groupings. For a binary classification model, you might start by comparing the selection rate (the number of positive predictions for each group) by using the selection_rate function. This function returns the overall selection rate for the test dataset. You can also use standard sklearn.metrics functions (such as accuracy_score, precision_score, or recall_score) to get an overall view of how the model performs.

Then, you can define one or more sensitive features in your dataset with which you want to group subsets of the population and compare selection rate and predictive performance. Fairlearn includes a MetricFrame function that enables you to create a dataframe of multiple metrics by the group.

![image](https://user-images.githubusercontent.com/71245576/116635317-71edd800-a92c-11eb-8515-664b3cc10f14.png)

For example, in a binary classification model for loan repayment prediction, where the sensitive feature Age consists of two possible categorical values (25-and-under and over-25), a MetricFrame for these groups might be similar to the following table:


Fairlearn provides an interactive dashboard widget that you can use in a notebook to display group metrics for a model. The widget enables you to choose a sensitive feature and performance metric to compare, and then calculates and visualizes the metrics and disparity like this:

![image](https://user-images.githubusercontent.com/71245576/116635270-55ea3680-a92c-11eb-9b0a-c832d3cb8e2e.png)

Fairlearn integrates with Azure Machine Learning by enabling you to run an experiment in which the dashboard metrics are uploaded to your Azure Machine Learning workspace. This enables you to share the dashboard in Azure Machine Learning studio so that your data science team can track and compare disparity metrics for models registered in the workspace.

## 3. Mitigating unfairness with Fairlearn

In addition to enabling you to analyze disparity in selection rates and predictive performance across sensitive features, Fairlearn provides support for mitigating unfairness in models.

### 3.1 Mitigation algorithms and parity constraints

The mitigation support in Fairlearn is based on the use of algorithms to create alternative models that apply parity constraints to produce comparable metrics across sensitive feature groups. Fairlearn supports the following mitigation techniques.

![image](https://user-images.githubusercontent.com/71245576/116635373-a2357680-a92c-11eb-810f-9d38068c14eb.png)

The choice of parity constraint depends on the technique being used and the specific fairness criteria you want to apply. Constraints in Fairlearn include:

![image](https://user-images.githubusercontent.com/71245576/116635399-ae213880-a92c-11eb-8a77-306896d920bb.png)

A common approach to mitigation is to use one of the algorithms and constraints to train multiple models, and then compare their performance, selection rate, and disparity metrics to find the optimal model for your needs. Often, the choice of the model involves a trade-off between raw predictive performance and fairness - based on your definition of fairness for a given scenario. 

Generally, fairness is measured by a reduction in the disparity of feature selection (for example, ensuring that an equal proportion of members from each gender group is approved for a bank loan) or by a reduction in the disparity of performance metric (for example, ensuring that a model is equally accurate at identifying repayers and defaulters in each age group).

Just as when analyzing an individual model, you can register all of the models found during your mitigation testing and upload the dashboard metrics to Azure Machine Learning.

## Reference

Build AI solutions with Azure Machine Learning, retrieved from https://docs.microsoft.com/en-us/learn/paths/build-ai-solutions-with-azure-ml-service/












