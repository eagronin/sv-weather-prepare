# Data Preparation

This section pre-processes the data that is subsequently used in the analysis of wheather patterns in Silicon Valley. Specifically, the analysis presented in subsequent sections will show that the range of temperatures has widen in 2015 as compared to the previous 10-year period (from 2005 to 2014).

Data acquisition is described in the [previous section](https://eagronin.github.io/sv-weather-acquire).

The analysis and visualization of results are described in the [next section](https://eagronin.github.io/sv-weather-analyze/).

The following code imports the data, then outputs the first five rows and summary statistics:

```python
df = read_weather_data()

print(df.head())

print('...\n\nSummary Statistics:\n\n', df.describe())
print('\nMissing values across all attributes and samples: ', df.isnull().sum().sum())
```

The output is shown below:

```
            ID     Date Element  Data_Value
0  USC00043244   7/1/12    TMAX         222
1  USC00047807  6/24/05    TMAX         211
2  USC00046599  6/25/06    TMIN         139
3  USR0000CALT  8/14/09    TMAX         256
4  USC00047339  1/23/08    TMIN          56
...

Summary Statistics:

           Data_Value
count  143226.000000
mean      147.061546
std        74.438715
min       -67.000000
25%        94.000000
50%       139.000000
75%       194.000000
max       456.000000

Missing values across all attributes and samples:  0
```

The original data has a column 'Element' with TMIN and TMAX as values.  First, we reshape the data to convert the Element column to two TMIN and TMAX columns by creating two datasets for TMIN and TMAX, respectively, and then joining the datasets.  Next, we remove leap days (i.e. February 29th) from the dataset for the purpose of subsequent analysis and visualization.

```python
def transform_weather_data():
    
    # Read data
    df = read_weather_data()
    
    # Reshape data
    df_min = (df[df.Element == 'TMIN']
              .drop(['ID', 'Element'], axis = 1)
              .groupby('Date').min()
              .sort_index()
              .reset_index(drop = False)
              .rename(columns = {'Data_Value': 'TMIN'}))
    df_min.Date = list(map(pd.to_datetime, df_min.Date))


    df_max = (df[df.Element == 'TMAX']
              .drop(['ID', 'Element'], axis = 1)
              .groupby('Date').max()
              .sort_index()
              .reset_index(drop = False)
              .rename(columns = {'Data_Value': 'TMAX'}))
    df_max.Date = list(map(pd.to_datetime, df_max.Date))


    df_joined = df_min.merge(df_max, how = 'inner', on = 'Date')

    # Remove leap days
    df_joined = df_joined[~((df_joined.Date.dt.month == 2) & (df_joined.Date.dt.day == 29))]
    
    return df_joined

print(df_joined.head())
```

The output below shows the first five rows of the transformed data frame:

```
        Date  TMIN  TMAX
0 2005-01-01    22   144
1 2006-01-01    56   172
2 2007-01-01    11   189
3 2008-01-01   -11   222
4 2009-01-01    44   178
```

Next step: [Analysis](https://eagronin.github.io/sv-weather-analyze/)
