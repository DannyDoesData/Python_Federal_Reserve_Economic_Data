# 🌐 Federal Reserve Economic Data Analysis  🌐

### Project goal 🎯

Retrieve and plot Federal Reserve Economic Data (FRED)(US)
This will be done using Python to analyze and FRED API to retrieve data


### Tools Used 🧰
1. **Python (Libraries used: Pandas, NumPy, MatplotLib and PlotlyExpress)**
2. **Visual Studio Code**
3. **Git & GitHub**

## The process

### Getting to know Fred

Importing libraries and setting up configuration

```python
import pandas as pd
import numpy as pn
import matplotlib.pyplot as plt
plt.style.use('fivethirtyeight')
import plotly.express as px
from fredapi import Fred
pd.options.display.max_rows = 200
pd.options.display.max_columns = 500
color_pal = plt.rcParams['axes.prop_cycle'].by_key()['color']
```

Setting up FRED API key and initializing fred instance 

```python 
fred_key = 'Hahaha I am not giving you my key, get your own key at https://fred.stlouisfed.org/, it is free'
fred = Fred(api_key=fred_key)
```

Testing out Fred connection by looking for S&P500 data

```python
sp_search = fred.search('S&P', order_by='popularity')
sp_search.head() #Finding the right data ID for what I want
```

Plotting the S&P data to ensure that we are good with Fred :)

```python
sp500 = fred.get_series(series_id='SP500')
sp500.plot(figsize=(10, 5), title= 'S&P 500', lw=2)
```

![S&P500](/S&P500.png)
*We seem to be on good terms with Fred, here is the S&P500 movements in the past 9 years;


### Unemployment rate 

Extracting the montly seasonally adjusted unemployment rate in % for different US states 

```python
unmp_df = fred.search('unemployment rate state', filter=('frequency','Monthly'))
unemp_df = unmp_df.query('seasonal_adjustment == "Seasonally Adjusted" and units == "Percent"')
unemp_df = unemp_df.loc[unemp_df['title'].str.contains('Unemployment Rate')]
```

Joining the data for various states in a single data frame

```python
all_results2 = []

for myid in unemp_df.index:
    results = fred.get_series(myid)
    results = results.to_frame(name=myid)
    all_results2.append(results)
    
unmp_results = pd.concat(all_results2, axis=1)
```

Filtering out non-state data 

```python
cols_to_drop = []
for i in unmp_results:
    if len(i) > 4:
        cols_to_drop.append(i)
unmp_results = unmp_results.drop(columns = cols_to_drop, axis=1)
```


Dropping N/As (entries from before the unemployment rate started to be published per state) and some formatting 

```python
unmp_states = unmp_results.copy() 
unmp_states = unmp_states.dropna()
id_to_state = unemp_df['title'].str.replace('Unemployment Rate in ','').to_dict()
unmp_states.columns = [id_to_state[c] for c in unmp_states.columns]
```

Plotting unemployment rate by state

```python
px.line(unmp_states)
```

![Unemployment Line Chart](/Unemployment_by_state.png)
*Line graph of unemployemnt rate by US state from 1976 to 2024 (the graph is interactive and states are scrollable if you run the code);


Plotting unemployment for May 2020 as a bar graph

```python
ax = unmp_states.loc[unmp_states.index == '2020-05-01'].T \
    .sort_values('2020-05-01') \
    .plot(kind='barh', figsize=(8, 12), width=0.7, edgecolor='black',
          title='Unemployment Rate by State, May 2020')
ax.legend().remove()
ax.set_xlabel('% Unemployed')
plt.show()
```

