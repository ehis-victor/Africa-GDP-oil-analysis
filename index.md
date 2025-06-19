---
title: African GDP Analysis Project
---

# 🧮 African GDP Analysis (2015–2023)

> **Author**: [Ekikhalo Ehinome Victor](https://github.com/ehis-victor)  
> **Published**: June 2025  
> **Tools Used**: Python, Pandas, Matplotlib, Seaborn, Jupyter Notebook

---

## 📌 Project Overview

This project analyzes the Gross Domestic Product (GDP) trends of African countries over the past 9 years (2015–2023). The analysis focuses on:

- Scraping and compiling GDP data from online sources
- Identifying the top 5 economies in Africa by GDP
- Visualizing their GDP growth over time
- Highlighting oil production data for the top oil-producing country among them
- Drawing conclusions based on economic trends

This project is part of a data analysis portfolio and demonstrates skills in data wrangling, exploratory data analysis (EDA), and storytelling through visualizations.

---

🔙 [Back to Portfolio](https://github.com/ehis-victor)

## Let us Begin By Webscraping the World Bank API

```python
%pip install squarify
```

```python
import requests, pandas as pd, time
import seaborn as sns
import matplotlib.pyplot as plt
import squarify as sq
```

```python


# 1️⃣ Get all African countries via the /country endpoint
COUNTRIES_URL = "https://api.worldbank.org/v2/country?region=AFR&format=json&per_page=400"
country_resp  = requests.get(COUNTRIES_URL).json()
# The second element (index 1) is the list of country dicts
all_african = country_resp[1]

# Extract ISO3 codes (and drop the aggregate “SSA” entry if present)
iso3_codes = [c["id"] for c in all_african if c["id"] != "SSA"]
print(f"Found {len(iso3_codes)} ISO3 codes: {iso3_codes[:5]} …")

# 2️⃣ Pull GDP data for each country, 2015–2024
YEARS     = "2015:2024"
INDICATOR = "NY.GDP.MKTP.CD"  # GDP (current US$)
records   = []

for iso3 in iso3_codes:
    url = (
        f"https://api.worldbank.org/v2/country/{iso3}/indicator/"
        f"{INDICATOR}?format=json&date={YEARS}&per_page=60"
    )
    resp = requests.get(url).json()
    # resp[1] holds the yearly entries (might be empty)
    if len(resp) < 2 or not resp[1]:
        continue
    for entry in resp[1]:
        records.append({
            "country": entry["country"]["value"],
            "iso3":    iso3,
            "year":    int(entry["date"]),
            "gdp_usd": entry["value"],
        })
    time.sleep(0.2)

# 3️⃣ Build DataFrame
df = pd.DataFrame(records)
df.sort_values(["country","year"], inplace=True)
print(df.head())
print(df.tail())
print(f"Total rows: {len(df):,}")

# 4️⃣ Save to CSV
#df.to_csv("africa_gdp_2015_2024.csv", index=False)
print("Saved → africa_gdp_2015_2024.csv")

```

    Found 54 ISO3 codes: ['AGO', 'BDI', 'BEN', 'BFA', 'BWA'] …
         country iso3  year       gdp_usd
    139  Algeria  DZA  2015  1.874939e+11
    138  Algeria  DZA  2016  1.807638e+11
    137  Algeria  DZA  2017  1.898809e+11
    136  Algeria  DZA  2018  1.945545e+11
    135  Algeria  DZA  2019  1.934597e+11
          country iso3  year       gdp_usd
    534  Zimbabwe  ZWE  2020  2.686794e+10
    533  Zimbabwe  ZWE  2021  2.724052e+10
    532  Zimbabwe  ZWE  2022  3.278975e+10
    531  Zimbabwe  ZWE  2023  3.523137e+10
    530  Zimbabwe  ZWE  2024           NaN
    Total rows: 540
    Saved → africa_gdp_2015_2024.csv

```python
#df.to_csv("africa_gdp_2015_2024.csv", index=False)
```

```python
#Display the dataframe for inspection
df.head(10)
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }

</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>country</th>
      <th>iso3</th>
      <th>year</th>
      <th>gdp_usd</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>139</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2015</td>
      <td>1.874939e+11</td>
    </tr>
    <tr>
      <th>138</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2016</td>
      <td>1.807638e+11</td>
    </tr>
    <tr>
      <th>137</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2017</td>
      <td>1.898809e+11</td>
    </tr>
    <tr>
      <th>136</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2018</td>
      <td>1.945545e+11</td>
    </tr>
    <tr>
      <th>135</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2019</td>
      <td>1.934597e+11</td>
    </tr>
    <tr>
      <th>134</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2020</td>
      <td>1.648734e+11</td>
    </tr>
    <tr>
      <th>133</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2021</td>
      <td>1.862312e+11</td>
    </tr>
    <tr>
      <th>132</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2022</td>
      <td>2.256385e+11</td>
    </tr>
    <tr>
      <th>131</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2023</td>
      <td>2.476262e+11</td>
    </tr>
    <tr>
      <th>130</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2024</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>

```python
# Cleaning the data
# Drop empty rows
df_clean = df.dropna(subset=['gdp_usd'])

# 2 convert gdp_usd to numeric column
pd.to_numeric(df_clean['gdp_usd'])

#romove country with less than 5 years data
min_years = 5
country_counts = df_clean["country"].value_counts()
valid_countries = country_counts[country_counts >= min_years].index
df_clean = df_clean[df_clean["country"].isin(valid_countries)]

```

```python
# check the datatype
df_clean.dtypes
```

    country     object
    iso3        object
    year         int64
    gdp_usd    float64
    dtype: object

```python
# Check the dataframe
df_clean.head(10)
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }

</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>country</th>
      <th>iso3</th>
      <th>year</th>
      <th>gdp_usd</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>139</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2015</td>
      <td>1.874939e+11</td>
    </tr>
    <tr>
      <th>138</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2016</td>
      <td>1.807638e+11</td>
    </tr>
    <tr>
      <th>137</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2017</td>
      <td>1.898809e+11</td>
    </tr>
    <tr>
      <th>136</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2018</td>
      <td>1.945545e+11</td>
    </tr>
    <tr>
      <th>135</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2019</td>
      <td>1.934597e+11</td>
    </tr>
    <tr>
      <th>134</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2020</td>
      <td>1.648734e+11</td>
    </tr>
    <tr>
      <th>133</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2021</td>
      <td>1.862312e+11</td>
    </tr>
    <tr>
      <th>132</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2022</td>
      <td>2.256385e+11</td>
    </tr>
    <tr>
      <th>131</th>
      <td>Algeria</td>
      <td>DZA</td>
      <td>2023</td>
      <td>2.476262e+11</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Angola</td>
      <td>AGO</td>
      <td>2015</td>
      <td>9.049642e+10</td>
    </tr>
  </tbody>
</table>
</div>

```python
# pivot table for exploration
pivot = df_clean.pivot(index='year', columns='country', values='gdp_usd')
pivot.head(10)
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }

</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>country</th>
      <th>Algeria</th>
      <th>Angola</th>
      <th>Benin</th>
      <th>Botswana</th>
      <th>Burkina Faso</th>
      <th>Burundi</th>
      <th>Cabo Verde</th>
      <th>Cameroon</th>
      <th>Central African Republic</th>
      <th>Chad</th>
      <th>...</th>
      <th>Sierra Leone</th>
      <th>Somalia</th>
      <th>South Africa</th>
      <th>Sudan</th>
      <th>Tanzania</th>
      <th>Togo</th>
      <th>Tunisia</th>
      <th>Uganda</th>
      <th>Zambia</th>
      <th>Zimbabwe</th>
    </tr>
    <tr>
      <th>year</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015</th>
      <td>1.874939e+11</td>
      <td>9.049642e+10</td>
      <td>1.138816e+10</td>
      <td>1.353075e+10</td>
      <td>1.183216e+10</td>
      <td>3.104004e+09</td>
      <td>1.749858e+09</td>
      <td>3.221023e+10</td>
      <td>1.695826e+09</td>
      <td>1.095039e+10</td>
      <td>...</td>
      <td>6.651627e+09</td>
      <td>6.152149e+09</td>
      <td>3.467098e+11</td>
      <td>5.172676e+10</td>
      <td>4.741392e+10</td>
      <td>5.755461e+09</td>
      <td>4.577949e+10</td>
      <td>3.238718e+10</td>
      <td>2.125122e+10</td>
      <td>1.996312e+10</td>
    </tr>
    <tr>
      <th>2016</th>
      <td>1.807638e+11</td>
      <td>5.276162e+10</td>
      <td>1.182107e+10</td>
      <td>1.508264e+10</td>
      <td>1.283336e+10</td>
      <td>2.644488e+09</td>
      <td>1.849790e+09</td>
      <td>3.381434e+10</td>
      <td>1.825018e+09</td>
      <td>1.009778e+10</td>
      <td>...</td>
      <td>6.042616e+09</td>
      <td>6.613743e+09</td>
      <td>3.235855e+11</td>
      <td>4.263038e+10</td>
      <td>4.977441e+10</td>
      <td>6.071171e+09</td>
      <td>4.436007e+10</td>
      <td>2.920399e+10</td>
      <td>2.095841e+10</td>
      <td>2.054868e+10</td>
    </tr>
    <tr>
      <th>2017</th>
      <td>1.898809e+11</td>
      <td>7.369015e+10</td>
      <td>1.270166e+10</td>
      <td>1.610516e+10</td>
      <td>1.410696e+10</td>
      <td>2.723587e+09</td>
      <td>1.996742e+09</td>
      <td>3.609855e+10</td>
      <td>2.072350e+09</td>
      <td>1.000039e+10</td>
      <td>...</td>
      <td>5.818480e+09</td>
      <td>7.621502e+09</td>
      <td>3.814488e+11</td>
      <td>4.128362e+10</td>
      <td>5.327488e+10</td>
      <td>6.387423e+09</td>
      <td>4.216353e+10</td>
      <td>3.074447e+10</td>
      <td>2.587360e+10</td>
      <td>5.107466e+10</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>1.945545e+11</td>
      <td>7.945069e+10</td>
      <td>1.426241e+10</td>
      <td>1.703194e+10</td>
      <td>1.589007e+10</td>
      <td>2.667182e+09</td>
      <td>2.205100e+09</td>
      <td>3.995555e+10</td>
      <td>2.220979e+09</td>
      <td>1.123917e+10</td>
      <td>...</td>
      <td>6.390515e+09</td>
      <td>7.873441e+09</td>
      <td>4.052607e+11</td>
      <td>3.233378e+10</td>
      <td>5.700371e+10</td>
      <td>7.029300e+09</td>
      <td>4.268650e+10</td>
      <td>3.292703e+10</td>
      <td>2.631151e+10</td>
      <td>3.415607e+10</td>
    </tr>
    <tr>
      <th>2019</th>
      <td>1.934597e+11</td>
      <td>7.089796e+10</td>
      <td>1.439169e+10</td>
      <td>1.672591e+10</td>
      <td>1.603281e+10</td>
      <td>2.576519e+09</td>
      <td>2.252177e+09</td>
      <td>3.966776e+10</td>
      <td>2.221301e+09</td>
      <td>1.131495e+10</td>
      <td>...</td>
      <td>6.523578e+09</td>
      <td>8.655024e+09</td>
      <td>3.893300e+11</td>
      <td>3.233808e+10</td>
      <td>6.102673e+10</td>
      <td>6.992700e+09</td>
      <td>4.190564e+10</td>
      <td>3.534816e+10</td>
      <td>2.330867e+10</td>
      <td>2.571741e+10</td>
    </tr>
    <tr>
      <th>2020</th>
      <td>1.648734e+11</td>
      <td>4.850156e+10</td>
      <td>1.565155e+10</td>
      <td>1.496029e+10</td>
      <td>1.772501e+10</td>
      <td>2.649680e+09</td>
      <td>1.821566e+09</td>
      <td>4.077324e+10</td>
      <td>2.326721e+09</td>
      <td>1.071540e+10</td>
      <td>...</td>
      <td>6.688595e+09</td>
      <td>8.628394e+09</td>
      <td>3.379747e+11</td>
      <td>2.703459e+10</td>
      <td>6.606874e+10</td>
      <td>7.400284e+09</td>
      <td>4.249178e+10</td>
      <td>3.760037e+10</td>
      <td>1.813776e+10</td>
      <td>2.686794e+10</td>
    </tr>
    <tr>
      <th>2021</th>
      <td>1.862312e+11</td>
      <td>6.650513e+10</td>
      <td>1.769008e+10</td>
      <td>1.875095e+10</td>
      <td>1.969752e+10</td>
      <td>2.775799e+09</td>
      <td>2.051843e+09</td>
      <td>4.501194e+10</td>
      <td>2.516498e+09</td>
      <td>1.177998e+10</td>
      <td>...</td>
      <td>7.165214e+09</td>
      <td>9.483997e+09</td>
      <td>4.208869e+11</td>
      <td>3.422951e+10</td>
      <td>7.065563e+10</td>
      <td>8.342244e+09</td>
      <td>4.681229e+10</td>
      <td>4.052979e+10</td>
      <td>2.209642e+10</td>
      <td>2.724052e+10</td>
    </tr>
    <tr>
      <th>2022</th>
      <td>2.256385e+11</td>
      <td>1.043997e+11</td>
      <td>1.740175e+10</td>
      <td>2.032196e+10</td>
      <td>1.882022e+10</td>
      <td>3.338723e+09</td>
      <td>2.247003e+09</td>
      <td>4.434721e+10</td>
      <td>2.382619e+09</td>
      <td>1.239681e+10</td>
      <td>...</td>
      <td>7.119137e+09</td>
      <td>1.020277e+10</td>
      <td>4.069200e+11</td>
      <td>5.166688e+10</td>
      <td>7.576997e+10</td>
      <td>8.169476e+09</td>
      <td>4.457976e+10</td>
      <td>4.556533e+10</td>
      <td>2.916378e+10</td>
      <td>3.278975e+10</td>
    </tr>
    <tr>
      <th>2023</th>
      <td>2.476262e+11</td>
      <td>8.482465e+10</td>
      <td>1.967605e+10</td>
      <td>1.939608e+10</td>
      <td>2.032462e+10</td>
      <td>2.642162e+09</td>
      <td>2.533819e+09</td>
      <td>4.927941e+10</td>
      <td>2.555492e+09</td>
      <td>1.314933e+10</td>
      <td>...</td>
      <td>6.411870e+09</td>
      <td>1.096852e+10</td>
      <td>3.806993e+11</td>
      <td>1.092655e+11</td>
      <td>7.906240e+10</td>
      <td>9.171262e+09</td>
      <td>4.852960e+10</td>
      <td>4.876896e+10</td>
      <td>2.757796e+10</td>
      <td>3.523137e+10</td>
    </tr>
  </tbody>
</table>
<p>9 rows × 52 columns</p>
</div>

```python
# Save the cleaned data
df_clean.to_csv('gdp_data_2014-2023_cleaned.csv', index=False)
```

```python
# Calculate the average gdp of all countries and group by country to get the top 5 countries
top5_avg_gdp = (
    df_clean.groupby('country')['gdp_usd']
    .mean(numeric_only= True)
    .sort_values(ascending=False)
    .head(5)
)

print("Top 5 African countries by average GDP (2015–2023):\n")
top5_avg_gdp

# Save their names for future filtering
top5_countries = top5_avg_gdp.index.tolist()
print("\nTop 5 countries:", top5_countries)
```

    Top 5 African countries by average GDP (2015–2023):


    Top 5 countries: ['nigeria', 'south africa', 'egypt, arab rep.', 'algeria', 'morocco']

```python
top5_avg_gdp.head()
```

    country
    Nigeria             4.315512e+11
    South Africa        3.769795e+11
    Egypt, Arab Rep.    3.525198e+11
    Algeria             1.967247e+11
    Morocco             1.261704e+11
    Name: gdp_usd, dtype: float64

## Line plot for top 5 countries

```python
#Set style to use for the plot
sns.set(style='white')

# set figure size
plt.figure(figsize=(10,6))

#Filter top 5 from the dataframe
df_top5 = df_clean[df_clean['country'].isin(top5_countries)].copy()

#create another column to display the numbers for the gdp in billions
df_top5['gdp_usd_billions'] = df_top5['gdp_usd'] /1e9

#plot
sns.lineplot(data=df_top5, x='year', y='gdp_usd_billions', hue='country', marker='o')

plt.title('Top 5 countries GDP Trends 2015 - 2023')
plt.xlabel('Years')
plt.ylabel('gdp (USD)')
plt.tight_layout()
plt.legend(title= 'Countries')
plt.grid(False)
plt.show()

```

![png](output_13_0.png)

## BAR CHART (MOST RECENT YEAR GDP)

```python
# filter by only 2023
df_2023 = df_top5[df_top5['year'] == 2023]

# sort values of 2023 from highest to lowest
df_2023_sorted = df_2023.sort_values('gdp_usd_billions', ascending= False)
```

```python
# Bar chart
sns.set(style='white')
width = 0.6

plt.figure(figsize=(10,6))

sns.barplot(data=df_2023_sorted, x='country', y='gdp_usd_billions', width=width,  color='b')

plt.title('Top 5 countries by GDP in 2023')
plt.xlabel('Country')
plt.ylabel('GDP (USD Billion)')
plt.tight_layout()
plt.show()

```

![png](output_16_0.png)

## AREA PLOT (cummulative GDP progress)

```python
# pivot for stacked colum chart
area_df = df_top5.pivot(index='year', columns='country', values='gdp_usd_billions')

plt.figure(figsize=(10,6))

area_df.plot(kind='area', figsize=(12, 6), cmap='coolwarm', alpha= 0.85)

plt.title('Cummulative GDP progress 2015 - 2023')
plt.xlabel('year')
plt.ylabel('gdp (USD BILLION)')
plt.legend(title= 'country')
plt.tight_layout()
plt.grid(True)
plt.show()

```

    <Figure size 1000x600 with 0 Axes>

![png](output_18_1.png)

## HEATMAP (GDP intensity over time)

```python
# normalise for better contrast
heatmap_df = area_df.copy()
heatmap_df = heatmap_df.T

plt.figure(figsize=(10,6))

sns.heatmap(heatmap_df, annot=True, fmt='.0f', cmap='coolwarm', cbar_kws={'label': 'GDP (Billions USD)'})
plt.title('Heatmap of GDP over time')
plt.xlabel('year')
plt.ylabel('gdp (Billions USD)')
plt.tight_layout()
plt.show()

```

![png](output_20_0.png)

## Boxplot: GDP Distribution (Variation Over 10 Years)

```python
plt.figure(figsize=(10, 6))

flierprops= dict(marker= 'D', markerfacecolor= 'r')

sns.boxplot(data=df_top5, hue="country", y="gdp_usd_billions", palette="pastel", notch= True, flierprops= flierprops)
plt.title("GDP Distribution (2015–2023) – Top 5 African Countries", fontsize=14)
plt.ylabel("GDP (Billion USD)")
plt.xlabel("Country")
plt.grid(axis='y')
#plt.legend(False)
plt.tight_layout()
plt.show()
```

![png](output_22_0.png)

## QUESTION 2

**creating a dataset from real data on the internet to find the top oil producing countries**

```python

# Create the data as a list of dictionaries
data = [
    {"Rank": 1, "Country": "Nigeria", "Production (bpd)": "1510000"},
    {"Rank": 2, "Country": "Libya", "Production (bpd)": "1400000"},
    {"Rank": 3, "Country": "Angola", "Production (bpd)": "1300000"},
    {"Rank": 4, "Country": "Algeria", "Production (bpd)": "960000"},
    {"Rank": 5, "Country": "Egypt", "Production (bpd)": "700000"},
    {"Rank": 6, "Country": "Equatorial Guinea", "Production (bpd)": "400000"},
    {"Rank": 7, "Country": "Gabon", "Production (bpd)": "300000"},
    {"Rank": 8, "Country": "Republic of the Congo", "Production (bpd)": "267000"},
    {"Rank": 9, "Country": "South Sudan", "Production (bpd)": "200000"},
    {"Rank": 10, "Country": "Ghana", "Production (bpd)": "150000"},
    {"Rank": 11, "Country": "Côte d’Ivoire", "Production (bpd)": "60000"},
    {"Rank": 12, "Country": "Niger", "Production (bpd)": "110000"},
]

# Convert to DataFrame
df_oil_prd = pd.DataFrame(data)

# Save to CSV
#df_oil_prd.to_csv('oil_producing_countries.csv', index=False)

```

```python
# inspect the data
df_oil_prd.head()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }

