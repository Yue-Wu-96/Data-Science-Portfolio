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

# Tag Products for E-commerce 

---

### Libraries & Keyword:

 - `Requests`
 - `Scrapy Selector(with css path)`
 - `Fuzzywuzzy`
 - `Plotly`
 - `Pandas`

### Process
 - Step 1 - Scrape Web for up-to-date collection names
 - Step 2 - Find a string score for Fuzzywuzzy
 - Step 3 - Tag collection in a new column
 - Step 4 - Deal with NAs with domain knowledge 
 - Step 5 - Concat Na and not_Na dataset
 - Wrap up 
     - What're the top 10 collections?
     - How's sales trend like?

---
__Author: Yue Wu__


### Load data & Inspect

```python
import pandas as pd 
df = pd.read_csv('kc_sales_sample.csv')
print(df.shape)
df.head()
```

## Step 1 - Scrape Web 

To get the most up-to-date product information without the mannual work of excel input and typos.

```python
import requests
import scrapy 
from scrapy import Selector

# request data from web
url = 'https://kylecavan.com/pages/schools'
html = requests.get(url).content
sel = Selector(text = html)

# define css path
css = '.school-menu a::text'
collections = sel.css(css).extract()
print(type(collections), len(collections))
```

```python
# create a collection dataframe
clt = pd.DataFrame(collections)
clt.rename(columns = {0:'Collection'}, inplace = True)
clt.head()
```

## Step 2 - Decide a string score 

The `process.extract` returns the similar string in the iterable, a score for string similarity, and its postion in a tuple.

```python
import fuzzywuzzy
from fuzzywuzzy import process

test_item = ['Villanova', 'Columbia', 'Alpha Omicron Pi']
test_score = [ process.extract(x, df['product_title'], limit = 3) for x in test_item]
test_score
```

#### Result:

__90__ is a good score, enough to filter out nuance.   
Example, `Alpha Omicron Pi` = 90, while `Alpha Delta Pi` = 86.


## Step 3 - Tag collection in a new column

```python

for item in clt['Collection']:

    matches = process.extract(item, df['product_title'], limit = df.shape[0])  
    # compare all titles with collection strings
    
    for potential_match in matches:
   
        if potential_match[1]>=90:  # the score discovered above
            match_test = df['product_title'] == potential_match[0]  # find the match in df.product_title
            df.loc[match_test, 'Collection'] = item  # tag it
    
```

```python
df.head()
```

## Step 4 - Deal with NAs

```python
na_flt = df['Collection'].isna()
not_na = df[~na_flt]
na = df[na_flt]
na.shape
```

```python
na.head()
```

### Stop and think:

NA values always occur in real business.   
Here, based on domain knowlege and company website, __MMXX__  and __Miss America__ are two other collections ~~not in school list~~.  
Other than these two, the rest are __custom__ pieces not for online sale.


## Fill Na

```python
# make filters
mx = na['product_title'].str.contains('MMXX')
ma = na['product_title'].str.contains('Miss America')

# fill na
na.loc[mx, 'Collection'] = 'MMXX'
na.loc[ma, 'Collection'] = 'Miss America'
na.loc[~(mx & ma), 'Collection'] = 'Custom'

na.head()
```

```python
na['Collection'].isna().sum()
```

## Step 5 - Concat na dataset to not_na dataset

```python
df_filled = pd.concat([not_na, na])
df_filled.reset_index(drop = True, inplace = True)

df_filled.head(3)
```

```python
df_filled.tail(3)
```

```python
df_filled.shape  # same shape as before
```

## Wrapping up 

### 1 - What're the __top 10__ collections by quantity?

```python
top10=df_filled.groupby('Collection').agg({'net_quantity':'sum'}).sort_values('net_quantity', ascending = False).head(10)

```
```python
import plotly as py
from plotly.offline import iplot
import cufflinks as cf

py.offline.init_notebook_mode(connected=True)
cf.go_offline()

top10.iplot(kind = 'bar', title='Top 10 Collections')
```




### 2 - How is the sales trend?

```python
by_time = df_filled.groupby('day').agg({'net_quantity':'sum'}).sort_values('net_quantity')
```

```python
print(by_time.index.min())
print(by_time.index.max())
```

```python
by_time.iplot(kind='bar', bins=20)
```

## The End : )
