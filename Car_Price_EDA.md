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

# Car Price EDA - Visualization

### Table of Content

---

 - [Preparation](#preparation)
 - [Exploratory Analysis](#analysis)
  - [Numerical Features](#num) 
      - Distplot, Heatmap, Regplot, Jointplot-[Summary1](#smr-num)
  - [Categorical Features](#cat) 
      - Boxplot, Swarmplot, Violinplot, Pointplot-[Summary2](#smr-cat)
  - [Multi-Variables](#multi) 
      - Lmplot, Pairplot
 - [Pearson Score & P-Values](#p-values)  
      - [Summary3](#smr-pvalue)
 - [Conclusion: Important Variables](#conclusion) 
 
### Libraries

- `pandas`, `numpy`, `matplotlib`, `seaborn(comprehensive)`    

---

__Author: Yue Wu__


# 1 - Preparation <a id='preparation'></a>


### Load Data

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline 
```

```python jupyter={"outputs_hidden": false}
data='./auto85/auto_clean.csv'
df = pd.read_csv(data)
df.head(2)
```

### Subset Numerical & Categorical Features

```python
df_cat = df.select_dtypes(include='object')
df_cat = pd.concat([df_cat,df['price']],axis=1)
```

```python
df_cat.shape
```

```python
df_num = df.select_dtypes(include = 'number')
```

```python
df_num.shape
```

<!-- #region -->
 
 
# 2 - Exploratory Data Analysis <a id='analysis'></a>
<!-- #endregion -->

## Numerical Features<a id="num"></a>

- distplot
- heatmap
- feature selection
- heatmap 
- regplot 


### Distribution 
- distplot

```python
plt.style.use('ggplot')
sns.set_context('talk')
```

```python
ncols=6
length = df_num.shape[1]
nrows=length//ncols if length%ncols==0 else length//ncols+1

fig, ax = plt.subplots(nrows=nrows, ncols = ncols, figsize = (22, 12))

for idx, col in enumerate(df_num): 
    i = idx//ncols
    j = idx%ncols
    sns.distplot(df_num[col], ax = ax[i][j])

plt.tight_layout()
```

### Heatmap

```python jupyter={"outputs_hidden": true}
corrmap = df_num.corr()

fig, ax = plt.subplots(figsize = (12, 10))
sns.heatmap(corrmap, annot = True, annot_kws = {'size': 8}, cmap='RdBu')

bottom, top = ax.get_ylim()
ax.set_ylim(bottom+0.5, top-0.5)
```

### Correlated Feature Selection

```python
def getCorrelatedFeature(corrdata, threshold): 
    feature = []
    value = []
    
    for i, index in enumerate(corrdata.index):  
        if abs(corrdata[index]) > threshold:  
            feature.append(index)
            value.append(corrdata[index])
            
    df = pd.DataFrame(data = value, index=feature, columns=['corr value'])
    
    return df
```

### Price-Related Features with Correlation Score > 0.5 

```python
threshold = 0.5
corr_features = getCorrelatedFeature(corrmap['price'],threshold)

print(len(corr_features))
corr_features
```

### Heatmap with Correlated Features

```python
corr_df = df_num[corr_features.index]

fig, ax = plt.subplots(figsize = (10, 8))

sns.heatmap(corr_df.corr(), annot = True, annot_kws = {'size': 12},cmap='RdYlBu')

bottom, top = ax.get_ylim()
ax.set_ylim(bottom+0.5, top-0.5)
```

### Regression Plot with Correlated Features

```python
ncols=5
length = corr_df.shape[1]
nrows=length//ncols if length%ncols==0 else length//ncols+1

fig, ax = plt.subplots(nrows=nrows, ncols = ncols, figsize = (22, 12))

for idx, col in enumerate(corr_df):  
    i=idx//ncols
    j=idx%ncols
    sns.regplot(x = 'price', y = col, data = corr_df, ax = ax[i][j])

plt.tight_layout()
```

As shown above, selected features are clearly correlated with Price.


### Jointplot 

Closely look at how high-corr score features relate to price with `kde` and `hex`


#### 1-Curb-weight & Price

```python
sns.jointplot(x='curb-weight',y='price',data=corr_df,kind='kde') 
```

Note: Curb-weight spreads mainly across 2000-3000, while price in range 0-20000.


#### 2-Engine-size & Price

```python
sns.jointplot(x='engine-size',y='price',data=corr_df,kind='hex') 
```

Note: Data mainly gather around lower engine-size and lower price part.

<!-- #region -->
### Summary1 - Numerical Predictors <a id="smr-num"></a> 

- positive correlation  
 - wheel-base
 - length
 - width
 - curb-weight
 - engine-size
 - bore
 - horsepower
 
 
- negative correlation
 - city-mpg
 - high-mpg
 
<!-- #endregion -->

---

## Categorical Features  <a id='cat'></a>
 
- boxplot
- violinplot
- swarm
- pointplot


```python
df_cat.head(2)
```

### Distribution - Boxplot
See how data spreads.

```python
rows = 5
cols = 2

fig, ax = plt.subplots(nrows=rows, ncols = cols, figsize = (12, 20))

col = df_cat.columns
index = 0

for i in range(rows):
    for j in range(cols):
        if col[index]!='price':
            index = index
        else:
            index = index +1
        sns.boxplot(x= df_cat[col[index]], y='price', data = df_cat, ax = ax[i][j])
        index +=1

plt.tight_layout()
```

### Swarmplot

See how data is gathered.

```python
df_cat6 = df_cat.drop(columns = ['make','num-of-doors','body-style','fuel-system'])
```

```python
rows = 3
cols = 2

fig, ax = plt.subplots(nrows=rows, ncols = cols, figsize = (12, 12))

col = df_cat6.columns
index = 0

for i in range(rows):
    for j in range(cols):
        if col[index]!='price':
            index = index
        else:
            index = index +1
        sns.swarmplot(x= df_cat6[col[index]], y='price', data = df_cat6, ax = ax[i][j])
        index +=1

plt.tight_layout()
```

### Violinplot
Double check distribution

```python
rows = 3
cols = 2

fig, ax = plt.subplots(nrows=rows, ncols = cols, figsize = (12, 12))

col = df_cat6.columns
index = 0

for i in range(rows):
    for j in range(cols):
        if col[index]!='price':
            index = index
        else:
            index = index +1
        sns.violinplot(x= df_cat6[col[index]], y='price', data = df_cat6, ax = ax[i][j])
        index +=1

plt.tight_layout()
```

### Pointplot
Further examine price correlation within selected groups

```python
fig=plt.figure(figsize=(10,4))

plt.subplot(1,2,1)
sns.pointplot(x='drive-wheels',y='price',data=df_cat6)

plt.subplot(1,2,2)
sns.pointplot(x='horsepower-binned',y='price',order = ['Low','Medium','High'],data=df_cat6)

plt.tight_layout()
```

Prices vary largely within __drive-wheels__ and __horsepower-binned__ groups


### Heatmap

```python
corr_df_c=df[['drive-wheels','horsepower-binned','price']]

grouped=corr_df_c.pivot_table(index='drive-wheels',
                              columns='horsepower-binned',
                              values='price',
                              aggfunc='mean',
                              fill_value=0)

sns.heatmap(grouped, cmap='RdBu')
```

<!-- #region -->

### Summary2 - Categorical Predictors <a id="smr-cat"></a>
#### Boxplot

- Bad predictors - distribution overlap(OV) or no pattern(NP)
 - make (OV)
 - num-of-doors (NP)
 - body-style (NP)
 - fuel-system (NP)

#### Swarmplot

- Bad predictors with uneven category sizes that will skew prediction
 - aspiration
 - engine-location
 - engine-type
 - num-of-cylinders


#### Potential Good Predictors
 - drive-wheels
 - horsepower-binned

<!-- #endregion -->

---

## Multi Variables <a id="multi"><a>

- Lmplot
- Pairplot


### Lmplot

```python
sns.lmplot(x='horsepower',y='price',col = 'drive-wheels',data=df) 
```

###  Pairplot <a id="pairplot"></a>

Group Correlation for Price-Correlated Features

Feature 1 to 5.

```python
sns.pairplot(corr_df.iloc[:, np.hstack(([0], range(1, 5)))], diag_kind='kde')
```

Feature 6 to 10.

```python
sns.pairplot(corr_df.iloc[:, np.hstack(([0], range(6, 10)))], diag_kind='kde')
```

Most pair groups are also somewhat correlated.


---

## 3 - Pearson Score & P-values <a id='p-values'></a>


### Calculate P-values

```python
from scipy import stats

test=corr_df[corr_df.columns.difference(['price'])]

p_value=[]
pearson_coef=[]
feature=[]

for idx, col in test.iteritems():
    pef, pv = stats.pearsonr(col, df['price'])
    
    p_value.append(pv)
    pearson_coef.append(pef)
    feature.append(idx)
    
prscoef=pd.DataFrame(zip(feature,pearson_coef,p_value), 
                    columns=['name','pearson_coef','p_value'])
prscoef
   
```

### Summary3 - P-values <a id="smr-pvalue"></a>

Since the p-value is $<$ 0.001, the correlations between initially filtered corr-features and price are __statistically significant__, and the linear relationship is __very strong (abs(coef)>0.5)__


---

## 4 - Conclusion: Important Variables  <a id='conclusion'></a>


<p>We now have a better idea of what our data looks like and which variables are important to take into account when predicting the car price. We have narrowed it down to the following variables:</p>

Continuous numerical variables:
<ul>
    <li>Length</li>
    <li>Width</li>
    <li>Curb-weight</li>
    <li>Engine-size</li>
    <li>Horsepower</li>
    <li>City-mpg</li>
    <li>Highway-mpg</li>
    <li>Wheel-base</li>
    <li>Bore</li>
</ul>
    
Categorical variables:
<ul>
    <li>Drive-wheels</li>
    <li>Horsepower-binned</li>
</ul>



# The End : )