</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Rank</th>
      <th>Country</th>
      <th>Production (bpd)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Nigeria</td>
      <td>1510000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Libya</td>
      <td>1400000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Angola</td>
      <td>1300000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Algeria</td>
      <td>960000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Egypt</td>
      <td>700000</td>
    </tr>
  </tbody>
</table>
</div>

```python
#check the datatypes
df_oil_prd.dtypes
```

    Rank                 int64
    Country             object
    Production (bpd)    object
    dtype: object

```python
# convert the production (bpd) to int
df_oil_prd['Production (bpd)'] = df_oil_prd['Production (bpd)'].astype('Int64')

#create another column to display the numbers for the bpd in millions
df_oil_prd['(bpd) million'] = df_oil_prd['Production (bpd)'] /1e6
```

```python
#View dataframe
df_oil_prd.head()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }

</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Rank</th>
      <th>Country</th>
      <th>Production (bpd)</th>
      <th>(bpd) million</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Nigeria</td>
      <td>1510000</td>
      <td>1.51</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Libya</td>
      <td>1400000</td>
      <td>1.4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Angola</td>
      <td>1300000</td>
      <td>1.3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Algeria</td>
      <td>960000</td>
      <td>0.96</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Egypt</td>
      <td>700000</td>
      <td>0.7</td>
    </tr>
  </tbody>
</table>
</div>

```python
# View dataframe general information
df_oil_prd.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 12 entries, 0 to 11
    Data columns (total 4 columns):
     #   Column            Non-Null Count  Dtype
    ---  ------            --------------  -----
     0   Rank              12 non-null     int64
     1   Country           12 non-null     object
     2   Production (bpd)  12 non-null     Int64
     3   (bpd) million     12 non-null     Float64
    dtypes: Float64(1), Int64(1), int64(1), object(1)
    memory usage: 540.0+ bytes

