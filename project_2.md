# create growth account table 1990-2019
## first try with jpn (def function code vreated below)
import pandas as pd
import numpy as np

countries = ['Japan']
## create a def function code
def growth_accounting(countries):
    pwt90 = pd.read_stata('https://www.rug.nl/ggdc/docs/pwt90.dta')
    data = pwt90[
        pwt90['country'].isin(countries) & 
        pwt90['year'].between(1990, 2019)
    ]
    
    #sorting data
    relevant_cols = ['countrycode', 'country', 'year', 'rgdpna', 'rkna', 'pop', 'emp', 'avh', 'labsh', 'rtfpna']
    data = data[relevant_cols].dropna()
    
    #deriving alpha(capital share) = 1 - (labor share)
    data['alpha'] = 1 - data['labsh'] 

    #労働者1人あたりの実質GDP
    data['y_n'] = data['rgdpna'] / data['emp']

    #total labor hours
    data['hours'] = data['emp'] * data['avh']
    data['tfp_term'] = data['rtfpna'] ** (1 / (1 - data['alpha']))  # 全要素生産性の調整項
    data['cap_term'] = (data['rkna'] / data['rgdpna']) ** (data['alpha'] / (1 - data['alpha']))  # 資本の寄与調整
    data['lab_term'] = data['hours'] / data['pop']  # 労働投入の割合

    data = data.sort_values('year').groupby('countrycode').apply(
        lambda x: x.assign(alpha=1 - x['labsh'],
        y_n_shifted=100 * x['y_n'] / x['y_n'].iloc[0],  # 基準年を100とした指標
        tfp_term_shifted=100 * x['tfp_term'] / x['tfp_term'].iloc[0],
        cap_term_shifted=100 * x['cap_term'] / x['cap_term'].iloc[0],
        lab_term_shifted=100 * x['lab_term'] / x['lab_term'].iloc[0]
    )).reset_index(drop=True).dropna()
        
        
    def calculate_growth_rates(country_data):
        # 期間の初年度と最終年度の値を取得
        start_year_actual = country_data['year'].min()
        end_year_actual = country_data['year'].max()
        start_data = country_data[country_data['year'] == start_year_actual].iloc[0]
        end_data = country_data[country_data['year'] == end_year_actual].iloc[0]

        years = end_data['year'] - start_data['year']

        g_y = ((end_data['y_n'] / start_data['y_n']) ** (1/years) - 1) * 100  # 労働生産性の年平均成長率
        g_k = ((end_data['cap_term'] / start_data['cap_term']) ** (1/years) - 1) * 100  # 資本深化の成長率
        g_a = ((end_data['tfp_term'] / start_data['tfp_term']) ** (1/years) - 1) * 100  # TFP成長率

        alpha_avg = (start_data['alpha'] + end_data['alpha']) / 2.0
        capital_deepening_contrib = alpha_avg * g_k
        tfp_growth_calculated = g_a

        tfp_share = (tfp_growth_calculated / g_y)
        cap_share = (capital_deepening_contrib / g_y)

        return {
            'Country': start_data['country'],
            'Growth Rate': round(g_y, 2),
            'TFP Growth': round(tfp_growth_calculated, 2),
            'Capital Deepening': round(capital_deepening_contrib, 2),
            'TFP Share': round(tfp_share, 2),
            'Capital Share': round(cap_share, 2)
        }

    results_list = data.groupby('country').apply(calculate_growth_rates).dropna().tolist()
    results_df = pd.DataFrame(results_list)
        
    avg_row_data = {
        'Country': 'Average',
        'Growth Rate': round(results_df['Growth Rate'].mean(), 2),
        'TFP Growth': round(results_df['TFP Growth'].mean(), 2),
        'Capital Deepening': round(results_df['Capital Deepening'].mean(), 2),
        'TFP Share': round(results_df['TFP Share'].mean(), 2),
        'Capital Share': round(results_df['Capital Share'].mean(), 2)
    }
    results_df = pd.concat([results_df, pd.DataFrame([avg_row_data])], ignore_index=True)

    print("\nGrowth Accounting in OECD Countries: 1990-2019 period")
    print("="*85)
    print(results_df.to_string(index=False))

growth_accounting(countries)

## apply other countries
countries = ['Australia', 'Austria', 'Belgium', 'Canada', 'Denmark', 'Finland', 'France', 'Germany', 'Greece','Iceland', 'Ireland', 'Italy', 'Japan', 'Netherlands', 'New Zealand', 'Norway', 'Portugal', 'Spain', 'Sweden', 'Switzerland', 'United Kingdom', 'United States'
]
growth_accounting(countries)