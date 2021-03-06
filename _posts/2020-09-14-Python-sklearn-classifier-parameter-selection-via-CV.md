---
layout: post
title: Classifier Parameter Selection via CV, in Python "sklearn"
tags: [Data science cheat sheets]
---

![png](https://fanbuduke17.github.io/img/Sklearn_classifiers_CV_selection_13_0.png)

This post is simply some practice code for fitting and doing parameter selection (via cross validation) of SVM and Random Forest classifiers, on 3 made-up datasets. Everything is done in the `scikit-learn` library in `Python`.

Code is adapted from [this webpage](https://scikit-learn.org/stable/auto_examples/classification/plot_classifier_comparison.html). 

It looks a bit long, but that's mainly due to the complexity of plotting in `matplotlib.pyplot`.


First, import the necessary functions. (I've commented out certain lines form the original webpage that are not used here.)

```python
import matplotlib.pyplot as plt
from matplotlib.colors import ListedColormap
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import make_moons, make_circles, make_classification

#from sklearn.neural_network import MLPClassifier
#from sklearn.neighbors import KNeighborsClassifier
from sklearn.svm import SVC
#from sklearn.gaussian_process import GaussianProcessClassifier
#from sklearn.gaussian_process.kernels import RBF
#from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier, AdaBoostClassifier
#from sklearn.naive_bayes import GaussianNB
#from sklearn.discriminant_analysis import QuadraticDiscriminantAnalysis

from sklearn.model_selection import GridSearchCV, RandomizedSearchCV

from numpy.random import Generator, PCG64
```

Make up example datsets


```python
X, y = make_classification(n_features=2, n_redundant=0, n_informative=2,
                           random_state=1, n_clusters_per_class=1)
                           
rng = np.random.default_rng(PCG64(12345))
X += 2 * rng.uniform(size=X.shape)
linearly_separable = (X, y)

datasets = [make_moons(noise=0.3, random_state=0),
            make_circles(noise=0.2, factor=0.5, random_state=1),
            linearly_separable]
```

Initialize classifiers and specify parameter grids to search on.

```python
names = ['Linear SVM', 'RBF SVM', 'RF']
```


```python
# classifiers

clf_svc_linear = SVC(kernel='linear')
clf_svc_rbf = SVC(kernel='rbf')
clf_rf = RandomForestClassifier(max_features=1)
classifiers = [clf_svc_linear, clf_svc_rbf, clf_rf]
```


```python
# search grids

svc_linear_grid = {'C': [0.1, 1, 10]}
svc_rbf_grid = {'gamma': [0.1, 1, 10]}
rf_grid = {'max_depth': [3,4,5], 'n_estimators': [5,10,20]}
grids = [svc_linear_grid, svc_rbf_grid, rf_grid]
```

Go over the datasets, do CV selection, and predict with the best paramters selected.

Also print accuracy score on the test data set. 


```python
n_classifiers = len(classifiers)
figure = plt.figure(figsize=(n_classifiers * 3, 9))

h = 0.1 # step size for the mesh grid (for plotting)
```


```python
i = 1
# iterate over datasets
for ds_cnt, ds in enumerate(datasets):
    # preprocess dataset, split into training and test part
    X, y = ds
    X = StandardScaler().fit_transform(X)
    X_train, X_test, y_train, y_test = \
        train_test_split(X, y, test_size=.3, random_state=42)

    x_min, x_max = X[:, 0].min() - .5, X[:, 0].max() + .5
    y_min, y_max = X[:, 1].min() - .5, X[:, 1].max() + .5
    xx, yy = np.meshgrid(np.arange(x_min, x_max, h),
                         np.arange(y_min, y_max, h))

    # just plot the dataset first
    cm = plt.cm.RdBu
    cm_bright = ListedColormap(['#FF0000', '#0000FF'])
    ax = plt.subplot(len(datasets), len(classifiers) + 1, i)
    if ds_cnt == 0:
        ax.set_title("Input data")
    # Plot the training points
    ax.scatter(X_train[:, 0], X_train[:, 1], c=y_train, cmap=cm_bright,
               edgecolors='k')
    # Plot the testing points
    ax.scatter(X_test[:, 0], X_test[:, 1], c=y_test, cmap=cm_bright, alpha=0.6,
               edgecolors='k')
    ax.set_xlim(xx.min(), xx.max())
    ax.set_ylim(yy.min(), yy.max())
    ax.set_xticks(())
    ax.set_yticks(())
    i += 1

    # iterate over classifiers
    for j, clf in enumerate(classifiers):
        ax = plt.subplot(len(datasets), len(classifiers) + 1, i)
        
        # do CV parameter selection
        clf_cv = GridSearchCV(clf, grids[j])
        clf_cv.fit(X_train, y_train)
        
        # get the prediction score on the test set (using best parameters)
        score = clf_cv.score(X_test, y_test)

        # Plot the decision boundary. For that, we will assign a color to each
        # point in the mesh [x_min, x_max]x[y_min, y_max].
        if hasattr(clf_cv, "decision_function"):
            Z = clf_cv.decision_function(np.c_[xx.ravel(), yy.ravel()])
        else:
            Z = clf_cv.predict_proba(np.c_[xx.ravel(), yy.ravel()])[:, 1]

        # Put the result into a color plot
        Z = Z.reshape(xx.shape)
        ax.contourf(xx, yy, Z, cmap=cm, alpha=.8)

        # Plot the training points
        ax.scatter(X_train[:, 0], X_train[:, 1], c=y_train, cmap=cm_bright,
                   edgecolors='k')
        # Plot the testing points
        ax.scatter(X_test[:, 0], X_test[:, 1], c=y_test, cmap=cm_bright,
                   edgecolors='k', alpha=0.6)

        ax.set_xlim(xx.min(), xx.max())
        ax.set_ylim(yy.min(), yy.max())
        ax.set_xticks(())
        ax.set_yticks(())
        if ds_cnt == 0:
            ax.set_title(names[j])
        ax.text(xx.max() - .3, yy.min() + .3, ('%.2f' % score).lstrip('0'),
                size=15, horizontalalignment='right')
        i += 1

plt.tight_layout()
plt.show()
```
