---
title: "Interleaved Neighborhood Algorithm: Fully Exploratory Optimization"
date: 2020-06-30T19:37:52Z
---

[In my previous post](https://capybasilisk.com/posts/2020/06/square-neighborhood-algorithm-balancing-exploration-and-exploitation-in-optimization/), I talked about the exploration/exploitation dilemma in optimization, and then 
defined an algorithm that seeks to resolve it. The algorithm described in that post was a variation
of neighborhood search and hill climbing methods. The amount of time it spends looking for
improvements in the "neighborhood" of a given solution is a direct function of the magnitude of 
improvement that a current best solution demonstrates over the previous best.

Large improvements cause the algorithm to remain in the current best solution's neighborhood for significant 
lengths of time. While this is a good thing if the neighborhood is rich in potential improvements,
it presents a problem if the neighborhood turns out to be sparse in improving solutions.

To counter this, I outline another, far more exploratory, numerical optimization algorithm. This
algorithm symetrically interleaves its exploration and exploitation steps, dividing its time equally
betweem searching the neighborhood of the current global best, and searching the global solution space
in a purely random pattern.

After initially evaluating a solution at random and setting its score as the global best, the algorithm
evaluates another solution in the global best's neighborhood. If this solution's score is better than that 
of the global best, it becomes the new global best, otherwise the optimizer tries a protential solution chosen at 
random from the entire search space. If this solution is better than global best, it becomes the global best and 
the search resumes from its neighborhood, otherwise the search returns to the current global best's neighborhood to 
try another solution at random.

In this way, on every other step, the algorithm bounces between searching the whole space randomly and a much more
focussed search in the global best's neighborhood.


The algorithm was designed to optimize machine learning model parameters, but it can be applied to any numerical optimization
problem.

Below is an outline of the algorithm, and below that is a Python implementation of the algorithm tuning the parameters of a 
LightGBM prediction model.




### Interleaved Neighborhood Algorithm

```

 evaluate random solution rs

 set var global_best to rs score 
 set var best_parameters to rs parameters
 set var horizon to "local"

 while true do
      
       if horizon = "local"
        
          neighborhood solution ns = gaussian distr(mean = best_parameters, deviation = random(0.1...1.0))
        
          if ns better than global_best
           
             set var global_best to ns score 
             set var best_parameters to ns parameters
             continue
        
          else
            
             set var horizon to "global"
             coninue
        
       else
      
           evaluate global random solution gs
          
           if gs score better than global_best
             
              set var global_best to gs score 
              set var best_parameters to gs parameters
              set var horizon to "local"
              continue
          
           else:
               
              set var horizon to "local"
              continue
             
```

&nbsp;
&nbsp;


### Optimizing A LightGBM Machine Learning Model

The Python code below uses the Interleaved Neighborhood Algorithm to search through the space of parameters
of a LightGBM machine learning model. The model is trained on, and used to make predictions about, the Kaggle
Iowa housing dataset.

LightGBM is a gradient boosting framework with a large number of parameters, each of which can take a huge range of values. This makes parameter optimization a nontrivial problem. For simplicity, the demonstration below uses only a few of the dozens of model parameters available for tuning.

&nbsp;
&nbsp;



``` Python

import numpy as np
import pandas as pd
from lightgbm import LGBMRegressor
from sklearn.metrics import mean_absolute_error
from sklearn.model_selection import train_test_split
from random import randrange, uniform, gauss
import clevercsv


#Read datasets into memory
complete_train = pd.read_csv(
    "train.csv",
    encoding = "UTF-8", 
    index_col = "Id")

complete_test = pd.read_csv(
    "test.csv",
    encoding = "UTF-8",
    index_col = "Id")


#Seperate predictors from target variable
X = complete_train.drop(
    columns = "SalePrice")

y = complete_train[
    "SalePrice"]


#Encode categoricals and impute missing data
def encode_impute(*datasets):
    for dataset in datasets:
        for column in dataset.columns:
            dataset[
                column].fillna(
                -999,
                inplace = True)
            if dataset[
                column].dtype ==  "object":
                dataset[
                    column] = dataset[
                    column].astype("category", copy = False)

encode_impute(X)


#Create validation set
X_train, X_valid, y_train, y_valid = train_test_split(X, y)


#Model evaluation
def model_check(parameters):
    
    local_copy ={
        key:value for key,value in parameters.items()}
    
    model = LGBMRegressor().set_params(**local_copy)
    model.fit(X_train, y_train)
    prediction = model.predict(X_valid)
    error_rate = mean_absolute_error(y_valid, prediction)
    
    return {"score": error_rate, "parameters": parameters}


#Parameter generation
def param_gen():
    
    k = randrange(10000)
    
    parameters = {
       "boosting_type": "dart",
       "n_estimators": k,
       "num_leaves": randrange(round(k/3 * 2)),
       "learning_rate": uniform(0.01, 1)}
    
    return parameters



#square_neighborhood algorithm
def interleaved_neighborhood():
    
    initialize = model_check(param_gen())
    
    global_best = {
        "score": initialize["score"], 
        "parameters": initialize["parameters"]}
    
    with open(
        "values.csv", "a", encoding = "UTF-8") as valuefile:
        values = clevercsv.writer(valuefile)
        values.writerow(
            [global_best["score"], 
            global_best["parameters"]])
    
    horizon = "local"
    
    
    while True:
        
        try:
            
            if horizon == "local":
                
                perturb = {
                    key:abs(gauss(value, uniform(0.1, 2.0))) 
                    if type(value) != str 
                    else value for key,value 
                    in global_best["parameters"].items()}
                
                rounded = {
                    key:round(value) 
                    if key in ["n_estimators", "num_leaves"]
                    else value for key,value in perturb.items()}
                
                local_solution = model_check(rounded)
                
                if local_solution["score"] < global_best["score"]:
                    
                    with open(
                        "values.csv", "a", encoding = "UTF-8") as valuefile:
                        values = clevercsv.writer(valuefile)
                        values.writerow(
                            [local_solution["score"], 
                            local_solution["parameters"]])
                    
                    global_best["score"] = local_solution["score"]
                    global_best["parameters"] = local_solution["parameters"]
                    
                    continue
               
                
                else:
                    
                    horizon = "global"
                     
                    continue
                        
                        
            else:
                
                random_solution = model_check(param_gen())
                
                if random_solution["score"] < global_best["score"]:
                    
                    with open(
                        "values.csv", "a", encoding = "UTF-8") as valuefile:
                        values = clevercsv.writer(valuefile)
                        values.writerow(
                            [random_solution["score"], 
                            random_solution["parameters"]])
                    
                    global_best["score"] = random_solution["score"]
                    global_best["parameters"] = random_solution["parameters"]
                    
                    horizon = "local"
                    
                    continue
                
                else:
                    
                    horizon = "local"
                    
                    continue
                    
        
        
        except Exception as error:
            
            print(error)
    
    

    

interleaved_neighborhood()




```



&nbsp;
&nbsp;


[Full code repo available on Github](https://github.com/Capybasilisk/Optimizers)

&nbsp;
&nbsp;

Related posts:

[Square Neighborhood Algorithm: Balancing Exploration And Exploitation In Optimization](https://capybasilisk.com/posts/2020/06/square-neighborhood-algorithm-balancing-exploration-and-exploitation-in-optimization/)

[Speculative Fiction Bot](https://capybasilisk.com/posts/2020/04/speculative-fiction-bot/)

[EXP-RTL: Exponential Retaliation In Iterated Prisoner's Dilemma Games](https://capybasilisk.com/posts/2020/04/exp-rtl-exponential-retaliation-in-iterated-prisoners-dilemma-games/)

&nbsp;
&nbsp;

[About Me](https://capybasilisk.com/about/)
