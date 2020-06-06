# San Francisco Crime

Visualize crime spots and find a safe home.

---
### Table of Content

- [Icon Marker Map](#icon)
- [Circle Marker Map](#circle) 
- [Crime Spot Cluster Map](#cluster)
- [Crime Area Choropleth Map](#choropleth)

### Libraries

- `folium`, `geopandas`, `pandas`
---

#### Tip: Remember to zoom-in, zoom-out, hover, and click map points to see what will happen : ) - Yue


## Load Data

```python
import pandas as pd
import folium 
```


```python
path = 'https://cocl.us/sanfran_crime_dataset'
df = pd.read_csv(path, nrows=100)
df.head(3)
```


```python
loc=[37.77,-122.42]
```

## Icon Marker Map <a id="icon"></a>

```python
sf_marker=folium.Map(location=loc,zoom_start=12)

for lag, lng, time in zip(df.Y, df.X, df.Time):
    folium.Marker(
        location=[lag, lng],
        radius=5,
        popup=time,
        icon=folium.Icon(color='red'),
        tooltip='click me for crime time'
    ).add_to(sf_marker)

sf_marker
```

## Circle Marker Map <a id="circle"></a>

```python

sf_circle=folium.Map(location=loc,zoom_start=12)

for lag, lng, time in zip(df.Y, df.X, df.Time):
    folium.CircleMarker(
        location=[lag, lng],
        radius=6,
        tooltip='click me for time',
        popup=time,
        color='Yellow',
        fill=True,
        fill_color='red',
        fill_opacity=0.5,
        marker_cluster=True
    ).add_to(sf_circle)


# sf_circle.save('circle-marker.html')
sf_circle 
```

## Crime Spot Cluster Map <a id="cluster"></a>

```python
from folium import plugins
from folium.plugins import MarkerCluster

sf_cluster=folium.Map(loc,zoom_start=12)
cluster = plugins.MarkerCluster()

for lag,lng, time in zip(df.Y,df.X,df.Time):
    folium.Marker(
        location=[lag, lng],
        radius=5,
        popup=time,
        icon=folium.Icon(color='red'),
        tooltip='click me for crime time'
    ).add_to(cluster)

sf_cluster.add_child(cluster)

# sf_cluster.save('cluster-map.html')
sf_cluster
```

## Crime Area Choropleth Map <a id="choropleth"></a>


Get geo-info.

```python
import geopandas as gpd

geo = gpd.read_file('./san-francisco.json') 
print(geo.head(2))  
```

```python
incidents = df['PdDistrict'].value_counts().to_frame().reset_index() # add index
incidents.columns=['Neighborhood','Count']
incidents.head(5)
```

```python

crime_area = folium.Map(zoom_start=12, location =loc)

crime_area.choropleth(
    geo_data=geo,
    data=incidents,
    columns=['Neighborhood', 'Count'],
    key_on='feature.properties.DISTRICT',
    fill_color='YlOrRd', 
    fill_opacity=0.7, 
    line_opacity=0.2,
    legend_name='SanF'
)

folium.LayerControl().add_to(crime_area)

# crime_area.save('choropleth-map.html')
crime_area
```

### Now you know where is safe : )
 The End.
