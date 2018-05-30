
abundance.tsvはtarget_idの列がサンプル間で等しくそのままconcatして良さそうだけど、確証が持てないため、dg.merge()する


```python
from pandas import Series, DataFrame
import pandas as pd
import csv
import matplotlib.pyplot as plt
import numpy as np
```


```python
e1 = pd.read_table("kallisto_test_result/ERR1551404/abundance.tsv")
e1 = e1.drop(columns=['length', 'eff_length', 'est_counts'])
e1.columns = ['target_id', 'TPM_ERR1551404']
e1.head()
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
      <th>target_id</th>
      <th>TPM_ERR1551404</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ENST00000434970.2</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ENST00000448914.1</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ENST00000415118.1</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ENST00000632684.1</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ENST00000631435.1</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
e2 = pd.read_table("kallisto_test_result/ERR1551408/abundance.tsv")
e2 = e2.drop(columns=['length', 'eff_length', 'est_counts'])
e2.columns = ['target_id', 'TPM_ERR1551408']
e2.head()
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
      <th>target_id</th>
      <th>TPM_ERR1551408</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ENST00000434970.2</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ENST00000448914.1</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ENST00000415118.1</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ENST00000632684.1</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ENST00000631435.1</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
e = pd.merge(e1, e2, on='target_id')
e.head()
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
      <th>target_id</th>
      <th>TPM_ERR1551404</th>
      <th>TPM_ERR1551408</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ENST00000434970.2</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ENST00000448914.1</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ENST00000415118.1</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ENST00000632684.1</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ENST00000631435.1</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.scatter(e.TPM_ERR1551404, e.TPM_ERR1551408)
plt.xlabel('ERR1551408')
plt.ylabel('ERR1551404')
```




    Text(0,0.5,'ERR1551404')




![png](output_5_1.png)



```python
e['log_ERR1551404'] = np.log10(e['TPM_ERR1551404'] + 1)
e['log_ERR1551408'] = np.log10(e['TPM_ERR1551408'] + 1)
e['diff'] = abs(e['log_ERR1551404'] - e['log_ERR1551408'])
e.head()
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
      <th>target_id</th>
      <th>TPM_ERR1551404</th>
      <th>TPM_ERR1551408</th>
      <th>log_ERR1551404</th>
      <th>log_ERR1551408</th>
      <th>diff</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ENST00000434970.2</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ENST00000448914.1</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ENST00000415118.1</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ENST00000632684.1</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ENST00000631435.1</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.scatter(e.log_ERR1551404, e.log_ERR1551408)
plt.xlabel('ERR1551408')
plt.ylabel('ERR1551404')
```




    Text(0,0.5,'ERR1551404')




![png](output_7_1.png)


同じ散布図ですがplotlyでインタラクティブに描画する方法も試してみます。
ただし、サンプル数が多いため、かなり重いかもしれません。


```python
from plotly.offline import download_plotlyjs, init_notebook_mode, plot, iplot
import plotly.graph_objs as go
#import plotly.offline

init_notebook_mode(connected=False)  

data = [go.Scatter(
        x = e['log_ERR1551404'],
        y = e['log_ERR1551408'],
        mode = 'markers'
    )]

plot(data, filename='basic-scatter')
```


<script type='text/javascript'>if(!window.Plotly){define('plotly', function(require, exports, module) {/**
* plotly.js v1.38.0
* Copyright 2012-2018, Plotly, Inc.
* All rights reserved.
* Licensed under the MIT license
*/
});require(['plotly'], function(Plotly) {window.Plotly = Plotly;});}</script>


    /Users/greendog/miniconda3/lib/python3.6/site-packages/plotly/offline/offline.py:459: UserWarning:
    
    Your filename `basic-scatter` didn't end with .html. Adding .html to the end of your file.
    





    'file:///Volumes/Inugoya0/shizuokangs_vol1/basic-scatter.html'



ふたつのサンプルのTPMの対数の差をソートし、上位x%をファイルに書きだす。