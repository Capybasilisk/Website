---
title: "Square Neighborhood Algorithm: Balancing Exploration And Exploitation In Optimization"
date: 2020-06-21T12:06:29Z
---

A central problem in optimization and artificial intelligence is how an agent, faced with
a large solution space and finite resources, should divide its
time between exploiting parts of the space known to contain good solutions, and exploring unknown 
regions that may contain solutions far better, or much worse, than those found so far.

An intuitive example of the problem is humans choosing entertainment options. When deciding what
music, films, literature, games etc. to consume, people tend to "stick with what they know," i.e.
options of similar type and genre, and therefore close together in the option space. Trying unfamiliar 
parts of the option space has a low likelihood of a high payoff, and a high likelihood of wasted time.

With time being a precious resource for all agents, the temptation is to only look at parts of the solution
space guaranteed to provide good values, but this strategy risks missing out on solutions with values 
potentially far greater than any contained within familiar regions of the space.

Defined here is a variant of the *random*, *neighborhood search*, and *hill climbing* classes of algorithms. It is a
highly explorative strategy in which *the amount of time spent looking at a solution's "neighborhood" is a direct
function of the newly found best solution's magnitude of improvement over that of the previous best*. In other words, 
the greater the difference in value between the new best solution and the last best solution, the longer the algorithm
will spend looking for even better options in the new best solution's neighborhood.

When searching a new best solution's neighborhood, the algorithm begins with values very close to the best solution, but
begins expanding the size of the neighborhood the longer it goes without finding any improvements. If the variable amount of time
allotted to the neighborhood search phase elapses without any improvements, the algorithm reverts back to a pure random search.

The algorithm was designed to optimize machine learning model parameters, but it can be applied to any numerical optimization
problem.

Below is an outline of the algorithm, and below that is a Python implementation of the algorithm tuning the parameters of a 
LightGBM prediction model.

&nbsp;
&nbsp;


### Square Neighborhood Algorithm

``` Python


    import pandas as pd
    from lightgbm import LGBMRegressor
    from sklearn.metrics import mean_absolute_error
    from sklearn.model_selection import train_test_split
    from random import choice, randrange, uniform, gauss
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


    #Build and validate the learning model
    def model_check(parameters):
        
        local_copy ={
        key:value for key,value in parameters.items()} 
        
        if local_copy["boosting_type"] == 0:
            local_copy["boosting_type"] = "gbdt"
        else:
            local_copy["boosting_type"] = "dart"
            
        model = LGBMRegressor().set_params(**local_copy)
        model.fit(X_train, y_train)
        prediction = model.predict(X_valid)
        error_rate = mean_absolute_error(
            y_valid, prediction)

        return error_rate

    #square_neighborhood algorithm
    def square_neighborhood():
        
        global_best = 17000
        best_parameters = {}
        neighborhood_search = 0
        
        while True:
            
            try:
                
                random_parameters = {
                    "boosting_type": randrange(2),
                    "n_estimators": randrange(10000),
                    "num_leaves": randrange(10000),
                    "learning_rate": uniform(0.01, 1)}
                
                global_evaluate = model_check(
                    random_parameters)
                
                if global_evaluate < global_best:
                    
                    neighborhood_search += round(
                        abs(
                            global_evaluate - global_best) ** 2)
                    
                    global_best = global_evaluate
                    best_parameters = random_parameters
                    neighborhood_steps = 0
                    neighborhood_size = 0.1
                    
                    with open(
                        "values.csv", "a", encoding = "UTF-8") as valuefile:
                        values = clevercsv.writer(valuefile)
                        values.writerow(
                            [global_best, best_parameters])
                        
                   
                    while neighborhood_search > 0:
                    
                        neighborhood_parameters = {
                            key:abs(
                                gauss(
                                    value, 
                                    neighborhood_size)) if type(value) == float 
                            else abs(
                                round(
                                    gauss(
                                        value, 
                                        neighborhood_size))) 
                            for key,value in best_parameters.items()}

                        neighborhood_evaluate = model_check(
                            neighborhood_parameters)
                        
                        if neighborhood_evaluate < global_best:
                            
                            neighborhood_search += abs(
                                neighborhood_evaluate - global_best) ** 2
                            
                            global_best = neighborhood_evaluate
                            best_parameters = neighborhood_parameters
                            neighborhood_steps = 0
                            neighborhood_size = 0.1
                            
                            with open(
                                "values.csv", 
                                "a", encoding = "UTF-8") as valuefile:
                                values = clevercsv.writer(valuefile)
                                values.writerow(
                                    [global_best, best_parameters])
                        
                        else:
                            
                            neighborhood_steps += 1
                            neighborhood_size += 0.0001 * neighborhood_steps
                        
                        neighborhood_search -= 1                  
                            
            except:
        
                continue
        
        
    square_neighborhood()



```


&nbsp;
&nbsp;

[Full code repo available on Github.](https://github.com/Capybasilisk/Optimizers)

&nbsp;

[UPDATE: I've formulated an improved optimizer called the Interleaved Neighborhood Algorithm. You can read about it here.](https://capybasilisk.com/posts/2020/06/interleaved-neighborhood-algorithm-fully-exploratory-optimization/)

&nbsp;

Related posts:

[Speculative Fiction Bot](https://capybasilisk.com/posts/2020/04/speculative-fiction-bot/)

[EXP-RTL: Exponential Retaliation In Iterated Prisoner's Dilemma Games](https://capybasilisk.com/posts/2020/04/exp-rtl-exponential-retaliation-in-iterated-prisoners-dilemma-games)

&nbsp;

[About Me](https://capybasilisk.com/about/)
