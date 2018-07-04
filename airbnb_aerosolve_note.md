## [aerosolve/airlearner/airlearner-strategy](https://github.com/airbnb/aerosolve/tree/master/airlearner/airlearner-strategy)

#### input feature
prob, basePivot
- prob or x is the house rent out probabilty w.r.t. house observed price(observedValue)
- basePivot or pivotValue, a base price of a house indicates the house's basic rent price

#### output target and model
observed price(or the ratio observed price/pivotVale) and 'linear regression'
- observed price is the user setting price for the house to be rent
- predict_observed_price =  ( w0 * (w1 * prob + 2) + 1) * pivotValue,
  (ratio = predict_observed_price/pivotValue = (w0*(w1 * prob + 2) + 1))

#### hyperparameter
false_label_lowerbound_rate, false_label_upperbound_rate, true_label_lowerbound_rate, true_label_upperbound_rate

The lowerbound price/upperbound price will be calculated by using these hyperparameters. 
The lowerbound price/upperbound price are used to decide the loss of the training data.

#### train loss
w0, w1, w2(the parameters in the model) are updated iteratively by calculating the gradient and loss

Each train example contains several useful data to calculate its loss and :
- x or prob (between 0 and 1)
- label or whether the house rent out(false or true)
- pivotValue or basePivot, average observed price in common sense
- observedValue or observed price
- trueValue or the price when house is rent out(label is true), meaningless/useless while label is false

Together with hyperparameters, we may calulate the loss and gradients for each example:
1. lowerbound/upperbound
    - label==true
        - lowerbound = (1 - (1 - true_label_lowerbound_rate) * 2 * (1-x)) * trueValue, 
        - upperbound =  (1 + (true_label_upperbound_rate - 1) * 2 * x) * pivotValue, 
        pivotValue could be replaced by trueValu which is not tested in the aerosolve repo
    - label==false
        - lowerbound = (1 - (1 - false_label_lowerbound_rate) * 2 * (1-x)) * min(pivotValue, observedValue),
        - upperbound = false_label_upperbound_rate * min(pivotValue, observedValue)
2. loss/grad_direction (loss for minimization; grad_direction for update parameters(i.e. w0 w1 w2))
    - the prediction(w.r.t. the given w0, w1, w2) < lowerbound
        - loss = lowerbound -prediction
        - grad_direction = -1
    - the prediction > upperbound
        - loss = prediction - upperbound
        - grad_direction = 1
    - othercase
        - loss = 0
        - grad_direction = 0
3. grad for w0, w1, w2 respectively at each example
    - can be calculated from grad_direction and the prediction folumn, 
    c.f., computeGradient function in [BaseParam.scala](https://github.com/airbnb/aerosolve/blob/987d97db8118ade55c10aace7685b1cf24347614/airlearner/airlearner-strategy/src/main/scala/com/airbnb/common/ml/strategy/params/BaseParam.scala)
    where gradA, gradB, gradC for the grad of w0, w1, w2 respectively.
    There might be an error in gradB check [this](https://github.com/airbnb/aerosolve/pull/289/files) pull request.

#### evaluate hypermeters

##### loss_ratio sum

A simple index to evaluate the performace for the given example
- true_sample_importance + false_sample_importace = 1
- label == true, loss_ratio = true_example_importance * max(0, trueValue - prediction) / trueValue
- label == false, loss_ratio = false_example_importance * max(0, prediction - observed_price) / observed_price

Evaluate hypermeters by the average of all the **training examples'** loss_ratios.
    
###### evaluation metric
the metric for evaluating the model performance by **evaluation examples** which helps to decide the hyperparameters

Each evaluation example contains same data as a train example.

All the examples used to evaluate a model will be grouped into 4 categories based on their labels and predict observed prices given by the model:
1. label==true(house rent out), prediction price higher than trueValue
2. label==true(house rent out), prediction price lower than trueValue
3. label==false(house not rent out), prediction price lower than observedValue
4. label==false(house not rent out), prediction price higher than observedValue

Check [BinaryMetrics.scala](https://github.com/airbnb/aerosolve/blob/987d97db8118ade55c10aace7685b1cf24347614/airlearner/airlearner-strategy/src/main/scala/com/airbnb/common/ml/strategy/eval/BinaryMetrics.scala) 
for the detail evaluation indices.

 

## Try [aerosolve/demo/income_prediction/](https://github.com/airbnb/aerosolve/tree/master/demo/income_prediction)

1. git clone airbnb/aerosolve repo
2. install thrift
3. install Gradle, intellij import project by gradle
4. install Spark 1.6.x (pre-built for hadoop 2.6) 
    - [Spark 1.6.3](https://spark.apache.org/docs/1.6.3/index.html)
    - [1.6.3 Spark Standalone Mode](https://spark.apache.org/docs/1.6.3/spark-standalone.html)

## ~~Try [aerosolve/airlearner/](https://github.com/airbnb/aerosolve/tree/master/airlearner)~~

0. install hadoop 2.6.x 
    - [Apache Hadoop 2.6.5 Setting up a Single Node Cluster](http://hadoop.apache.org/docs/r2.6.5/hadoop-project-dist/hadoop-common/SingleCluster.html)
1. install hive 2.1.x
    - [Apache Hive](http://hive.apache.org/)
    - ~~[Hive on Spark](https://cwiki.apache.org/confluence/display/Hive/Hive+on+Spark)~~
    - ~~[Hive on Spark Getting Started](https://cwiki.apache.org/confluence/display/Hive/Hive+on+Spark%3A+Getting+Started)~~
