# DynaML

[![Build Status](https://travis-ci.org/mandar2812/DynaML.svg?branch=branch-1.0)](https://travis-ci.org/mandar2812/DynaML)

Aim
============

DynaML is a scala library/repl for implementing and working with general Machine Learning models. Machine Learning/AI applications make heavy use of various entities such as graphs, vectors, matrices etc as well as classes of mathematical models which deal with broadly three kinds of tasks, prediction, classification and clustering.

The aim is to build a robust set of abstract classes and interfaces, which can be extended easily to implement advanced models for small and large scale applications.

But the library can also be used as an educational/research tool for multi scale data analysis. 

Currently DynaML has implementations of Least Squares Support Vector Machine (LS-SVM) for binary classification and regression. For further background on LS-SVM consider [Wikipedia](https://en.wikipedia.org/wiki/Least_squares_support_vector_machine) or the [book] (http://www.amazon.com/Least-Squares-Support-Vector-Machines/dp/9812381511).   


Installation
============
Prerequisites: Maven

* Clone this repository
* Run the following.
```shell
  mvn clean compile
  mvn package
```

* Make sure you give execution permission to `DynaML` in the `target/bin` directory.
```shell
  chmod +x target/bin/DynaML
  target/bin/DynaML
```
  You should get the following prompt.
  
```
    ___       ___       ___       ___       ___       ___   
   /\  \     /\__\     /\__\     /\  \     /\__\     /\__\  
  /::\  \   |::L__L   /:| _|_   /::\  \   /::L_L_   /:/  /  
 /:/\:\__\  |:::\__\ /::|/\__\ /::\:\__\ /:/L:\__\ /:/__/   
 \:\/:/  /  /:;;/__/ \/|::/  / \/\::/  / \/_/:/  / \:\  \   
  \::/  /   \/__/      |:/  /    /:/  /    /:/  /   \:\__\  
   \/__/               \/__/     \/__/     \/__/     \/__/  

Welcome to DynaML v 1.2
Interactive Scala shell

DynaML>
```

Getting Started
===============

The `data/` directory contains a few sample data sets, and the root directory also has example scripts which can be executed in the shell.

* First we create a linear classification model on a csv data set. We will assume that the last column in each line of the file is the target value, and we build an LS-SVM model.

```scala
	val config = Map("file" -> "data/ripley.csv", "delim" -> ",", "head" -> "false", "task" -> "classification")
	val model = LSSVMModel(config)
```

* We can now (optionally) add a Kernel on the model to create a generalized linear Bayesian model.

```scala
  val rbf = new RBFKernel(1.025)
  model.applyKernel(rbf)
```

```
15/08/03 19:07:42 INFO GreedyEntropySelector$: Returning final prototype set
15/08/03 19:07:42 INFO SVMKernel$: Constructing key-value representation of kernel matrix.
15/08/03 19:07:42 INFO SVMKernel$: Dimension: 13 x 13
15/08/03 19:07:42 INFO SVMKernelMatrix: Eigenvalue decomposition of the kernel matrix using JBlas.
15/08/03 19:07:42 INFO SVMKernelMatrix: Eigenvalue stats: 0.09104374173019622 =< lambda =< 3.110068839504519
15/08/03 19:07:42 INFO LSSVMModel: Applying Feature map to data set
15/08/03 19:07:42 INFO LSSVMModel: DONE: Applying Feature map to data set
DynaML>
```

* Now we can solve the optimization problem posed by the LS-SVM in the parameter space. Since the LS-SVM problem is equivalent to ridge regression, we have to specify a regularization constant.

```scala
  model.setRegParam(1.5).learn
```

* We can now predict the value of the target variable given a new point consisting of a Vector of features using `model.predict()`.

* Evaluating models is easy in DynaML. You can create an evaluation object as follows. 

```scala
	val configtest = Map("file" -> "data/ripleytest.csv", "delim" -> ",", "head" -> "false")
	val met = model.evaluate(configtest)
	met.print
```

* The object `met` has a `print()` method which will dump some performance metrics in the shell. But you can also generate plots by using the `generatePlots()` method.

```
15/08/03 19:08:40 INFO BinaryClassificationMetrics: Classification Model Performance
15/08/03 19:08:40 INFO BinaryClassificationMetrics: ============================
15/08/03 19:08:40 INFO BinaryClassificationMetrics: Accuracy: 0.6172839506172839
15/08/03 19:08:40 INFO BinaryClassificationMetrics: Area under ROC: 0.2019607843137254
```

```scala
met.generatePlots
```

![Image of Plots](https://lh3.googleusercontent.com/_5_3y0lkD2lDETqw8RHfUzdFJOO4CLn4jreI5iaWu51vyERoy4F5VzROzhZJM2sYnDPh3VuYAZBKNOM=w1291-h561-rw)

* Although kernel based models allow great flexibility in modeling non linear behavior in data, they are highly sensitive to the values of their hyper-parameters. For example if we use a Radial Basis Function (RBF) Kernel, it is a non trivial problem to find the best values of the kernel bandwidth and the regularization constant.

* In order to find the best hyper-parameters for a general kernel based supervised learning model, we use methods in gradient free global optimization. This is relevant because the cost (objective) function for the hyper-parameters is not smooth in general. In fact in most common scenarios the objective function is defined in terms of some kind of cross validation performance.

* DynaML has a robust global optimization API, currently Coupled Simulated Annealing and Grid Search algorithms are implemented, the API in the package ```org.kuleven.esat.optimization``` can be extended to implement any general gradient or gradient free optimization methods.

* Lets tune an RBF kernel on the Ripley data.

```scala
import com.tinkerpop.blueprints.Graph
import com.tinkerpop.frames.FramedGraph
import io.github.mandar2812.dynaml.graphutils.CausalEdge
val (optModel, optConfig) = KernelizedModel.getOptimizedModel[FramedGraph[Graph],
      Iterable[CausalEdge], model.type](model, "csa",
      "RBF", 13, 7, 0.3, true)
```

We see a long list of logs which end in something like the snippet below, the Coupled Simulated Annealing model, gives us a set of hyper-parameters and their values. 
```
optModel: io.github.mandar2812.dynaml.models.svm.LSSVMModel = io.github.mandar2812.dynaml.models.svm.LSSVMModel@3662a98a
optConfig: scala.collection.immutable.Map[String,Double] = Map(bandwidth -> 3.824956165264642, RegParam -> 12.303758608075587)
```

To inspect the performance of this kernel model on an independent test set, we can use the ```model.evaluate()``` function. But before that we must train this 'optimized' kernel model on the training set.

```scala
optModel.setMaxIterations(2).learn()
val met = optModel.evaluate(configtest)
met.print()
met.generatePlots()
```

And the evaluation results follow ...

```
15/08/03 19:10:13 INFO BinaryClassificationMetrics: Classification Model Performance
15/08/03 19:10:13 INFO BinaryClassificationMetrics: ============================
15/08/03 19:10:13 INFO BinaryClassificationMetrics: Accuracy: 0.8765432098765432
15/08/03 19:10:13 INFO BinaryClassificationMetrics: Area under ROC: 0.9143790849673203
```

Documentation
=============
You can refer to the project [home page](http://mandar2812.github.io/DynaML/) or the [documentation](http://mandar2812.github.io/DynaML/target/site/scaladocs/index.html#package) for getting started with DynaML. Bear in mind that this is still at its infancy and there will be many more improvements/tweaks in the future.
