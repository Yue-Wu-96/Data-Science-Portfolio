---
jupyter:
  jupytext:
    formats: ipynb,md
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.4.2
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
---

### Table of Content

---
- [Preparation](#preparation)
 - Load data, Clean Headers, Inspect
 
- [Missing Values](#na)
 - Identify NA 
 - NA distribution, dtypes, fill NAs
 - Correct data format
 
- [Data standadization](#standadization)
- [Data Normalization (centering/scaling)](#normalization)
- [Binning](#binning)
- [Indicator variable](#dummy)

### Libraries

- `pandas`, `numpy`, `matplotlib`, `pyplot`, `missingno`, `seaborn`

---
__Author: Yue Wu__


  # Preparation <a id="preparation"></a>


### Load Data

```python
import pandas as pd
```

```python
url='https://archive.ics.uci.edu/ml/machine-learning-databases/autos/imports-85.data'
df=pd.read_csv(url)
df.head(3)
```

### Clean Headers 

```python
headers = ["symboling","normalized-losses","make","fuel-type","aspiration", "num-of-doors","body-style",
         "drive-wheels","engine-location","wheel-base", "length","width","height","curb-weight","engine-type",
         "num-of-cylinders", "engine-size","fuel-system","bore","stroke","compression-ratio","horsepower",
         "peak-rpm","city-mpg","highway-mpg","price"]
```

```python
df.columns=headers
df.head(3)
```

### Inspect

```python
df.shape
```

```python
any(df.isna())
```

```python
df.info()
```

# Missing Values <a id="na"></a>


### Replace ? with np.nan

```python jupyter={"outputs_hidden": false}
import numpy as np

df.replace("?", np.nan, inplace = True)
df.head(3)
```

### NA Distribution

```python
import missingno as msno
import matplotlib.pyplot as plt

msno.matrix(df)
plt.show()
```

### NA Counts

```python
na = df.isna().sum().sort_values(ascending=False)[0:10]
na
```

### NA Dtypes

```python
df[na.index[0:7]].dtypes
```

```python
df[na.index[0:7]].describe()
```

<!-- #region -->
### Fill NAs


### Replace by mean:

- normalized-losses: 41 missing
- stroke: 4 missing
- bore: 4 missing
- horsepower: 2 missing
- peak-rpm: 2 missing
<!-- #endregion -->

```python
for col in df[['normalized-losses','bore','stroke','horsepower','peak-rpm']].columns:
    x = df[col].astype('float').mean()
    df[col].replace(np.nan, x, inplace=True)
```

```python
df.isna().sum().sort_values(ascending=False)[0:10]
```

### Replace by Frequency

- num-of-doors: 2 missing

```python jupyter={"outputs_hidden": false}
df['num-of-doors'].value_counts().idxmax()
```

```python jupyter={"outputs_hidden": false}
df["num-of-doors"].replace(np.nan, "four", inplace=True)
```

### Drop Row

- price: 4 missing, target variable

```python
df.dropna(subset=["price"], axis=0, inplace=True)
df.reset_index(drop=True, inplace=True)
```

```python jupyter={"outputs_hidden": false}
df.isna().sum().sort_values(ascending=False)[0:10]
```

```python
msno.matrix(df) # double check
```

#### Missing data cleaned.


### Clean Data Format
 
- `astype`

```python jupyter={"outputs_hidden": false}
df.dtypes
```

```python jupyter={"outputs_hidden": false}
df[["bore", "stroke"]] = df[["bore", "stroke"]].astype("float")
df[["normalized-losses"]] = df[["normalized-losses"]].astype("int")
df[["price"]] = df[["price"]].astype("float")
df[["peak-rpm"]] = df[["peak-rpm"]].astype("float")
```

```python jupyter={"outputs_hidden": false}
df[["bore",'stroke',"normalized-losses","price","peak-rpm"]].dtypes
```

# Data Standardization <a id="standardization"></a>


Convert mpg to L/100km: `L/100km = 235 / mpg`


```python jupyter={"outputs_hidden": false}
df['city-L/100km'] = 235/df["city-mpg"]
df.head(2)
```

```python jupyter={"outputs_hidden": false}
df['highway-L/100km']= 235/df["highway-mpg"]
df.head(2)
```

# Data Normalization <a id="normalization"></a>

Scale variables to the same range with simple feature scaling.

```python
plt.style.use('seaborn-talk')
```

```python
shape = df[['length','width','height']].copy()

fig,ax=plt.subplots(1,3,figsize=(12,5))

for n, col in enumerate(shape):
    ax[n].hist(df[col])
    ax[n].set_title(col)
```

```python jupyter={"outputs_hidden": false}
for idx, col in shape.iteritems(): 
    shape.loc[idx]=shape[idx]/col.max()

shape.head()
```

```python
df[['length','width','height']]=shape
```


# Binning <a id="binning"></a>

- `pd.cut` numerical variable "horsepower" into categories.

```python jupyter={"outputs_hidden": false}
df["horsepower"]=df["horsepower"].astype(int, copy=True)
```

```python
sns.distplot(df['horsepower'], hist=True)
```

```python jupyter={"outputs_hidden": false}
bins = np.linspace(min(df["horsepower"]), max(df["horsepower"]), 4)
group_names = ['Low', 'Medium', 'High']

df['horsepower-binned'] = pd.cut(df['horsepower'], bins, labels=group_names, include_lowest=True )

df[['horsepower','horsepower-binned']].head(5)
```

```python
df['horsepower-binned'].value_counts()
```

```python
sns.countplot(df["horsepower-binned"])  
```

# Indicator Variable <a id="dummy"></a>

- `pd.get_dummies` Get dummy variable for "fuel-type" and "aspiration" for future prediction

```python
df['fuel-type'].value_counts()
```

```python jupyter={"outputs_hidden": false}
dummy_1 = pd.get_dummies(df["fuel-type"])
dummy_1.head(2)
```

```python jupyter={"outputs_hidden": false}
df = pd.concat([df, dummy_1], axis=1)
df.drop("fuel-type", axis = 1, inplace=True)
```

```python jupyter={"outputs_hidden": false}
df.head(2)
```

```python jupyter={"outputs_hidden": false}
dummy_2=pd.get_dummies(df['aspiration'])
dummy_2.head(2)
```

```python jupyter={"outputs_hidden": false}
df = pd.concat([df, dummy_2], axis=1)
df.drop('aspiration', axis = 1, inplace=True)
```

```python
df.head(2)
```

## Final look

```python
df.head(3)
```

```python
df.isna().any().sum()
```

```python
df.describe()
```

```python
df.describe(include='object')
```

```python
# df.to_csv('data_cleaned.csv', index=False)
```

# Well Done : )
