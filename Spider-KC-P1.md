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

```python
import requests
import scrapy
from scrapy.crawler import CrawlerProcess
from scrapy import Selector
import pandas as pd

# request data from web
url = 'https://kylecavan.com/pages/schools'
html = requests.get(url).content

# create selector
sel = Selector(text = html)
```

## Inspect the web

Collections names are under the class "school-menu"

```python
# set css path to get all link text from class "school-menu"
css = '.school-menu a::text'

# extract and show results
collections = sel.css(css).extract()
print(type(collections), len(collections))
collections[0]
```

```python
clt = pd.DataFrame(collections)
clt.head()
```

```python
#clt.to_csv('Collections.csv',index = False)
```

## Spider to get product information

```python

```

```python
# step 1 - Get collection link
clt_lks = '.school-menu a::attr(href)'

cmp_site = 'http://kylecavan.com'

links = [cmp_site + x for x in sel.css(clt_lks).extract()]
links[0]
len(links)
```

```python
links[5]
```

```python
test = 'https://kylecavan.com/collections/alabama'
alabama = requests.get(test).text
selal = Selector(text = alabama)
selal.css('h3.product-title::text').extract()

```

```python
# try spider - don't know how to show results
import scrapy
from scrapy.crawler import CrawlerProcess

class prd(scrapy.Spider):
    name = 'prd'

    def start_requests(self):
        url = 'https://kylecavan.com/pages/schools'
        yield scrapy.Request(url = url, callback= self.get_collection)

    def get_collection(self, response):

        # define css path for collections
        clt_css = '.school-menu a::attr(href)'
        cmp_site = 'http://kylecavan.com'

        clt_name = response.css(clt_css).extract()
        clt_links = [cmp_site +links for x in links]

        for c_link in clt_links:
            yield response.follow(url = c_link, callback = self.get_product)#, meta = {'collection':clt_name})
    
    def get_product(self, response):

        # define css path for products
        prd_name_css = 'h3.product-title::text'
        prd_price_css = 'span.product-price::text'

        prd_name = response.css(prd_css).extract()
        prd_price = response.css(prd_price_css).extract()
        result[clt_name] = zip(prd_name, prd_price)
        #return result#clt_name, prd_price
        #for product, price in zip(prd_name, prd_price):
            #price_product[prd_name] = prd_price
            #yield {'collection':clt_name, 'product': product, 'price':price}

```

```python


process = CrawlerProcess()
process.crawl(prd)
process.start()
process.stop()

```

dir(scrapy.Spider)
dir(products_in_collection)
dir(scrapy)

```python
scrapy.version_info
```

```python
# Create the Spider class



class DC_Description_Spider(scrapy.Spider):
  name = "dc_chapter_spider"
  # start_requests method
  def start_requests(self):
    yield scrapy.Request(url = 'datacamp.',
                         callback = self.parse_front)
  # First parsing method
  def parse_front(self, response):
    course_blocks = response.css('div.course-block')
    course_links = course_blocks.xpath('./a/@href')
    links_to_follow = course_links.extract()
    for url in links_to_follow:
      yield response.follow(url = url,
                            callback = self.parse_pages)
  # Second parsing method
  def parse_pages(self, response):
    # Create a SelectorList of the course titles text
    crs_title = response.xpath('//h1[contains(@class,"title")]/text()')
    # Extract the text and strip it clean
    crs_title_ext = crs_title.extract_first().strip()
    # Create a SelectorList of course descriptions text
    crs_descr = response.css( 'p.course__description::text' )
    # Extract the text and strip it clean
    crs_descr_ext = crs_descr.extract_first().strip()
    # Fill in the dictionary
    dc_dict[crs_title_ext] = crs_descr_ext

# Initialize the dictionary **outside** of the Spider class
dc_dict = dict()

# Run the Spider
process = CrawlerProcess()
process.crawl(DC_Description_Spider)
process.start()

# Print a preview of courses
previewCourses(dc_dict)
```

```

```python
dir(process.crawl)
```

```python

```
