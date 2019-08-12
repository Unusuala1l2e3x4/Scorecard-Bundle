# Scorecard-Bundle

The one package you need for Scorecard modeling in Python | 评分卡建模尽在于此

- [English Document](#english-document)
- [中文文档  (Chinese Document)](#中文文档--chinese-document)

## English Document

**Scorecard-Bundle is a Python toolkit for Scorecard modeling of binary targets**. The transformer and model classes in Scorecard-Bundle **comply with Scikit-Learn**‘s fit-transform-predict convention.

There is a three-stage plan for Scorecard-Bundle:

- Stage 1 (Have been covered in v1.0): Replicate all functions of convectional Scorecard modeling, including:
  - Feature discretization with Chi-Merge;
  - WOE transformation and IV calculation;
  - Scorecard modeling based on Logistic regression;
  - Model Evaluation (binary classification evaluation);
- Stage 2 (Will be covered in v2.0): Add additional functionality, including:
  - Feature selection criteria (predictability + co-linearity + explainability);
  - Model scores discretization (if ratings are required);
  - Model Rating Evaluation (clustering quality evaluation);
  - Add discretization methods other than ChiMerge;
  - Add support for Scorecard based on algorithms other than Logistic Regression;
- Stage 3 (Will be covered in v3.0): Automate the modeling process, including:
  - Automatically select proper discretization methods for different features;
  - Automatically perform hyper-parameter tuning for LR-based Scorecard;
  - Automatically perform feature selection with consideration of predictability, co-linearity and explainability;
  - Provide an model pipeline that takes the input features, perform all the tasks (discretization, woe, etc.) inside it and return the scored samples and Scorecard rules. This simplify the modeling process to one line of code `model.fit_predict(X, y)`;

<img src="https://github.com/Lantianzz/ScorecardBundle/blob/master/pics/framework.svg">

## Quick Start

### Installment

- Pip: Scorecard-Bundle can be installed with pip `pip install scorecardbundle` 

- Manually: Down codes from github `<https://github.com/Lantianzz/Scorecard-Bundle>` and import them directly:

  ~~~python
  import sys
  sys.path.append('E:\Github\Scorecard-Bundle') # add path that contains the codes
  from scorecardbundle.feature_discretization import ChiMerge as cm
  from scorecardbundle.feature_encoding import WOE as woe
  from scorecardbundle.model_training import LogisticRegressionScoreCard as lrsc
  from scorecardbundle.model_evaluation import ModelEvaluation as me
  ~~~

### Usage

- Like Scikit-Learn, Scorecard-Bundle basiclly have two types of obejects, transforms and predictors. They comply with the fit-transform and fit-predict convention;
- An usage example can be found in https://github.com/Lantianzz/Scorecard-Bundle/blob/master/examples/Example_Basic_scorecard_modeling_with_Scorecard-Bundle.ipynb



## User Guide

### Feature discretization

### class: scorecardbundle.feature_discretization.ChiMerge.ChiMerge

ChiMerge is a discretization algorithm introduced by Randy Kerber in "ChiMerge: Discretization of Numeric Attributes". It can transform a numerical features into categorical feature or reduce the number of intervals in a ordinal feature based on the feature's distribution and the target classes' relative frequencies in each interval. As a result, it keep statistically significantly different intervals and merge similar ones.

##### Parameters

~~~mar
m: integer, optional(default=2)
    The number of adjacent intervals to compare during chi-squared test.

confidence_level: float, optional(default=0.9)
    The confidence level to determine the threshold for intervals to 
    be considered as different during the chi-square test.

max_intervals: int, optional(default=None)
    Specify the maximum number of intervals the discretized array will have.
    Sometimes (like when training a scorecard model) fewer intervals are 
    prefered. If do not need this option just set it to None.

min_intervals: int, optional(default=None)
    Specify the mininum number of intervals the discretized array will have.
    If do not need this option just set it to None.

initial_intervals: int, optional(default=100)
    The original Chimerge algorithm starts by putting each unique value 
    in an interval and merging through a loop. This can be time-consumming 
    when sample size is large. 
    Set the initial_intervals option to values other than None (like 10 or 100) 
    will make the algorithm start at the number of intervals specified (the 
    initial intervals are generated using quantiles). This can greatly shorten 
    the run time. If do not need this option just set it to None.

delimiter: string, optional(default='~')
    The returned array will be an array of intervals. Each interval is 
    representated by string (i.e. '1~2'), which takes the form 
    lower+delimiter+upper. This parameter control the symbol that 
    connects the lower and upper boundaries.

output_boundary: boolean, optional(default=False)
    If output_boundary is set to True. This function will output the 
    unique upper  boundaries of discretized array. If it is set to False,
    This funciton will output the discretized array.
    For example, if it is set to True and the array is discretized into 
    3 groups (1,2),(2,3),(3,4), this funciton will output an array of 
    [1,3,4].
~~~

##### Attributes

~~~
boundaries_: dict
    A dictionary that maps feature name to its merged boundaries.
fit_sample_size_: int
    The sampel size of fitted data.
transform_sample_size_:  int
    The sampel size of transformed data.
num_of_x_:  int
    The number of features.
columns_:  iterable
    An array of list of feature names.
~~~

##### Methods

~~~
fit(X, y): 
    fit the ChiMerge algorithm to the feature.

transform(X): 
    transform the feature using the ChiMerge fitted.

fit_transform(X, y): 
    fit the ChiMerge algorithm to the feature and transform it.    
~~~

### Feature encoding

#### class: scorecardbundle.feature_encoding.WOE.WOE_Encoder

Perform WOE transformation for features and calculate the information value (IV) of features with reference to the target variable y.

##### Parameters
~~~
epslon: float, optional(default=1e-10)
        Replace 0 with a very small number during division 
        or logrithm to avoid infinite value.       

output_dataframe: boolean, optional(default=False)
        if output_dataframe is set to True. The transform() function will
        return pandas.DataFrame. If it is set to False, the output will
        be numpy ndarray.
~~~
##### Attributes
~~~
iv_: a dictionary that contains feature names and their IV

result_dict_: a dictionary that contains feature names and 
    their WOE result tuple. Each WOE result tuple contains the
    woe value dictionary and the iv for the feature.
~~~
##### Methods
~~~
fit(X, y): 
        fit the WOE transformation to the feature.

transform(X): 
        transform the feature using the WOE fitted.

fit_transform(X, y): 
        fit the WOE transformation to the feature and transform it.         
~~~

### Feature selection

### Model training

#### class: scorecardbundle.model_training.LogisticRegressionScoreCard

Take woe-ed features, fit a regression and turn it into a scorecard

##### Parameters
~~~
woe_transformer: WOE transformer object from WOE module.

C:  float, optional(Default=1.0)
    regularization parameter in linear regression. Default value is 1. 
    A smaller value implies more regularization.
    See details in scikit-learn document.

class_weight: dict, optional(default=None)
    weights for each class of samples (e.g. {class_label: weight}) 
    in linear regression. This is to deal with imbalanced training data. 
    Setting this parameter to 'auto' will aotumatically use 
    class_weight function from scikit-learn to calculate the weights. 
    The equivalent codes are:
    >>> from sklearn.utils import class_weight
    >>> class_weights = class_weight.compute_class_weight('balanced', 
                                                          np.unique(y), y)

random_state: int, optional(default=None)
    random seed in linear regression. See details in scikit-learn doc.

PDO: int,  optional(default=-20)
    Points to double odds. One of the parameters of Scorecard.
    Default value is -20. 
    A positive value means the higher the scores, the lower 
    the probability of y being 1. 
    A negative value means the higher the scores, the higher 
    the probability of y being 1.

basePoints: int,  optional(default=100)
    the score for base odds(# of y=1/ # of y=0).

decimal: int,  optional(default=0)
    Control the number of decimals that the output scores have.
    Default is 0 (no decimal).

start_points: boolean, optional(default=False)
    There are two types of scorecards, with and without start points.
    True means the scorecard will have a start poitns. 

output_option: string, optional(default='excel')
    Controls the output format of scorecard. For now 'excel' is 
    the only option.

output_path: string, optional(default=None)
    The location to save the scorecard. e.g. r'D:\\Work\\jupyter\\'.

verbose: boolean, optioanl(default=False)
    When verbose is set to False, the predict() method only returns
    the total scores of samples. In this case the output of predict() 
    method will be numpy.array;
    When verbose is set to True, the predict() method will return
    the total scores, as well as the scores of each feature. In this case
    The output of predict() method will be pandas.DataFrame in order to 
    specify the feature names.

delimiter: string, optional(default='~')
    The feature interval is representated by string (i.e. '1~2'), 
    which takes the form lower+delimiter+upper. This parameter 
    is the symbol that connects the lower and upper boundaries.
~~~
##### Attributes
~~~
woe_df_: pandas.DataFrame, the scorecard.

AB_ : A and B when converting regression to scorecard.
~~~
##### Methods
~~~
fit(woed_X, y): 
        fit the Scorecard model.

predict(X_beforeWOE, load_scorecard=None): 
        Apply the model to the original feature 
        (before discretization and woe encoding).
        If user choose to upload their own Scorecard,
        user can pass a pandas.DataFrame to `load_scorecard`
        parameter. The dataframe should contain columns such as 
        feature, value, woe, beta and score. An example would
        be as followed (value is the range of feature values, woe 
        is the WOE encoding of that range, and score is the socre
        for that range):
        feature value   woe         beta        score
        x1      30~inf  0.377563    0.631033    5.0
        x1      20~-30  1.351546    0.631033    37.0
        x1      -inf~20 1.629890    0.631033    -17.0
~~~

### Model evaluation

#### function: scorecardbundle.model_evaluation import ModelEvaluation



## Update Log

### Updates in v0.5

- ChiMerge:
  - Rewrite everything with Numpy (basically no Pandas at all). Now no error would be raised during training, even with unbalanced samples where the old implementation usually crash.

- WOE
  - Rewrite everything with Numpy. The code efficiency is boosted due to matrix computation;
  - The feature selection function is removed from WOE and will be included in an independent feature selection submodule;

### Updates in v0.4

- ChiMerge：
  - When the distribution of a feature is heavily unbalanced (e.g. most values are the same), pandas.qcut will crash. Thus we will switch to pandas.cut durng the above circumstances.
- ModelEvaluation:
  - Fixed a bug in lift curve. WNow the codes can generalize better.

- Scorecard
  - Add predict_proba() function to return scores only
  - Modify predict() and predict_proba() so that they support numpy array as input.

### Updates in v0.3

- ChiMerge
  - Fix a bug in ChiMerge that caused errors when bining data with pandas.qcut. If there are too many decimals in the minimum value of the column (e.g. 10 decimals), this minimum value would become the left boundary of the smallest interval procuced by qcut. The problem is that all intervals produced by qcut  are open on the left and close on the right. This means the smallest interval will not contain this minimum value.  To fix this, just round the column with pands.Series.round() before applying qcut.
  - Add a parameter `min_intervals`. When we don't want any feature droppped due to lack of predictability, we can use this parameter to make it happen.
- Scorecard
  - When using scorecard the data ranges may exceed those encountered in training, thus now the lowest and highest boundaries for each feature is set to negative infinity and positive infinity respectively.
- ModelEvaluation
  - If this module is run in jupyter notebook, the charts it saved to local used to be blank. This bug is fixed.

### Updates in v0.2

- Fix errors in notes. E.g. the default criterion for corr should be 0.6 rather than 0.7;
- Add example of using sklearn.utils.class_weight;
- Make Sure most default parameters  are optimal for Suitability scorecard model; 



## 中文文档  (Chinese Document)