```python
# Sort the dataset in descending order
top_oil_prod = df_oil_prd.sort_values('Production (bpd)', ascending=False)
top_oil_prod
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }

</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Rank</th>
      <th>Country</th>
      <th>Production (bpd)</th>
      <th>(bpd) million</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Nigeria</td>
      <td>1510000</td>
      <td>1.51</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Libya</td>
      <td>1400000</td>
      <td>1.4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Angola</td>
      <td>1300000</td>
      <td>1.3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Algeria</td>
      <td>960000</td>
      <td>0.96</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Egypt</td>
      <td>700000</td>
      <td>0.7</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>Equatorial Guinea</td>
      <td>400000</td>
      <td>0.4</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>Gabon</td>
      <td>300000</td>
      <td>0.3</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>Republic of the Congo</td>
      <td>267000</td>
      <td>0.267</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>South Sudan</td>
      <td>200000</td>
      <td>0.2</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>Ghana</td>
      <td>150000</td>
      <td>0.15</td>
    </tr>
    <tr>
      <th>11</th>
      <td>12</td>
      <td>Niger</td>
      <td>110000</td>
      <td>0.11</td>
    </tr>
    <tr>
      <th>10</th>
      <td>11</td>
      <td>Côte d’Ivoire</td>
      <td>60000</td>
      <td>0.06</td>
    </tr>
  </tbody>
</table>
</div>

## Plot the top oil producers

```python
plt.figure(figsize=(15,8))

