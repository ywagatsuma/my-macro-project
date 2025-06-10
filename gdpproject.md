# comparison of Norway and Japan GDP
## Collected real GDP for Norway 1978-2022
### python code for collecting data and draw graph for lamda = 1600

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

### different lambda values
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

### cyclical components norway
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

## real GDP of Japan
### following is the code for 1994-2022, was not able to find 1978- data since oecd stat page was not working
!pip install pandas_datareader
import pandas as pd
import pandas_datareader.data as web
import matplotlib.pyplot as plt
import statsmodels.api as sm
import pandas_datareader as pdr
import numpy as np

start_date = '1994-01-01'
end_date = '2022-01-01'

series_id = 'JPNRGDPEXP'
gdpjp = web.DataReader(series_id, 'fred', start_date, end_date)
log_gdp_jp = np.log(gdpjp)

cycle_jp, trend_jp = sm.tsa.filters.hpfilter(log_gdp_jp, lamb=1600)

plt.plot(log_gdp_jp, label="Original GDP (in log)")

plt.plot(trend_jp, label="Trend")
plt.title("Japan GDP and trend")

plt.legend()
plt.show()

![Quarterly GDP growth chart for Japan, 1994–2022](\Users\yuiwa\keio-macro\my-macro-project-1/jp_trend_1600.png)

### lambda comparison jp
lambdas_jp = [10, 100, 1600]
trends_jp = {}
cycles_jp = {}

for lam in lambdas_jp:
    cycle_jp, trend_jp = sm.tsa.filters.hpfilter(log_gdp_jp, lamb=lam)
    trends_jp[lam] = trend_jp
    cycles_jp[lam] = cycle_jp

plt.figure(figsize=(12, 6))
plt.plot(log_gdp_jp, label="Log Real GDP", color='gray', linewidth=1)
for lam in lambdas_jp:
    plt.plot(trends_jp[lam], label=f"Trend (λ = {lam})")
    
plt.title("logGDP and lambda comparison JP")
plt.xlabel("year")
plt.ylabel("logGDP")
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()


### cyclical components jp
plt.figure(figsize=(12, 6))
for lam in lambdas_jp:
    plt.plot(cycles_jp[lam], label=f"Cycle (λ = {lam})")

plt.title("Comparison of Cyclical Components JP")
plt.xlabel("year")
plt.ylabel("Cyclical Component (Log GDP)")
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()

## compare standard deviation
std = cycle.std()
std_jp = cycle_jp.std()

print("ノルウェーの循環変動成分の標準偏差:", std)
print("日本の循環変動成分の標準偏差:", std_jp)

## correlation

#2つのSeriesを列方向に結合（共通期間で揃える）
combined = pd.concat([cycle_jp, cycle], axis=1, join='inner')
combined.columns = ['Japan', 'Norway']

common_dates = cycle_jp.index.intersection(cycle.index)
print("共通する期間:", common_dates.min(), "〜", common_dates.max())
print("共通するデータ数（四半期数など）:", len(common_dates))

#相関係数を計算
correlation = combined.corr().iloc[0, 1]

print("日本とノルウェーの循環変動成分の相関係数:", correlation)