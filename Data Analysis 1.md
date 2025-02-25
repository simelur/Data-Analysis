```python
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split, GridSearchCV
from sklearn.datasets import make_regression
from sklearn.ensemble import RandomForestRegressor
from sklearn.feature_selection import RFE
from sklearn.metrics import mean_squared_error, r2_score
from sklearn.preprocessing import StandardScaler
from sklearn.impute import SimpleImputer
from sklearn.pipeline import Pipeline
import matplotlib.pyplot as plt
# Define the pipeline
pipeline = Pipeline([
    ('imputer', SimpleImputer(strategy='mean')),
    ('scaler', StandardScaler()),
    ('model', RandomForestRegressor())
])
#1. Data Collection and Pre-processing
# Load data (replace with your actual file names)
df = pd.read_csv('PENICAL ALL.csv')

# Split dataframe into features (X) and target (y)
X = df.drop(columns=['SPAD'])  # Drop 'sample_id' and 'SPAD' to get the features
y = df['SPAD']  # Target variable

# Check for missing values
print(df.isnull().sum())

```


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    ~\AppData\Local\Temp\1\ipykernel_21104\3175347921.py in <module>
    ----> 1 import pandas as pd
          2 import numpy as np
          3 from sklearn.model_selection import train_test_split, GridSearchCV
          4 from sklearn.datasets import make_regression
          5 from sklearn.ensemble import RandomForestRegressor
    

    C:\ProgramData\Anaconda3\lib\site-packages\pandas\__init__.py in <module>
         20 
         21 # numpy compat
    ---> 22 from pandas.compat import is_numpy_dev as _is_numpy_dev
         23 
         24 try:
    

    C:\ProgramData\Anaconda3\lib\site-packages\pandas\compat\__init__.py in <module>
         13 
         14 from pandas._typing import F
    ---> 15 from pandas.compat.numpy import (
         16     is_numpy_dev,
         17     np_version_under1p19,
    

    C:\ProgramData\Anaconda3\lib\site-packages\pandas\compat\numpy\__init__.py in <module>
          2 import numpy as np
          3 
    ----> 4 from pandas.util.version import Version
          5 
          6 # numpy versioning
    

    C:\ProgramData\Anaconda3\lib\site-packages\pandas\util\__init__.py in <module>
    ----> 1 from pandas.util._decorators import (  # noqa:F401
          2     Appender,
          3     Substitution,
          4     cache_readonly,
          5 )
    

    C:\ProgramData\Anaconda3\lib\site-packages\pandas\util\_decorators.py in <module>
         12 import warnings
         13 
    ---> 14 from pandas._libs.properties import cache_readonly  # noqa:F401
         15 from pandas._typing import F
         16 
    

    C:\ProgramData\Anaconda3\lib\site-packages\pandas\_libs\__init__.py in <module>
         11 
         12 
    ---> 13 from pandas._libs.interval import Interval
         14 from pandas._libs.tslibs import (
         15     NaT,
    

    C:\ProgramData\Anaconda3\lib\site-packages\pandas\_libs\interval.pyx in init pandas._libs.interval()
    

    ValueError: numpy.dtype size changed, may indicate binary incompatibility. Expected 96 from C header, got 88 from PyObject



```python
#Explore data statistics
df.describe()
```


```python
#1. Pre-processing-Normalization
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
standardized_df = scaler.fit_transform(df)
X_train_scaled = scaler.fit_transform(df)
X_test_scaled = scaler.transform(df)

# Normalize spectral data
df_normalized = (df- df.mean()) / df.std()
```


```python
# Apply Savitzky-Golay filter # smoothing
from scipy.signal import savgol_filter
smoothed_df = savgol_filter(df_normalized, window_length=11, polyorder=3, axis=0)

# Calculate correlation coefficients #3. Data Analysis
correlation_matrix = df.corr()

# Extract correlations with SPAD
correlations_with_spad = correlation_matrix['SPAD'].drop('SPAD')

# Display correlation values
print(correlations_with_spad)
```


```python
# Measures the statistical relationship between two features.
# Sort correlations with SPAD #Strongest linear relationships with the target variable. 
sorted_correlations = correlations_with_spad.abs().sort_values(ascending=False)
print("Top features correlated with SPAD:")
print(sorted_correlations.head(10))
```


```python
# 4. Split the data into training and testing sets
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=175)
```


```python
#4. cross-validation
#Robust Performance Estimate: It provides a more reliable estimate of the model's performance 
#by training and testing the model on different subsets of the data.
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import cross_val_score
import numpy as np

# Example data (replace with your dataset)
X = np.random.rand(100, 10)  # 100 samples, 10 features
y = np.random.rand(100)      # 100 samples, 1 target variable
# Initialize the Random Forest Regressor
model = RandomForestRegressor(n_estimators=100, random_state=175)

# Perform 5-fold cross-validation
# Note: cross_val_score returns negative MSE values by default
mse_scores = cross_val_score(model, X, y, scoring='neg_mean_squared_error', cv=5)

# Convert negative MSE to positive MSE
mse_scores = -mse_scores

# Calculate the average MSE
average_mse = np.mean(mse_scores)

# Print the results
print("MSE scores for each fold:", mse_scores)
print("Average MSE:", average_mse)
```


```python
import matplotlib.pyplot as plt
from sklearn.decomposition import PCA
# Standardize the features
X_scaled = scaler.fit_transform(X)
# Apply PCA
n_components = 10  # Number of principal components to keep
pca = PCA(n_components=n_components)
X_pca = pca.fit_transform(X_scaled)

# Verify the shape of the transformed data
print("Shape of X_pca:", X_pca.shape)
# Fit PCA
pca = PCA()
pca.fit(X_scaled)

# Plot explained variance ratio
plt.figure(figsize=(6, 2))
plt.plot(np.cumsum(pca.explained_variance_ratio_), marker='o')
plt.xlabel('Number of Components')
plt.ylabel('Cumulative Explained Variance')
plt.title('Explained Variance by Number of Components')
plt.grid(True)
plt.show()

# Print the explained variance ratio for each component
print("Explained variance ratio by each component:")
print(pca.explained_variance_ratio_)
```


```python
# Define the parameter grid
param_grid = {
    'n_estimators': [50, 100, 150, 200,],
    'max_depth': [5, 10, 15, 20],
    'min_samples_split': [30,35,40,45,50],
    'min_samples_leaf': [7,8,9,10, 11],
    'max_features': [1, 2, 3, 4, 5]  # Number of features to consider at each split
}

# Initialize the Random Forest Regressor
rf = RandomForestRegressor(random_state=175)

# Initialize GridSearchCV
grid_search = GridSearchCV(estimator=rf, param_grid=param_grid, cv=5, scoring='neg_mean_squared_error', n_jobs=-1)

# Fit the model
grid_search.fit(X, y)

# Get the best parameters
best_params = grid_search.best_params_
print("Best parameters found: ", best_params)

# Get the best model
best_rf = grid_search.best_estimator_

# Evaluate the best model
best_mse_scores = cross_val_score(best_rf, X, y, scoring='neg_mean_squared_error', cv=5)
best_mse_scores = -best_mse_scores
average_best_mse = np.mean(best_mse_scores)
print("Best Random Forest Cross-validated MSE:", average_best_mse)

```


```python
from sklearn.ensemble import RandomForestRegressor
from sklearn.feature_selection import RFE

# Define the model
rf = RandomForestRegressor(random_state=175)

# Initialize RFE with the Random Forest model and select the top 5 features
rfe = RFE(estimator=rf, n_features_to_select=5, step=25)
rfe.fit(X_train, y_train)

# Get the selected features
selected_features = X_train.columns[rfe.support_]
print(f"Selected features: {selected_features}")

# Transform the data to include only the selected features
X_train_rfe = rfe.transform(X_train)
X_test_rfe = rfe.transform(X_test)

def calculate_metrics(y_true, y_pred):
    r2 = r2_score(y_true, y_pred)
    rmse = np.sqrt(mean_squared_error(y_true, y_pred))
    mae = mean_absolute_error(y_true, y_pred)
    mse = mean_squared_error(y_true, y_pred)
    rpd = np.std(y_true) / rmse
    return r2, rmse, mae, mse, rpd

def display_metrics(model_name, y_train, y_train_pred, y_test, y_test_pred):
    print(f"---{model_name}---")
    print("Train Metrics:")
    train_metrics = calculate_metrics(y_train, y_train_pred)
    print(f"R2: {train_metrics[0]}, RMSE: {train_metrics[1]}, MAE: {train_metrics[2]}, MSE: {train_metrics[3]}, RPD: {train_metrics[4]}")
    
    print("Test Metrics:")
    test_metrics = calculate_metrics(y_test, y_test_pred)
    print(f"R2: {test_metrics[0]}, RMSE: {test_metrics[1]}, MAE: {test_metrics[2]}, MSE: {test_metrics[3]}, RPD: {test_metrics[4]}")
    print("\n")

# Re-train the Random Forest model using only the selected features
rf.fit(X_train_rfe, y_train)
y_train_pred_rf = rf.predict(X_train_rfe)
y_test_pred_rf = rf.predict(X_test_rfe)
display_metrics("Random Forest with RFE", y_train, y_train_pred_rf, y_test, y_test_pred_rf)

```


```python
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense

# RFE and BPNN
# Define the model for RFE
rf = RandomForestRegressor(random_state=175)

# Initialize RFE with the Random Forest model and select the top 5 features
rfe = RFE(estimator=rf, n_features_to_select=5, step=25)
rfe.fit(X_train, y_train)

# Get the selected features
selected_features = X_train.columns[rfe.support_]
print(f"Selected features: {selected_features}")

# Transform the data to include only the selected features
X_train_rfe = rfe.transform(X_train)
X_test_rfe = rfe.transform(X_test)

# Standardize the features
scaler = StandardScaler()
X_train_rfe = scaler.fit_transform(X_train_rfe)
X_test_rfe = scaler.transform(X_test_rfe)

# Define the BPNN model
def create_bpnn(input_dim):
    model = Sequential()
    model.add(Dense(128, input_dim=input_dim, activation='relu'))
    model.add(Dense(64, activation='relu'))
    model.add(Dense(32, activation='relu'))
    model.add(Dense(1))
    model.compile(optimizer='adam', loss='mse')
    return model

# Create and train the BPNN model
bpnn = create_bpnn(X_train_rfe.shape[1])
history = bpnn.fit(X_train_rfe, y_train, epochs=400, batch_size=10, verbose=1, validation_split=0.2)

# Make predictions
y_train_pred_bpnn = bpnn.predict(X_train_rfe).flatten()
y_test_pred_bpnn = bpnn.predict(X_test_rfe).flatten()

def calculate_metrics(y_true, y_pred):
    r2 = r2_score(y_true, y_pred)
    rmse = np.sqrt(mean_squared_error(y_true, y_pred))
    mae = mean_absolute_error(y_true, y_pred)
    mse = mean_squared_error(y_true, y_pred)
    rpd = np.std(y_true) / rmse
    return r2, rmse, mae, mse, rpd

def display_metrics(model_name, y_train, y_train_pred, y_test, y_test_pred):
    print(f"---{model_name}---")
    print("Train Metrics:")
    train_metrics = calculate_metrics(y_train, y_train_pred)
    print(f"R2: {train_metrics[0]}, RMSE: {train_metrics[1]}, MAE: {train_metrics[2]}, MSE: {train_metrics[3]}, RPD: {train_metrics[4]}")
    
    print("Test Metrics:")
    test_metrics = calculate_metrics(y_test, y_test_pred)
    print(f"R2: {test_metrics[0]}, RMSE: {test_metrics[1]}, MAE: {test_metrics[2]}, MSE: {test_metrics[3]}, RPD: {test_metrics[4]}")
    print("\n")

# Display metrics
display_metrics("BPNN with RFE", y_train, y_train_pred_bpnn, y_test, y_test_pred_bpnn)

```


```python
from xgboost import XGBRegressor

# Generate a random regression problem
from sklearn.datasets import make_regression
X, y = make_regression(n_samples=100, n_features=10, noise=0.1, random_state=175)

# Define the model for RFE
xgb_model = XGBRegressor(objective='reg:squarederror', random_state=100)

# Initialize RFE with the XGBoost model and select the top 5 features
rfe = RFE(estimator=xgb_model, n_features_to_select=5, step=25)
rfe.fit(X_train, y_train)

# Get the selected features
selected_features = X_train.columns[rfe.support_]
print(f"Selected features: {selected_features}")

# Transform the data to include only the selected features
X_train_rfe = rfe.transform(X_train)
X_test_rfe = rfe.transform(X_test)

# Standardize the features
scaler = StandardScaler()
X_train_rfe = scaler.fit_transform(X_train_rfe)
X_test_rfe = scaler.transform(X_test_rfe)

# Train the XGBoost model with the selected features
xgb_model.fit(X_train_rfe, y_train)

# Make predictions
y_train_pred_xgb = xgb_model.predict(X_train_rfe)
y_test_pred_xgb = xgb_model.predict(X_test_rfe)

def calculate_metrics(y_true, y_pred):
    r2 = r2_score(y_true, y_pred)
    rmse = np.sqrt(mean_squared_error(y_true, y_pred))
    mae = mean_absolute_error(y_true, y_pred)
    mse = mean_squared_error(y_true, y_pred)
    rpd = np.std(y_true) / rmse
    return r2, rmse, mae, mse, rpd

def display_metrics(model_name, y_train, y_train_pred, y_test, y_test_pred):
    print(f"---{model_name}---")
    print("Train Metrics:")
    train_metrics = calculate_metrics(y_train, y_train_pred)
    print(f"R2: {train_metrics[0]}, RMSE: {train_metrics[1]}, MAE: {train_metrics[2]}, MSE: {train_metrics[3]}, RPD: {train_metrics[4]}")
    
    print("Test Metrics:")
    test_metrics = calculate_metrics(y_test, y_test_pred)
    print(f"R2: {test_metrics[0]}, RMSE: {test_metrics[1]}, MAE: {test_metrics[2]}, MSE: {test_metrics[3]}, RPD: {test_metrics[4]}")
    print("\n")

# Display metrics
display_metrics("XGBoost with RFE", y_train, y_train_pred_xgb, y_test, y_test_pred_xgb)

```


```python
from sklearn.linear_model import ElasticNet

# Generate a random regression problem
from sklearn.datasets import make_regression
X, y = make_regression(n_samples=100, n_features=10, noise=0.1, random_state=175)

# Define the model for RFE
elnet_model = ElasticNet(random_state=100)

# Initialize RFE with the Elastic Net model and select the top 5 features
rfe = RFE(estimator=elnet_model, n_features_to_select=5, step=25)
rfe.fit(X_train, y_train)

# Get the selected features
selected_features = X_train.columns[rfe.support_]
print(f"Selected features: {selected_features}")

# Transform the data to include only the selected features
X_train_rfe = rfe.transform(X_train)
X_test_rfe = rfe.transform(X_test)

# Standardize the features
scaler = StandardScaler()
X_train_rfe = scaler.fit_transform(X_train_rfe)
X_test_rfe = scaler.transform(X_test_rfe)

# Train the Elastic Net model with the selected features
elnet_model.fit(X_train_rfe, y_train)

# Make predictions
y_train_pred_elnet = elnet_model.predict(X_train_rfe)
y_test_pred_elnet = elnet_model.predict(X_test_rfe)

def calculate_metrics(y_true, y_pred):
    r2 = r2_score(y_true, y_pred)
    rmse = np.sqrt(mean_squared_error(y_true, y_pred))
    mae = mean_absolute_error(y_true, y_pred)
    mse = mean_squared_error(y_true, y_pred)
    rpd = np.std(y_true) / rmse
    return r2, rmse, mae, mse, rpd

def display_metrics(model_name, y_train, y_train_pred, y_test, y_test_pred):
    print(f"---{model_name}---")
    print("Train Metrics:")
    train_metrics = calculate_metrics(y_train, y_train_pred)
    print(f"R2: {train_metrics[0]}, RMSE: {train_metrics[1]}, MAE: {train_metrics[2]}, MSE: {train_metrics[3]}, RPD: {train_metrics[4]}")
    
    print("Test Metrics:")
    test_metrics = calculate_metrics(y_test, y_test_pred)
    print(f"R2: {test_metrics[0]}, RMSE: {test_metrics[1]}, MAE: {test_metrics[2]}, MSE: {test_metrics[3]}, RPD: {test_metrics[4]}")
    print("\n")

# Display metrics
display_metrics("Elastic Net with RFE", y_train, y_train_pred_elnet, y_test, y_test_pred_elnet)

```


```python
from xgboost import XGBRegressor
# Initialize the XGBoost regressor
xgb = XGBRegressor(objective='reg:squarederror', random_state=175)

# Initialize RFE with XGBoost as the estimator
rfe = RFE(estimator=xgb, n_features_to_select=10, step=25)

# Fit RFE
rfe.fit(X_train, y_train)

# Get the selected features
selected_features = X_train.columns[rfe.support_]

print(f"Selected features: {selected_features}")

# Train the model with selected features
xgb.fit(X_train[selected_features], y_train)

# Predict on the test set
y_pred = xgb.predict(X_test[selected_features])

# Evaluate the model
mae = mean_absolute_error(y_test, y_pred)
print(f"Mean Absolute Error: {mae}")
# Predict and evaluate
y_pred = stacking_regressor.predict(X_test)
mse = mean_squared_error(y_test, y_pred)
r2 = r2_score(y_test, y_pred)
mae = mean_absolute_error(y_test, y_pred)
print("XGBRegressor MSE:", mse)
print("XGBRegressor R^2 Score:", r2)
print('XGBRegressor Absolute Error:',{mae})

# Display feature importance
importance = pd.DataFrame({
    'Feature': selected_features,
    'Importance': xgb.feature_importances_
}).sort_values(by='Importance', ascending=False)

print(importance)
```


```python
import shap
from xgboost import XGBRegressor
from sklearn.linear_model import ElasticNet
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score

# Generate a random regression problem
X, y = make_regression(n_samples=100, n_features=10, noise=0.1, random_state=100)

# Train the XGBoost model
model = XGBRegressor(objective='reg:squarederror', random_state=25)
model.fit(X_train, y_train)

# Explain the model predictions using SHAP
explainer = shap.Explainer(model)
shap_values = explainer(X_test)

# Plot feature importance
shap.summary_plot(shap_values, X_test, feature_names=df.columns[:-1])

# Get mean absolute SHAP values for feature importance
shap_importance = np.abs(shap_values.values).mean(axis=0)
shap_importance_df = pd.DataFrame({
    'feature': df.columns[:-1],
    'importance': shap_importance
}).sort_values(by='importance', ascending=False)

print(shap_importance_df)

# Select the top N features based on SHAP importance
top_n_features = shap_importance_df['feature'].head(5).values
print(f"Top {len(top_n_features)} features: {top_n_features}")

# Transform the data to include only the selected features
X_train_shap = X_train[:, shap_importance_df.index[:5]]
X_test_shap = X_test[:, shap_importance_df.index[:5]]

# Train the Elastic Net model with the selected features
elnet_model = ElasticNet(random_state=42)
elnet_model.fit(X_train_shap, y_train)

# Make predictions
y_train_pred_elnet = elnet_model.predict(X_train_shap)
y_test_pred_elnet = elnet_model.predict(X_test_shap)

def calculate_metrics(y_true, y_pred):
    r2 = r2_score(y_true, y_pred)
    rmse = np.sqrt(mean_squared_error(y_true, y_pred))
    mae = mean_absolute_error(y_true, y_pred)
    mse = mean_squared_error(y_true, y_pred)
    rpd = np.std(y_true) / rmse
    return r2, rmse, mae, mse, rpd

def display_metrics(model_name, y_train, y_train_pred, y_test, y_test_pred):
    print(f"---{model_name}---")
    print("Train Metrics:")
    train_metrics = calculate_metrics(y_train, y_train_pred)
    print(f"R2: {train_metrics[0]}, RMSE: {train_metrics[1]}, MAE: {train_metrics[2]}, MSE: {train_metrics[3]}, RPD: {train_metrics[4]}")
    
    print("Test Metrics:")
    test_metrics = calculate_metrics(y_test, y_test_pred)
    print(f"R2: {test_metrics[0]}, RMSE: {test_metrics[1]}, MAE: {test_metrics[2]}, MSE: {test_metrics[3]}, RPD: {test_metrics[4]}")
    print("\n")

# Display metrics
display_metrics("Elastic Net with SHAP-selected features", y_train, y_train_pred_elnet, y_test, y_test_pred_elnet)

```


```python
from catboost import CatBoostRegressor

# Generate a random regression problem
from sklearn.datasets import make_regression
X, y = make_regression(n_samples=100, n_features=10, noise=0.1, random_state=175)

# Define a base model for RFE
base_model = RandomForestRegressor(n_estimators=10, random_state=42)

# Initialize RFE with the base model and select the top 5 features, using a moderate step size
rfe = RFE(estimator=base_model, n_features_to_select=5, step=25)
rfe.fit(X_train, y_train)

# Get the selected features
selected_features = X_train.columns[rfe.support_]
print(f"Selected features: {selected_features}")

# Transform the data to include only the selected features
X_train_rfe = pd.DataFrame(rfe.transform(X_train), columns=selected_features)
X_test_rfe = pd.DataFrame(rfe.transform(X_test), columns=selected_features)

# Standardize the features
scaler = StandardScaler()
X_train_rfe = scaler.fit_transform(X_train_rfe)
X_test_rfe = scaler.transform(X_test_rfe)

# Train the CatBoost model with the selected features
catboost_model = CatBoostRegressor(verbose=0, random_state=42)
catboost_model.fit(X_train_rfe, y_train)

# Make predictions
y_train_pred_catboost = catboost_model.predict(X_train_rfe)
y_test_pred_catboost = catboost_model.predict(X_test_rfe)

def calculate_metrics(y_true, y_pred):
    r2 = r2_score(y_true, y_pred)
    rmse = np.sqrt(mean_squared_error(y_true, y_pred))
    mae = mean_absolute_error(y_true, y_pred)
    mse = mean_squared_error(y_true, y_pred)
    rpd = np.std(y_true) / rmse
    return r2, rmse, mae, mse, rpd

def display_metrics(model_name, y_train, y_train_pred, y_test, y_test_pred):
    print(f"---{model_name}---")
    print("Train Metrics:")
    train_metrics = calculate_metrics(y_train, y_train_pred)
    print(f"R2: {train_metrics[0]}, RMSE: {train_metrics[1]}, MAE: {train_metrics[2]}, MSE: {train_metrics[3]}, RPD: {train_metrics[4]}")
    
    print("Test Metrics:")
    test_metrics = calculate_metrics(y_test, y_test_pred)
    print(f"R2: {test_metrics[0]}, RMSE: {test_metrics[1]}, MAE: {test_metrics[2]}, MSE: {test_metrics[3]}, RPD: {test_metrics[4]}")
    print("\n")

# Display metrics
display_metrics("CatBoost with RFE-selected features", y_train, y_train_pred_catboost, y_test, y_test_pred_catboost)

```