width= 0.8

sns.barplot(x='Country', y='Production (bpd)', data=top_oil_prod, color='b', width=width)
plt.title('Top oil producing countries in Africa (BPD)')
plt.xlabel('Country')
plt.ylabel('Production BPD')
plt.tight_layout()
plt.grid(axis='y')
plt.show()
```

![png](output_32_0.png)

```python
# Manually create the data
data = {
    'Year': [2015, 2016, 2017, 2018, 2019, 2020, 2021, 2022, 2023],
    'GDP_USD_Billion': [493, 405, 375, 397, 448, 432, 440, 477, 489],
    'Oil_Price (pb)': [52, 43, 54, 71, 64, 41, 70, 99, 82]  # Brent crude avg est.
}

# Create DataFrame
df_nigeria = pd.DataFrame(data)
df_nigeria.head()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }

</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Year</th>
      <th>GDP_USD_Billion</th>
      <th>Oil_Price (pb)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2015</td>
      <td>493</td>
      <td>52</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2016</td>
      <td>405</td>
      <td>43</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017</td>
      <td>375</td>
      <td>54</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018</td>
      <td>397</td>
      <td>71</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2019</td>
      <td>448</td>
      <td>64</td>
    </tr>
  </tbody>
</table>
</div>

```python
fig, ax1 = plt.subplots(figsize=(10, 6))

# First axis: GDP
color = 'tab:blue'
ax1.set_xlabel('Year')
ax1.set_ylabel('GDP (USD Billion)', color=color)
ax1.plot(df_nigeria['Year'], df_nigeria['GDP_USD_Billion'], color=color, marker='o', label='Nigeria GDP')
ax1.tick_params(axis='y', labelcolor=color)

# Second axis: Oil Price
ax2 = ax1.twinx()
color = 'tab:red'
ax2.set_ylabel('Oil Price (USD/barrel)', color=color)
ax2.plot(df_nigeria['Year'], df_nigeria['Oil_Price (pb)'], color=color, marker='s', linestyle='--', label='Oil_Price (pb)')
ax2.tick_params(axis='y', labelcolor=color)

plt.title('Nigeria GDP vs Global Oil Prices (2015–2023)')
fig.tight_layout()
plt.grid(True)
plt.show()
```

