import abc
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import datetime as dt
import warnings
import os

warnings.filterwarnings('ignore')

for dirname, _, filenames in os.walk('/kaggle/input'):
    for filename in filenames:
        print(os.path.join(dirname, filename))

coffee_data = pd.read_csv('/kaggle/input/coffee-sales/index.csv')
print(coffee_data.head())
print(coffee_data.isnull().sum())
print(coffee_data.duplicated().sum())
print(coffee_data.describe().T)
print(coffee_data.loc[:, ['cash_type', 'card', 'coffee_name']].describe().T)
print(coffee_data[coffee_data['card'].isnull()]['cash_type'].value_counts())
coffee_data['cash_type'].hist()
print(coffee_data['cash_type'].value_counts(normalize=True))
print(pd.DataFrame(coffee_data)['coffee_name'].value_counts(normalize=True).sort_values(ascending=False))

coffee_data['date'] = pd.to_datetime(coffee_data['date'])
coffee_data['datetime'] = pd.to_datetime(coffee_data['datetime'])

coffee_data['month'] = coffee_data['date'].dt.strftime('%Y-%m')
coffee_data['day'] = coffee_data['date'].dt.strftime('%w')
coffee_data['hour'] = coffee_data['datetime'].dt.strftime('%H')

print(coffee_data.info())
print(coffee_data.head())

# Example revenue_data assumed to be defined earlier
# revenue_data = ... (Define this variable according to your dataset)

plt.figure(figsize=(10, 4))
ax = sns.barplot(data=revenue_data, x='money', y='coffee_name', color='steelblue')
ax.bar_label(ax.containers[0], fontsize=6)
plt.xlabel('Revenue')

monthly_sales = coffee_data.groupby(['coffee_name', 'month']).count()['date'].reset_index().rename(columns={'date': 'count'})
monthly_sales_pivot = monthly_sales.pivot(index='month', columns='coffee_name', values='count').reset_index()
print(monthly_sales_pivot.describe().T.loc[:, ['min', 'max']])

plt.figure(figsize=(12, 6))
sns.lineplot(data=monthly_sales_pivot)
plt.legend(loc='upper left')
plt.xticks(ticks=range(len(monthly_sales_pivot['month'])), labels=monthly_sales_pivot['month'], size='small')

weekday_sales = coffee_data.groupby(['day']).count()['date'].reset_index().rename(columns={'date': 'count'})
plt.figure(figsize=(12, 6))
sns.barplot(data=weekday_sales, x='day', y='count', color='steelblue')
plt.xticks(ticks=range(len(weekday_sales['day'])), labels=['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'], size='small')

daily_sales = coffee_data.groupby(['coffee_name', 'date']).count()['datetime'].reset_index().rename(columns={'datetime': 'count'})
daily_sales_pivot = daily_sales.pivot(index='date', columns='coffee_name', values='count').reset_index().fillna(0)
print(daily_sales_pivot.describe().T.loc[:, ['min', 'max']])

hourly_sales = coffee_data.groupby(['hour']).count()['date'].reset_index().rename(columns={'date': 'count'})
sns.barplot(data=hourly_sales, x='hour', y='count', color='steelblue')

hourly_sales_by_coffee = coffee_data.groupby(['hour', 'coffee_name']).count()['date'].reset_index().rename(columns={'date': 'count'})
hourly_sales_by_coffee_pivot = hourly_sales_by_coffee.pivot(index='hour', columns='coffee_name', values='count').fillna(0).reset_index()

fig, axs = plt.subplots(2, 4, figsize=(20, 10))
axs = axs.flatten()

for i, column in enumerate(hourly_sales_by_coffee_pivot.columns[1:]):
    axs[i].bar(hourly_sales_by_coffee_pivot['hour'], hourly_sales_by_coffee_pivot[column])
    axs[i].set_title(column)
    axs[i].set_xlabel('Hour')

plt.tight_layout()

