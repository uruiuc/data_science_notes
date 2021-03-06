# Statistics

<!-- TOC -->

- [Statistics](#statistics)
    - [1. Descriptive statistics (summarize)](#1-descriptive-statistics-summarize)
        - [1.1. Measures of central tendency](#11-measures-of-central-tendency)
        - [1.2. Measures of variations](#12-measures-of-variations)
    - [2. Inferential statistics (generalize)](#2-inferential-statistics-generalize)
        - [2.1. Sample vs. population](#21-sample-vs-population)
        - [2.2. Hypothesis testing](#22-hypothesis-testing)
        - [2.3. Standard error of the mean - a measure of confidence](#23-standard-error-of-the-mean---a-measure-of-confidence)
        - [2.4. The Student's t-test](#24-the-students-t-test)
        - [2.5. One Sample T-Test](#25-one-sample-t-test)
        - [2.6. Independent T-Test](#26-independent-t-test)
        - [2.7. Analysis of Variance (ANOVA)](#27-analysis-of-variance-anova)
        - [2.8. Chi-Square test of independence](#28-chi-square-test-of-independence)
    - [3. Predictive statistics (unknown)](#3-predictive-statistics-unknown)
        - [3.1. Regression](#31-regression)

<!-- /TOC -->

## 1. Descriptive statistics (summarize)

### 1.1. Measures of central tendency
  
* mean
* median
* mode

| | pros | cons |
|---|---|---|
|mean| use all numbers, easy |susceptible to outliers|
|median| robust to outliers | only reflects 1-2 values of the dataset|
|mode|good for nominal or categorical data, or multiple clusters|less meaningful in small dataset| <br>

`from stats import mean, median, mode, multi_mode`

### 1.2. Measures of variations

* variance
* standard deviation
* z-score:   how far one value is from the mean

    These statistics are susceptible to outliers <br>
    Okay to drop outliers when ... <br>
    Not okay to drop outliers when ... <br>

* interquartile range (IQR): 3rd quartile - 1st quartile

    IQR as automatic way to identify outliers <br>
    lower outliers: Q1 - 1.5 * IQR <br>
    upper outliers: Q3 + 1.5 * IQR <br>

    `plt.boxplot(arr, showmeans=True)` <br>
    median, IQR, whisker shows data excluding outliers, outliers plotted as individual points

## 2. Inferential statistics (generalize)

**Inferential statistics** are concerned with making inferences based on relations found in the sample, to relations in the population.

### 2.1. Sample vs. population

* Terminology

    * **Parameter:** a number that describes the population, typically unknown
    * **Statistic:** a number that is computed from a sample
    * **Sampling variability:** the statistics of different samples of a population will differ somewhat
    * Sample skewness + fundamental randomness
    * **Sampling distribution** is the distribution of a statistic across an infinite number of samples

* Central Limit Theorem

    As long as adequately large samples and adequately large number of samples are used from a population, the distribution of the statistics of the samples will be normally distributed.

### 2.2. Hypothesis testing

**Hypothesis testing:** Assessing evidence provided by the data in favor of or against each hypothesis about the population.

1. Specify the null ($H_0$) and alternative ($H_a$) hypothesis
2. Choose a sample
3. Assess the evidence
    * A result is **statistically significant** if it is unlikely to have occurred by chance.
    * Significance level of a test: $\alpha=0.05$
    * **p-value** is also called **type one error rate** = the probability we would be wrong in rejecting null hypothesis
    * Probability value **p-value < 0.05**: more than 95% likely that the association of interest would be present following repeated samples drawn from the population (aka., sampling distribution).
    <img src="resources/bivariate_stat_tools.png" width=400>
4. Draw conclusions
   * If p-value < $\alpha$: the data provide significant evidence against the null hypothesis, so we reject the null hypothesis and accept the alternative hypothesis.
   * If p-value > $\alpha$: the data do not provide enough evidence to reject the null hypothesis; the data do not provide enough evidence to accept the alternative hypothesis.

### 2.3. Standard error of the mean - a measure of confidence

* SE = s/sqrt(n)
* s: standard deviation of sample
* n: sample size

    `from scipy.stats import sem` <br>
    `plt.errorbar(x_axis, means, yerr = standard_errors, fmt='o')` <br>

    comparing error bars could infer whether samples were drawn from the same population

### 2.4. The Student's t-test

how likely that two samples represent the same underlying population

`from scipy.stats import sem, ttest_ind`
`(t_stat, p) = ttest_ind(high_prices, low_prices, equal_var=False)`

### 2.5. One Sample T-Test

compare sample mean to the population mean

Assumptions

* Data is normally distributed
* Data is independent
* Data is randomly sampled

```python
import scipy.stats as stats
stats.ttest_1samp(sample, population.mean())
```

### 2.6. Independent T-Test

compare the means of 2 independent populations

* Assumptions

    * Data is normally distributed
    * Data is independent
    * Data is homogenous (The standard deviations are roughly equal)

* Code

    ```python
    import scipy.stats as stats

    # Note: Setting equal_var=False performs Welch's t-test which does 
    # not assume equal population variance
    stats.ttest_ind(population1, population2, equal_var=False)
    ```

### 2.7. Analysis of Variance (ANOVA)

Examine differences in the mean response variable for each category of our explanatory variable. E.g. Are any of the treatments significantly different than the rest?

* Hypothesis

    $H_0$: $\mu_1=\mu_2=\mu_3=\mu_4$ <br>
    $H_a$: not all the $\mu$ are equal

* ANOVA F test
  
    Are the differences among the sample means due to true differences among the population means or merely due to sampling variability?

    **F statistic:** $F=\frac{variation\ among\ sample\ means}{variation\ within\ groups}$

* Code

    * Explanatory variable with 2 levels

        ```python
        import statsmodels.formula.api as smf
        
        # using ols function for calculating the F-statistic and associated p value
        # C(...) indicates categorical variable
        # dataset has no missing values
        model1 = smf.ols(formula='NUMCIGMO_EST ~ C(MAJORDEPLIFE)', data=sub1)
        results1 = model1.fit()
        print (results1.summary())
        ```

        Alternative syntax

        ```python
        import scipy.stats as stats

        # Perform the ANOVA
        stats.f_oneway(group1, group2, group3, group4, group5)
        ```

    * Explanatory variable with >2 levels

        ```python
        model2 = smf.ols(formula='NUMCIGMO_EST ~ C(ETHRACE2A)', data=sub3).fit()
        print (model2.summary())
        ```

        A significant ANOVA does not tell which groups are different from the others

    * Post hoc test when explanatory variable has >2 levels

        Cannot compare each pair one-by-one due to the increase in the Type 1 error rate called the family-wise error rate, the error rate for the group of pair comparison.

        <img src="resources/family_wise_error.png" width=400>

        Use one of the post hoc tests that are designed to evaluate the difference between pairs of means while protecting against inflation of Type 1 errors:
        
        * Sidak
        * Holm T test
        * Fisher's Least Significant Difference Test
        * Tukey's Honestly Significant Difference Test
        * Scheffe test
        * Newman-Keuls test
        * Dunnett's Multiple Comparison Test
        * Duncan Multiple Range Test
        * Bonferroni Procedure
  
        ```python
        import statsmodels.stats.multicomp as multi
        mc1 = multi.MultiComparison(sub3['NUMCIGMO_EST'], sub3['ETHRACE2A'])
        res1 = mc1.tukeyhsd()
        print(res1.summary())
        ```

### 2.8. Chi-Square test of independence

Which of the 2 categorical variables plays the role of the explanatory variable and then calculating the conditional percentages separately.

Whether there is a significant difference between the expected frequencies and the observed frequencies in one or more categories.

* Hypothesis

    $H_0$:
    
    * There is no difference in the drunk driving rate between males and females.
    * There is no relationship between the 2 categorical variables - they are independent.
    * Proportion of male drunk drivers = proportion of female drunk drivers.
  
    $H_a$:
    
    * There is a difference in the drunk driving rate between males and females.
    * There is a relationship between the 2 categorical variables - they are not independent.
    * Proportion of male drunk drivers ≠ proportion of female drunk drivers.

* $\chi^2$ test of independence

    <img src="resources/chi_suqare.png" width=400>
    <img src="resources/chi_suqare_table.png" width=600>
    <img src="resources/chi_suqare_txt.png" width=400>

    The p-value of the $\chi^2$ test of independence is the probability of getting counts like those observed, assuming that the 2 variables are not related.

* Code

    * Chi Square test of independence

        Test

        ```python
        import scipy.stats
        import seaborn
        import matplotlib.pyplot as plt

        # contingency table of observed counts
        ct1=pandas.crosstab(sub2['TAB12MDX'], sub2['USFREQMO']) # response, explanatory
        print (ct1)

        # column percentages
        colsum=ct1.sum(axis=0)
        colpct=ct1/colsum
        print(colpct)

        # chi-square
        print ('chi-square value, p value, expected counts')
        cs1= scipy.stats.chi2_contingency(ct1)
        print (cs1)
        ```

        Visualize

        ```python
        # set variable types 
        sub2["USFREQMO"] = sub2["USFREQMO"].astype('category')
        
        # new code for setting variables to numeric:
        sub2['TAB12MDX'] = pandas.to_numeric(sub2['TAB12MDX'], errors='coerce')
        
        # graph percent with nicotine dependence within each smoking frequency group 
        seaborn.factorplot(x="USFREQMO", y="TAB12MDX", data=sub2, kind="bar", ci=None)
        plt.xlabel('Days smoked per month')
        plt.ylabel('Proportion Nicotine Dependent')
        ```

        Alternative syntax

        ```python
        import scipy.stats as stats

        # The degree of freedom is assumed 2
        # With a p-value of 0.05, the confidence level is 1.00-0.05 = 0.95.
        critical_value = stats.chi2.ppf(q = 0.95, df = 2)

        # Run the chi square test with stats.chisquare()
        stats.chisquare(df['observed'], df['expected'])
        ```

    * Post hoc test when explanatory variable has >2 levels
        
        Bonferroni Adjusted p-value = $\frac{0.05}{\#\ comparisons}$ <br>
        Run $\chi^2$ test of independence for each paired comparison

        <img src="resources/chi_suqare_post_hoc.png" width=400>

        ```python
        recode2 = {1: 1, 2.5: 2.5}
        sub2['COMP1v2']= sub2['USFREQMO'].map(recode2)

        # contingency table of observed counts
        ct2=pandas.crosstab(sub2['TAB12MDX'], sub2['COMP1v2'])
        print (ct2)

        # column percentages
        colsum=ct2.sum(axis=0)
        colpct=ct2/colsum
        print(colpct)

        print ('chi-square value, p value, expected counts')
        cs2= scipy.stats.chi2_contingency(ct2)
        print (cs2)

        recode3 = {1: 1, 6: 6}
        sub2['COMP1v6']= sub2['USFREQMO'].map(recode3)

        # contingency table of observed counts
        ct3=pandas.crosstab(sub2['TAB12MDX'], sub2['COMP1v6'])
        print (ct3)

        # column percentages
        colsum=ct3.sum(axis=0)
        colpct=ct3/colsum
        print(colpct)

        print ('chi-square value, p value, expected counts')
        cs3= scipy.stats.chi2_contingency(ct3)
        print (cs3)
        ```



## 3. Predictive statistics (unknown)

### 3.1. Regression

`from scipy.stats import linregress`
`(slope, intercept, _, _, _) = linregress(x_data, y_data)`
`y_pred = slope * x_data + intercept`