![png](output_34_0.png)

```python
# Strip whitespace and make all country names lowercase for consistency
df_clean['country'] = df_clean['country'].str.strip().str.lower()
df_oil_prd['country'] = df_oil_prd['Country'].str.strip().str.lower()

```

```python
# Merge initial GDP dataframe with Oil Production Datframe
df_merged = pd.merge(df_clean, df_oil_prd, on='country', how='inner')
#df_merged.to_csv('Merged_dataframe.csv', index=True)
```

```python
# Format column to millions
df_merged['Production (bpd) million'] = df_merged['Production (bpd)'] / 1e6
```

```python
#Display dataframe
df_merged
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }

</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>country</th>
      <th>iso3</th>
      <th>year</th>
      <th>gdp_usd</th>
      <th>Rank</th>
      <th>Country</th>
      <th>Production (bpd)</th>
      <th>(bpd) million</th>
      <th>Production (bpd) million</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>algeria</td>
      <td>DZA</td>
      <td>2015</td>
      <td>1.874939e+11</td>
      <td>4</td>
      <td>Algeria</td>
      <td>960000</td>
      <td>0.96</td>
      <td>0.96</td>
    </tr>
    <tr>
      <th>1</th>
      <td>algeria</td>
      <td>DZA</td>
      <td>2016</td>
      <td>1.807638e+11</td>
      <td>4</td>
      <td>Algeria</td>
      <td>960000</td>
      <td>0.96</td>
      <td>0.96</td>
    </tr>
    <tr>
      <th>2</th>
      <td>algeria</td>
      <td>DZA</td>
      <td>2017</td>
      <td>1.898809e+11</td>
      <td>4</td>
      <td>Algeria</td>
      <td>960000</td>
      <td>0.96</td>
      <td>0.96</td>
    </tr>
    <tr>
      <th>3</th>
      <td>algeria</td>
      <td>DZA</td>
      <td>2018</td>
      <td>1.945545e+11</td>
      <td>4</td>
      <td>Algeria</td>
      <td>960000</td>
      <td>0.96</td>
      <td>0.96</td>
    </tr>
    <tr>
      <th>4</th>
      <td>algeria</td>
      <td>DZA</td>
      <td>2019</td>
      <td>1.934597e+11</td>
      <td>4</td>
      <td>Algeria</td>
      <td>960000</td>
      <td>0.96</td>
      <td>0.96</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>67</th>
      <td>nigeria</td>
      <td>NGA</td>
      <td>2019</td>
      <td>4.745175e+11</td>
      <td>1</td>
      <td>Nigeria</td>
      <td>1510000</td>
      <td>1.51</td>
      <td>1.51</td>
    </tr>
    <tr>
      <th>68</th>
      <td>nigeria</td>
      <td>NGA</td>
      <td>2020</td>
      <td>4.321989e+11</td>
      <td>1</td>
      <td>Nigeria</td>
      <td>1510000</td>
      <td>1.51</td>
      <td>1.51</td>
    </tr>
    <tr>
      <th>69</th>
      <td>nigeria</td>
      <td>NGA</td>
      <td>2021</td>
      <td>4.408336e+11</td>
      <td>1</td>
      <td>Nigeria</td>
      <td>1510000</td>
      <td>1.51</td>
      <td>1.51</td>
    </tr>
    <tr>
      <th>70</th>
      <td>nigeria</td>
      <td>NGA</td>
      <td>2022</td>
      <td>4.774034e+11</td>
      <td>1</td>
      <td>Nigeria</td>
      <td>1510000</td>
      <td>1.51</td>
      <td>1.51</td>
    </tr>
    <tr>
      <th>71</th>
      <td>nigeria</td>
      <td>NGA</td>
      <td>2023</td>
      <td>3.638463e+11</td>
      <td>1</td>
      <td>Nigeria</td>
      <td>1510000</td>
      <td>1.51</td>
      <td>1.51</td>
    </tr>
  </tbody>
</table>
<p>72 rows × 9 columns</p>
</div>

```python
#plot GDP vs oil production in African countries
plt.figure(figsize=(10, 6))
sns.scatterplot(data=df_merged, x='Production (bpd)', y='gdp_usd', hue='Country')
plt.xlabel("Oil Production (Million Barrels Per Day)")
plt.ylabel("GDP (USD Billions)")
plt.title("GDP vs Oil Production in African Countries")
plt.grid(True)
plt.show()
```

![png](output_39_0.png)

## Conclusion

**📌 Africa's Economic Giants — 9 years in Review**

Over the last 9 years, Africa has undergone significant economic shifts, with a few standout nations consistently leading the continent in terms of GDP. Our analysis, leveraging World Bank data from 2015 to 2023, reveals a clear pattern: economic leadership in Africa is both diverse in foundation and deeply influenced by resource management, industrial capacity, and geopolitical stability.
