# Logistic Regression Task

This article describes an example of a Delta Logistic Regression Task.

The dataset is the [Spector dataset](https://www.statsmodels.org/stable/datasets/generated/spector.html) which is the experimental data on the effectiveness of the personalized system of instruction (PSI) program. The data is stored in a CSV file including 4 columns:

* GPA: whether or not a student's grade improved
* TCUE: test score on economics test
* PSI: participation in program
* Grade: grade point average

And our task is to train a logistic regression model to predict whether the student's grade will be improved given his TCUE, PSI and Grade data.



