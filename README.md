# c-lasso: a Python package for constrained sparse regression and classification 
=========

c-lasso is a Python package that enables sparse and robust linear regression and classification with linear equality
constraints on the model parameters. The package implements several algorithmic strategies, including path and proximal
splitting algorithms, that are applicable to different problem formulations, e.g., the constrained Lasso, the constrained
scaled Lasso, and sparse Huber M-estimation with linear equality constraints. We also include two model selection strategies
for determining the sparsity of the model parameters: k-fold cross-validation and stability selection.   

This package is intended to fill the gap between popular python tools such as [scikit-learn](https://scikit-learn.org/stable/) which CANNOT solve sparse constrained problems and general-purpose optimization solvers that do not scale well for the particular problems considered here.

Below we show several use cases of the package, including an application of sparse log-contrast
regression tasks for compositional microbiome data.

The code builds on results from several papers which can be found in the [References](#references).

## Table of Contents

* [Installation](#installation)
* [Regression and classification problems](#regression-and-classification-problems)
* [Getting started](#getting-started)
* [Two main functions](#two-main-functions)
* [Misc functions](#little-functions)
* [Log-contrast regression for microbiome data](#example)
* [Optimization schemes](#optimization-schemes)
* [References](#references)


##  Installation

c-lasso is available on pip. You can install the package using

```shell
pip install c_lasso
```
To use the c-lasso package in Python, type 

```python
from classo import *
```

The c-lasso package depends on several standard packages. 
To import these packages, use 

```shell
pip install numpy
pip install matplotlib
pip install scipy
pip install pandas
pip install time
```
    
##  Regression and classification problems

The c-lasso package can solve four different types of optimization problems: 
four regression-type and two classification-type problems.

### [R1] Standard constrained Lasso regression:             

<img src="https://latex.codecogs.com/gif.latex?\min_{C\beta=0}&space;||&space;X\beta-y&space;||^2&space;&plus;&space;\lambda&space;||\beta||_1" />

This is the default regression problem in c-lasso. The objective function combines Least-Squares for 
model fitting with l1 and linear equality constraints on the &beta; vector.   

### [R2] Contrained sparse Huber regression:                   

<img src="https://latex.codecogs.com/gif.latex?\min_{C\beta=0}&space;h_{\rho}(X\beta-y)&space;&plus;&space;\lambda&space;||\beta||_1"  />

This regression problem uses the [Huber loss](https://en.wikipedia.org/wiki/Huber_loss) as objective function 
for robust model fitting with l1 and linear equality constraints on the &beta; vector. The parameter &rho;=1.345.

### [R3] Contrained scaled Lasso regression: 

<img src="https://latex.codecogs.com/gif.latex?\min_{C\beta=0}&space;\frac{||&space;X\beta-y&space;||^2}{\sigma}&plus;&space;n\sigma&space;&plus;&space;\lambda&space;||\beta||_1"  />

This formulation is similar to [R1] but allows joint estimation of the (constrained) &beta; vector and 
the standard deviation &sigma; in a concomitant fashion (see References(#references) [4,5] for further info).

### [R4] Contrained sparse Huber regression with concomitant scale estimation:        

<img src="https://latex.codecogs.com/gif.latex?\min_{C\beta=0}&space;h_{\rho}(\frac{X\beta-y}{\sigma}&space;)&space;&plus;&space;n\sigma&space;&plus;&space;\lambda&space;||\beta||_1" />

This formulation combines [R2] and [R3] to allow robust joint estimation of the (constrained) &beta; vector and 
the standard deviation &sigma; in a concomitant fashion (see References(#references) [4,5] for further info).

### [C1] Contrained sparse classification with Square Hinge loss:        

### [C2] Contrained sparse classification with Huberized Square Hinge loss:        



## Getting started

Here is an example of use of one of the methods  : concomitant algorithm with theoretical lambda, tested on data generated randomly. 

To generate the data :
```python
m,d,d_nonzero,k,sigma =100,100,5,1,0.5
(X,C,y),sol = random_data(m,d,d_nonzero,k,sigma,zerosum=True)
```
Use of the package with default settings (example1) :
```python
problem = classo_problem(X,y,C) 
problem.solve()
print(problem)
print(problem.solution)
```

Results : 

```
FORMULATION : Concomitant
 
MODEL SELECTION COMPUTED :  Stability selection, 
 
STABILITY SELECTION PARAMETERS: method = first;  lamin = 0.01;  B = 50;  q = 10;  pourcent_nS = 0.5;  threshold = 0.9;  numerical_method = ODE

SPEEDNESS : 
Running time for Cross Validation    : 'not computed'
Running time for Stability Selection : 2.15s
Running time for Fixed LAM           : 'not computed'
```

![Ex1.1](figures/example1/Figure1.png)

![Ex1.2](figures/example1/Figure2.png)

![Ex1.3](figures/example1/Figure3.png)


Example of different settings (example2) : 
```python
problem                                     = classo_problem(X,y,C)
problem.formulation.huber                   = True
problem.formulation.concomitant             = False
problem.model_selection.CV                  = True
problem.model_selection.LAMfixed            = True
problem.model_selection.SSparameters.method = 'max'
problem.solve()
print(problem)
print(problem.solution)

problem.solution.CV.graphic(mse_max = 1.)
```

Results : 
```
FORMULATION : Huber
 
MODEL SELECTION COMPUTED :  Cross Validation,  Stability selection, Lambda fixed
 
CROSS VALIDATION PARAMETERS: Nsubset = 5  lamin = 0.001  n_lam = 500;  numerical_method = ODE
 
STABILITY SELECTION PARAMETERS: method = max;  lamin = 0.01;  B = 50;  q = 10;  pourcent_nS = 0.5;  threshold = 0.9;  numerical_method = ODE
 
LAMBDA FIXED PARAMETERS: lam = theoritical;  theoritical_lam = 0.3988;  numerical_method = ODE

SPEEDNESS : 
Running time for Cross Validation    : 1.013s
Running time for Stability Selection : 2.281s
Running time for Fixed LAM           : 0.065s
```


![Ex2.1](figures/example2/Figure1.png)

![Ex2.2](figures/example2/Figure2.png)

![Ex2.3](figures/example2/Figure3.png)

![Ex2.4](figures/example2/Figure4.png)

![Ex2.5](figures/example2/Figure5.png)


## Example on microbiome data

Here is now the result of running the file "example_COMBO" which uses microbiome data :  
```
FORMULATION : Concomitant
 
MODEL SELECTION COMPUTED :  Path,  Stability selection, Lambda fixed
 
STABILITY SELECTION PARAMETERS: method = lam;  lamin = 0.01;  lam = theoritical;  B = 50;  q = 10;  percent_nS = 0.5;  threshold = 0.7;  numerical_method = ODE
 
LAMBDA FIXED PARAMETERS: lam = theoritical;  theoritical_lam = 19.1709;  numerical_method = ODE
 
PATH PARAMETERS: Npath = 40  n_active = False  lamin = 0.011220184543019636;  numerical_method = ODE
 objc[46200]: Class FIFinderSyncExtensionHost is implemented in both /System/Library/PrivateFrameworks/FinderKit.framework/Versions/A/FinderKit (0x7fff96e66b68) and /System/Library/PrivateFrameworks/FileProvider.framework/OverrideBundles/FinderSyncCollaborationFileProviderOverride.bundle/Contents/MacOS/FinderSyncCollaborationFileProviderOverride (0x116315cd8). One of the two will be used. Which one is undefined.
SELECTED PARAMETERS : 
27  Clostridium
SIGMA FOR LAMFIXED  :  8.43571426081596
SPEEDNESS : 
Running time for Path computation    : 0.057s
Running time for Cross Validation    : 'not computed'
Running time for Stability Selection : 1.002s
Running time for Fixed LAM           : 0.028s
 
 
FORMULATION : Concomitant_Huber
 
MODEL SELECTION COMPUTED :  Path,  Stability selection, Lambda fixed
 
STABILITY SELECTION PARAMETERS: method = lam;  lamin = 0.01;  lam = theoritical;  B = 50;  q = 10;  percent_nS = 0.5;  threshold = 0.7;  numerical_method = ODE
 
LAMBDA FIXED PARAMETERS: lam = theoritical;  theoritical_lam = 19.1709;  numerical_method = ODE
 
PATH PARAMETERS: Npath = 40  n_active = False  lamin = 0.011220184543019636;  numerical_method = ODE
 SELECTED PARAMETERS : 
SIGMA FOR LAMFIXED  :  6.000336772926475
SPEEDNESS : 
Running time for Path computation    : 18.517s
Running time for Cross Validation    : 'not computed'
Running time for Stability Selection : 3.166s
Running time for Fixed LAM           : 0.065s


```


![Ex3.1](figures/exampleCOMBO/path.png)

![Ex3.2](figures/exampleCOMBO/sigma.png)

![Ex3.3](figures/exampleCOMBO/distr.png)

![Ex3.4](figures/exampleCOMBO/beta.png)

![Ex3.5](figures/exampleCOMBO/path_huber.png)

![Ex3.6](figures/exampleCOMBO/sigma_huber.png)

![Ex3.7](figures/exampleCOMBO/distr_huber.png)

![Ex3.8](figures_exampleCOMBO/beta_huber.png)


Here is now the result of running the file "example_PH" which uses microbiome data : 
```
FORMULATION : Concomitant
 
MODEL SELECTION COMPUTED :  Path,  Stability selection, Lambda fixed
 
STABILITY SELECTION PARAMETERS: method = lam;  lamin = 0.01;  lam = theoritical;  B = 50;  q = 10;  percent_nS = 0.5;  threshold = 0.7;  numerical_method = ODE
 
LAMBDA FIXED PARAMETERS: lam = theoritical;  theoritical_lam = 19.1991;  numerical_method = ODE
 
PATH PARAMETERS: Npath = 500  n_active = False  lamin = 0.05  n_lam = 500;  numerical_method = ODE


SIGMA FOR LAMFIXED  :  0.7473015322224758
SPEEDNESS : 
Running time for Path computation    : 0.08s
Running time for Cross Validation    : 'not computed'
Running time for Stability Selection : 1.374s
Running time for Fixed LAM           : 0.024s
```

![Ex4.1](figures/examplePH/Path.png)

![Ex4.2](figures/examplePH/Sigma.png)

![Ex4.3](figures/examplePH/Sselection.png)

![Ex4.4](figures/examplePH/beta.png)


## Details on the objects of the package : 

### Type classo_problem : 

Those objected will contains all the information about the problem
 
 #### 5 main attributes :
   - data (type : classo_data): 
   the matrices X, C, y to solve a problem of type : <img src="https://latex.codecogs.com/gif.latex?y&space;=X\beta&space;+\sigma&space;\epsilon&space;\qquad\txt{st.}\qquad&space;C\beta=0" /> 
     
   - formulation (type : classo_formulation) : 
   to know the formulation of the problem, robust ?  ; Jointly estimate sigma (Concomitant) ? , classification ? Default parameter is only concomitant.
       
   - model_selection (type : classo_model_selection) : 
   Path computation ; Cross Validation ; stability selection ; or Lasso problem for a fixed lambda. also contains the parameters of each of those model selection.
   
   - solution (type : classo_solution) : 
   Type that contains the informations about the solution once it is computed. 
   This attribute exists only if the method solve() has been applied to the object problem.
   
   - optional: label (type : list, or numpy array, or boolean False by default) : 
   gives the labels of each variable, and can be set to False (default value) if no label is given.
       
#### 3 methods :

   - init : classo_problem(X=X,y=y,C=C, label=False) will create the object, with its default value, with the good data. 
   If C is not specified, it is set to "zero-sum" which make zero sum contraint. 
   
   - repr : this method allows to print this object in a way that it prints the important informations about what we are solving. 
   
   - solution : once we used the method .solve() , this componant will be added, with the solutions of the model-selections selected, with respect to the problem formulation selected


### Type data :
#### 4 attributes :
  - rescale (type : boolean) : True if regression has to be done after rescaling the data. Default value : False
  - X , y , C (type : numpy.array) : matrices representing the data of the problem.
  
### Type formulation :
#### 6 attributes :
  - huber (type : boolean) : True if the formulation of the problem should be robust
  Default value = False
  
  - concomitant (type : boolean) : True if the formulation of the problem should be with an M-estimation of sigma.
  Default value = True
  
  - classification (type : boolean) : True if the formulation of the problem should be classification (if yes, then it will not be concomitant)
  Default value = False
  
  - rho (type = float) : Value of rho for robust problem. 
  Default value = 1.345
  
  - rho_classification (type = float) : value of rho for huberized hinge loss function for classification (this parameter has to be negative).
  Default value = -1.
  
  - e (type = float or string)  : value of e in concomitant formulation.
  If 'n/2' then it becomes n/2 during the method solve(), same for 'n'.
  Default value : 'n' if huber formulation ; 'n/2' else

### Type model_selection : 
#### 8 attributes :
  - PATH (type : boolean): True if path should be computed. 
  Default Value = False
  
  - PATHparameters (type : PATHparameters): 
  object with as attributes : 
    - numerical_method ; 
    - n_active ; lambdas ;
    - plot_sigma

  
  - CV (type : boolean):  True if Cross Validation should be computed. 
  Default Value = False
  
  - CVparameters (type : CVparameters): 
  object with as attributes : 
    - seed
    - numerical_method
    - lambdas
    - oneSE
    - Nsubsets

  
  - StabSel (type : boolean):  True if Stability Selection should be computed. 
  Default Value = True
  
  - StabSelparameters (type : StabSelparameters): 
  object with as attributes : 
    - seed
    - numerical_method
    - method
    - B
    - q
    - percent_nS
    - lamin
    - hd
    - lam
    - true_lam
    - threshold
    - threshold_label
    - theoritical_lam
  
  - LAMfixed (type : boolean):  True if solution for a fixed lambda should be computed. 
  Default Value = False
  
  - LAMfixedparameters (type : LAMparameters): 
  object with as attributes : 
    - numerical_method
    - lam
    - true_lam
    - theoritical_lam
  

### Type solution :
#### 4 attributes : 
  - PATH (type : solution_PATH): object with as attributes : 
    
    - BETAS
    - SIGMAS
    - LAMBDAS
    - method
    - save
    - formulation
    - time
  
  - CV (type : solution_CV): object with as attributes :
    - beta
    - sigma 
    - xGraph
    - yGraph
    - standard_error
    - index_min
    - index_1SE
    - selected_param
    - refit
    - formulation
    - time
  
  - StabSel (type : solution_StabSel) : object with as attributes :
    - distribution
    - lambdas_path
    - selected_param
    - to_label
    - refit
    - formulation
    - time 
    
  
  - LAMfixed (type : solution_LAMfixed) : object with as attributes :
    - beta
    - sigma
    - lambdamax
    - selected_param
    - refit
    - formulation
    - time


## Optimization schemes

The different problem formulations require different algorithmic strategies for 
efficiently solving the underlying optimization problem. We have implemented four 
algorithms (which have provable convergence guarantee) 
that vary in generality and are not necessarily applicable to all problems.
For each problem type, c-lasso has a default algorithm setting that proved to be the fastest
in preliminary numerical experiments.

### Path algorithms (see, e.g., [1])
From the KKT conditions, we can derive an simple ODE for the solution of
the non concomitants problems, which shows that the solution is piecewise-
affine. For the least square, as the problem can always be reported to a a non
concomitant problem for another lambda, one can use the whole non-concomitant-
path computed with the ODE method to then solve the concomitant-path.

### Projected primal-dual splitting method [2]:
Standard way to solve a convex minimisation problem with an addition of
smooth and non-smooth function : Projected Proximal Gradient Descent. This
method only works with the two non concomitants problems. For the huber
problem, we use the second formulation.

### Projection-free primal-dual splitting method:
Similar to the Projected Proximal Gradient Descent, but which does not involve
a projection, which can be difficult to compute for some matrix C. Only for
non concomitant problems.

### Douglas-Rachford-type splitting method [4,5]
Use of Doulgas Rachford splitting algorithm which use the proximal operator of
both functions. It also solves concomitant problems, but it is usefull even in the
non concomitant case because it is usually more efficient than forward backward
splitting method. For the huber problem, we use the second formulation, then
we change it into a Least square problem of dimension m (m + d) instead of m d.



## References 

* [1] B. R. Gaines, J. Kim, and H. Zhou, [Algorithms for Fitting the Constrained Lasso](https://www.tandfonline.com/doi/abs/10.1080/10618600.2018.1473777?journalCode=ucgs20), J. Comput. Graph. Stat., vol. 27, no. 4, pp. 861–871, 2018.

* [2] L. Briceno-Arias, S.L. Rivera, [A Projected Primal–Dual Method for Solving Constrained Monotone Inclusions](https://link.springer.com/article/10.1007/s10957-018-1430-2?shared-article-renderer), J. Optim. Theory Appl., vol 180, Issue 3 March 2019

* [3] 

* [4] P. L. Combettes and C. L. Müller, [Perspective M-estimation via proximal decomposition](https://arxiv.org/abs/1805.06098), Electronic Journal of Statistics, 2020, [Journal version](https://projecteuclid.org/euclid.ejs/1578452535) 

* [5] P. L. Combettes and C. L. Müller, [Regression models for compositional data: General log-contrast formulations, proximal optimization, and microbiome data applications](https://arxiv.org/abs/1903.01050), arXiv, 2019.
