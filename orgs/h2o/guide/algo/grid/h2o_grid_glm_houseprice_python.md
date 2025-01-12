# H2O Grid (Hyperparameter) Search for GLM in Python #

Hyperparameter Optimization is the process of setting of all combinations of values for these knobs is called the hyperparameter space. We'd like to find a set of hyperparameter values which gives us the best model for our data in a reasonable amount of time. This process is called hyperparameter optimization. Learn more about gird search in H2O [here](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/grid-search.html).

## Dataset ##
The dataset used in this example can be obtained from here:
 - [house_price_train.csv](https://raw.githubusercontent.com/Avkash/mldl/master/data/house_price_train.csv)
 - [house_price_test.csv](https://raw.githubusercontent.com/Avkash/mldl/master/data/house_price_test.csv)

Note: Use "wget" and above links to pull the the data locally or use the URL above directly to load data into H2O.
  
## Get the Sample Python Notebook ##
  - [H2O GLM Regression with House Price Dataset Notebook](https://github.com/Avkash/mldl/blob/master/notebook/h2o/H2O-GridSearch-GLM-HousePrice-Regression.ipynb)
  
## GLM Regression Grid Search Sample in Python ##

###: Loading H2O Library
```python
import h2o
```

###: Get H2O Version
```python
h2o.__version__
```

###: Initalizing H2O cluster
```python
h2o.init()
```

###: Importing both training and test dataset into H2O cluster memory
```python
train_df = h2o.import_file("https://raw.githubusercontent.com/Avkash/mldl/master/data/house_price_train.csv")
test_df = h2o.import_file("https://raw.githubusercontent.com/Avkash/mldl/master/data/house_price_test.csv")
```

###: Understanding Training dataset
```python
train_df.describe()
```

###: Understanding Test dataset
```python
test_df.describe()
```

###: Training and test dataset - columns and rows details
```python
print(train_df.shape)
print(test_df.shape)
```

###: Training and Test Dataframes - columns names
```python
print(train_df.col_names)
print(test_df.col_names)
```

###: Settings response or target variable for supervised machine learning
```python
response = "medv"
features = train_df.col_names
print(features)
```

###: Creating a list of all features we will use for machine learning
```python
features.remove(response)
print(features)
```

###: Understanding response variable values as historgram in Training data
```python
train_df['medv'].hist()
```

###: Importing H2O H2OGeneralizedLinearEstimator to build GLM Model
```python
from h2o.estimators.glm import H2OGeneralizedLinearEstimator
```

###:Building Gradient Boosting (GBM) -  Regression model with cross validation
```python
glm_model_with_cv = H2OGeneralizedLinearEstimator(nfolds=5)
glm_model_with_cv.train(x = features, y = response, training_frame=train_df)
```

###: Getting model performance
```python
glm_model_with_cv.model_performance(valid=True,test_data=test_df).r2()
```

###:Building GLM -  Regression model with cross validation andkey GBM parameters configuration
```python
glm_model_cv_config = H2OGeneralizedLinearEstimator(nfolds=5,
                                                    keep_cross_validation_predictions=True,
                                                    lambda_search = True,
                                                    alpha = 0.1,
                                                    seed=1)
```

###: Training GBM Model
```python
glm_model_cv_config.train(x = features, y = response, 
                                            training_frame=train_df, 
                                           model_id = "glm_model_with_training_and_validtion_python")
```

###: Getting GLM model performance on test data
```python
glm_model_cv_config.model_performance(valid=True,test_data=test_df).r2()
```

###: Importing H2O Grid Library
```python
from h2o.grid import H2OGridSearch
```

###: Settings GLM grid parameters
```python
glm_hyper_params = { 'alpha': [0.01,0.1,0.3,0.5,0.7,0.9], 
                     'lambda': [1e-1,1e-3,1e-5,1e-7,1e-9] }
```

###: Setting H2O Grid Search Criteria
```python
grid_search_criteria = { 'strategy': "RandomDiscrete", 
                    'seed': 123,
                    'stopping_metric': "AUTO", 
                    'stopping_tolerance': 0.01,
                    'stopping_rounds': 5 }
```

###: Finalzing the H2I Grid searching settings
```python
house_price_glm_grid = H2OGridSearch(model=H2OGeneralizedLinearEstimator(
                                                        seed=12345,
                                                        nfolds=5,
                                                        fold_assignment="Modulo",
                                                        keep_cross_validation_predictions=True),
                     hyper_params=glm_hyper_params,
                     search_criteria=grid_search_criteria,
                     grid_id="house_price_glm_grid")
```

###: Finally training H2O Grid with data 
```python
house_price_glm_grid.train(x=features, y=response, training_frame=train_df)
```

###: Finally getting total count of GLM models
```python
len(house_price_glm_grid)
```

###: Defining a function to find the best model from the grid based on r2 or auc
```python
def find_best_model_from_grid(h2o_grid, test_parameter):    
    model_list = []
    for grid_item in h2o_grid:
        if test_parameter is "r2":
            if not (grid_item.r2() == "NaN"):
                model_list.append(grid_item.r2())
            else:
                model_list.append(0.0)            
        elif test_parameter is "auc":
            if not (grid_item.auc() == "NaN"):
                model_list.append(grid_item.auc())
            else:
                model_list.append(0.0)            
    #print(model_list)        
    max_index = model_list.index(max(model_list))
    #print(max_index)
    best_model = h2o_grid[max_index]
    print("Model ID with best R2: " +  best_model.model_id)
    if test_parameter is "r2":
        print("Best R2: " +  str(best_model.r2()))
    elif test_parameter is "auc":
        print("Best AUC: " +  str(best_model.auc()))
    return best_model
```

###: Applying the function to get the best model from the grid
```python
best_model = find_best_model_from_grid(house_price_glm_grid, "r2")
```

###: Getting the best model performance on test data
```python
best_model.model_performance(valid=True,test_data=test_df).r2()
```

###: Performing predictions with one of the above model
```python
glm_predictions = best_model.predict(test_df)
glm_predictions
```

###: Understanding/Validating predictions based on prediction results historgram
```python
glm_predictions.hist()
```

###: Getting Scorring History
```python
best_glm_model.scoring_history()
```

###: Getting GBM model variable importance 
```python
best_glm_model.varimp()
```

###: Getting GBM model variable importance PLOT
```python
best_glm_model.varimp_plot()
```
