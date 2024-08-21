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

![Unemployment Bar Chart](/Unemployment_bar.png)
*Unemployment rate by US state bar chart for May 2020;


### Workforce Participation Rate 

Extracting the montly seasonally adjusted participation rate in % for different US states 

```python
part_df = fred.search('participation rate state', filter=('frequency','Monthly'))
part_df = part_df.query('seasonal_adjustment == "Seasonally Adjusted" and units == "Percent"')
```

Joining the data

```python
part_id_to_state = part_df['title'].str.replace('Labor Force Participation Rate for ','').to_dict()

all_results = []

for myid in part_df.index:
    results = fred.get_series(myid)
    results = results.to_frame(name=myid)
    all_results.append(results)
part_states = pd.concat(all_results, axis=1)
part_states.columns = [part_id_to_state[c] for c in part_states.columns]
```

Fixing name of Columbia 

```python
unmp_states = unmp_states.rename(columns={'the District of Columbia':'District Of Columbia'})
```

### Unemployment rate against Participation rate in US states

Plotting comparison of unemployment and workforce participation rate by US state

```python
fig, axs = plt.subplots(10, 5, figsize=(30, 30), sharex=True)
axs = axs.flatten()

i = 0
for state in unmp_states.columns:
    if state in ["District Of Columbia","Puerto Rico"]:
        continue
    ax2 = axs[i].twinx()
    unmp_states.query('index >= 2020 and index < 2024')[state] \
        .plot(ax=axs[i], label='Unemployment')
    part_states.query('index >= 2020 and index < 2024')[state] \
        .plot(ax=ax2, label='Participation', color=color_pal[1])
    ax2.grid(False)
    axs[i].set_title(state)
    i += 1
plt.tight_layout()
plt.show()
```

![Unemployment rate VS Participation rate by each US state from 2020 to 2024](/Participation%20vs%20Unemployment%20by%20state.png)
*;

Looking into single state of California since in gave strange results in previous chart 

```python
state = 'California' # Just change it to your state of interest to look into its unemploment vs participation
fig, ax = plt.subplots(figsize=(10, 5), sharex=True)
ax2 = ax.twinx()
uemp_states2 = unmp_states.asfreq('MS')
l1 = uemp_states2.query('index >= 2020 and index < 2024')[state] \
    .plot(ax=ax, label='Unemployment')
l2 = part_states.dropna().query('index >= 2020 and index < 2024')[state] \
    .plot(ax=ax2, label='Participation', color=color_pal[1])
ax2.grid(False)
ax.set_title(state)
fig.legend(labels=['Unemployment','Participation'])
plt.show()
```

![Unemployment rate VS Participation rate in state of California from 2020 to 2024](/California.png)
*Graph seems to be fine here when we looking only at California, maybe a small blue point at the bottom left indicates that data needs cleaning, but it is for the next time. Data cleaning is going to be another project;


# What I Learned

Throughout this project, I've practiced my Python Data Analyst toolkit with various challenges:

- **🧩 Data manipulation with Pandas and Numpy:** Mastered the art of data manipulation with pandas library. Merging, filtering and manipulating data like a true ?Pandaren? haha
- **📊 Data Visualization:** Created simple and complex data visualisations using MatplotLib and PlotlyExpress libraries
- **💡 Analytical Approach:** Leveled up my real-world puzzle-solving skills, turning my interest in data into actionable scripts and insightful graphs.