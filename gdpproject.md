# Collected real GDP for Norway 1978-2022
## python code for collecting data and draw graph for lamda = 1600

!pip install pandas_datareader

import pandas as pd
import pandas_datareader.data as web
import matplotlib.pyplot as plt
import statsmodels.api as sm
import pandas_datareader as pdr
import numpy as np

start_date = '1978-01-01'
end_date = '2022-01-01'

series_id = 'CLVMNACNSAB1GQNO'
gdp = web.DataReader(series_id, 'fred', start_date, end_date)
log_gdp = np.log(gdp)

cycle, trend = sm.tsa.filters.hpfilter(log_gdp, lamb=1600)

plt.plot(log_gdp, label="Original GDP (in log)")

plt.plot(trend, label="Trend")

plt.legend()
plt.show()

![image](https://github.com/user-attachments/assets/97a5fcd0-0cbe-443b-8253-3c60219d74d0)

## graph 1 for different lambda values
lambdas = [10, 100, 1600]
trends = {}
cycles = {}

for lam in lambdas:
    cycle, trend = sm.tsa.filters.hpfilter(log_gdp, lamb=lam)
    trends[lam] = trend
    cycles[lam] = cycle

plt.figure(figsize=(12, 6))
plt.plot(log_gdp, label="Log Real GDP", color='gray', linewidth=1)
for lam in lambdas:
    plt.plot(trends[lam], label=f"Trend (λ = {lam})")
    
plt.title("logGDP and lambda comparison")
plt.xlabel("year")
plt.ylabel("logGDP")
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()

![image](https://github.com/user-attachments/assets/f426a07f-b972-4a8b-9a4f-dec3aff92213)

## graph 2
plt.figure(figsize=(12, 6))
for lam in lambdas:
    plt.plot(cycles[lam], label=f"Cycle (λ = {lam})")

plt.title("Comparison of Cyclical Components")
plt.xlabel("year")
plt.ylabel("Cyclical Component (Log GDP)")
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()

![image](https://github.com/user-attachments/assets/62794854-e364-40ac-a63c-fd8c796ebd9a)
