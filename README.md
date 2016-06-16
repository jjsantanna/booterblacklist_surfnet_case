
<h2>Booter Blacklist - SURFnet's case (the Dutch NREN)</h2> 
<h3>Data analysis to support the usage of Booter blacklist (booterblacklist.com)</h3> 

<br>
<div id="TOC">
<ul>

<li><a href="#1">1. Preamble & Pre-analysis</a></li>
<ul>
<li><a href="#1.1">1.1. What are the libraries that we use to analyse the data?</a></li>
<li><a href="#1.2">1.2. How the data looks like? [loding the raw data]</a></li>
<li><a href="#1.3">1.3. What should be formated/fix/added to facilitate our analisis?</a></li>
<li><a href="#1.4">1.4. What are the outliers that should be *removed from the analysis?</a></li>
<li><a href="#1.5">1.5. Defining the NEW data frame (excluding the outlier)</a></li>
<li><a href="#1.6">1.6. How many different record types are there in the (remaining) data frame? and removing the outlier client?</a></li> 
<li><a href="#1.7">1.7. What is the first, the last record, and the time window of the data?</a></li>

</ul>

<li><a href="#2">2. The analysis</a></li>
<ul>
<li><a href="#2.0.1">Defining 4-months periods</a></li>

<li><a href="#2.1">2.1. Booters analysis</a></li>
<ul>
<li><a href="#2.1.1">2.1.1. How many distinct Booters were accessed by users in total? in Q1, Q2 and Q3?</a></li>
<li><a href="#2.1.2">2.1.2. What is distribution of the number of access to Booters?</a></li>
<li><a href="#2.1.3">2.1.3. What are the most accessed Booters? and for each 4-months period?</a></li>
<li><a href="#2.1.4">2.1.4. What are the medians of access to the most accessed Booters?</a></li>
</ul>

<li><a href="#2.2.">2.2. Users Analysis</a></li>
<ul>
<li><a href="#2.2.1.">2.2.1. How many users are in the data?</a></li>
<li><a href="#2.2.2.">2.2.2. What is the cumulative distribution of the requests per users?</a></li>
<li><a href="#2.2.3.">2.2.3. Which are the top10 clients?and how many times each top10 client request for a Booter?</a></li>
<li><a href="#2.2.4.">2.2.4. How many accesses to Booters each top 10 client performed? How is the overall distribution of access to different Booters each client performed?</a></li>
<li><a href="#2.2.5">2.2.5. What are the medians of access to the most accessed Booters?</a></li>
</ul>

</ul>
</div>


<div id="1"><h1><a href="#TOC">1. Preamble & Pre-analysis</a></h1></div>

<div id="1.1"><h2><a href="#TOC">1.1. What are the libraries that we use to analyse the data?</a></h2></div>


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline

import matplotlib.dates as mdates
import matplotlib.patches as mpatches
from mpl_toolkits.axes_grid.inset_locator import inset_axes

# import plotly.plotly as py
# from plotly.graph_objs import *
# import plotly.tools as tls

from itertools import cycle, islice

# Style
mystyle = list(islice(cycle(['s-','ro-','y^-','c*-','r-o', 'g--^', 'b:*', 'm-.D', 'k:o']), None, 18))

#This is the color template that I use. For more information https://personal.sron.nl/~pault/colourschemes.pdf
mycolors4 = list(islice(cycle(['#4477AA','#117733','#DDCC77','#CC6677']), None, 4)) # we can change the '4' len(df that you want plot)
mycolors12 = list(islice(cycle(['#332288','#6699CC','#88CCEE','#44AA99','#117733','#999933','#DDCC77','#661100','#CC6677','#AA4466','#882255','#AA4499']), None, 12))
```

<div id="1.2"><h2><a href="#TOC">1.2. How the data looks like? [loding the raw data]</a></h2></div>


```python
# Loading the data
# In pandas a dataset is loaded as "data frame" for this reason we use the acronym "df"
# We also skipp the lines that have more columns than expected ( we just need this 6 columns)
df0 = pd.read_csv('data/anon_booters.txt.gz', error_bad_lines=False, sep=',',\
                  names = ['timestamp', 'recordtype', 'client', 'in','querytype','value'])

# Presenting the first 10 lines
df0.head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>timestamp</th>
      <th>recordtype</th>
      <th>client</th>
      <th>in</th>
      <th>querytype</th>
      <th>value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1434735481</td>
      <td>Q(Q)</td>
      <td>c861aaa8307395e94c0bc1d88e9846ff1680712521986...</td>
      <td>IN</td>
      <td>A</td>
      <td>quezstresser.com.</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1434735481</td>
      <td>Q(R)</td>
      <td>c861aaa8307395e94c0bc1d88e9846ff1680712521986...</td>
      <td>IN</td>
      <td>A</td>
      <td>quezstresser.com.</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1434735481</td>
      <td>R(ANS)</td>
      <td>c861aaa8307395e94c0bc1d88e9846ff1680712521986...</td>
      <td>IN</td>
      <td>A</td>
      <td>185.62.190.40</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1434737327</td>
      <td>Q(Q)</td>
      <td>209cd9935519df117524240334f11c3ec3ee5d9932ffb...</td>
      <td>IN</td>
      <td>A</td>
      <td>booter.xyz.</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1434737327</td>
      <td>Q(R)</td>
      <td>209cd9935519df117524240334f11c3ec3ee5d9932ffb...</td>
      <td>IN</td>
      <td>A</td>
      <td>booter.xyz.</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1434737327</td>
      <td>R(ANS)</td>
      <td>209cd9935519df117524240334f11c3ec3ee5d9932ffb...</td>
      <td>IN</td>
      <td>A</td>
      <td>104.28.26.6</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1434737327</td>
      <td>R(ANS)</td>
      <td>209cd9935519df117524240334f11c3ec3ee5d9932ffb...</td>
      <td>IN</td>
      <td>A</td>
      <td>104.28.27.6</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1434737327</td>
      <td>R(ADD)</td>
      <td>209cd9935519df117524240334f11c3ec3ee5d9932ffb...</td>
      <td>UNSPECIFIED</td>
      <td>UNSPECIFIED</td>
      <td>EDNS version 0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1434738950</td>
      <td>Q(Q)</td>
      <td>c861aaa8307395e94c0bc1d88e9846ff1680712521986...</td>
      <td>IN</td>
      <td>A</td>
      <td>quezstresser.com.</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1434738950</td>
      <td>Q(R)</td>
      <td>c861aaa8307395e94c0bc1d88e9846ff1680712521986...</td>
      <td>IN</td>
      <td>A</td>
      <td>quezstresser.com.</td>
    </tr>
  </tbody>
</table>
</div>



<div id="1.3"><h2><a href="#TOC">1.3. What should be formated/fix/added to facilitate our analisis?</a></h2></div>


```python
# Converting the collumn 'value' to lower case and remove 'www'.
# "www.booter.xyz." is the same as "booter.xyz"
df0['value'] = map(lambda x: x.lower().replace('www.','',-1), df0['value'])

# Converting epoch timestamp into date.
df0['timestamp'] = pd.to_datetime(df0['timestamp'],unit='s')

# Adding a column with 1's
df0['unit'] = 1 
```

<div id="1.4"><h2><a href="#TOC">1.4. What are the outliers that should be *removed from the analysis?</a></h2></div>


```python
df0[df0['recordtype']==' Q(Q)']['client'].value_counts()[0:10]
```




     b8f093c23ba0c6f5df242e935218578262309846e7f45ad6fc8dc4c69eb23ee1    4494
     7b4f45b838b71c0b422ac872c22b31650d8a8765afcc003b8b3b6ca5b2cbed55     991
     561e20341386f1b637011983cd6b800e4e77496fd22c14843bfe8521706e68ad     968
     18f9eec93778aa08e31a56b1f416f0d8704db9a56d36f3eebefeac80c7cb2cc5     703
     8acd6af477c9b86d6ef54ee4dbf9a4910396bc7c9ad64d4c22b89e77626bf72a     586
     6936a119cf12e9551e0f08ba9e71750108f4807cc522fbb23de6b6240e07eb38     436
     0f972ef7b4abd26a505e0d0aa8d997f01a9ac75a5a4b0f8ec31852fd8979e74e     384
     3d871e5389b8434ca72af572b8885e3c9104897cb7ce0f5d5623e4c6bdcac15f     371
     7f79658a9cc4e7e34f0c8ca40de8bfa2f76fa33f6653df351da3325c9e3c8781     370
     6a08a0ed68df7495e74392068398313951d40c84b7f000d5aeedc01d86d0e1f3     346
    Name: client, dtype: int64



- The first client identified by 'b8f093c23ba0c6f5df242e935218578262309846e7f45ad6fc8dc4c69eb23ee1' is clearly an outlier. By talking with our SURFnet they disclosured that such client is a TOR exit node. They also provide the identifier of a second exit node: '4fcc1341c071b5b7e0159696766868fbb63728e7db978c153ccc47df386410c1'. Although we analyse the behaviour of both clients in the end of this notebook, we remove then from the main analysis of the dataset.

<div id="1.5"><h2><a href="#TOC">1.5. Defining the NEW data frame (excluding the outlier)</a></h2></div>   


```python
df = df0[(df0['client']!=' b8f093c23ba0c6f5df242e935218578262309846e7f45ad6fc8dc4c69eb23ee1') & \
         (df0['client']!=' 4fcc1341c071b5b7e0159696766868fbb63728e7db978c153ccc47df386410c1')]
```

<div id="1.6"><h2><a href="#TOC">1.6. How many different record types are there in the (remaining) data frame? and removing the outlier client?</a></h2></div>


```python
# Data
records_withoutliers= df0['recordtype'].value_counts()
records_withoutoutliers=df['recordtype'].value_counts()

merged = pd.concat([records_withoutliers,records_withoutoutliers], axis=1)
merged.columns = ['With Outliers','Without Outliers (2 users)']
merged_sorted = merged.sort_values(['With Outliers','Without Outliers (2 users)'], ascending=[False,True])
# min(min(e) for e in merged_sorted if e)
if (min_val== ' '):
    min_val = 0
min_val

# Plotting
fig = plt.figure()
ax = merged_sorted.plot.bar(color=mycolors4,width = .8)
ax.legend(prop={'size':10})

fig.show()
```


    <matplotlib.figure.Figure at 0x7ffa35484150>



![png](output_14_1.png)


<div id="1.7"><h2><a href="#TOC">1.7. What is the first, the last record and the time span of the data?</a></h2></div>


```python
first_record=df['timestamp'].head(1).values[0]
last_record=df['timestamp'].tail(1).values[0]

day_span=(last_record-first_record).astype('timedelta64[D]')

first_record,last_record,day_span
```




    (numpy.datetime64('2015-06-19T17:38:01.000000000'),
     numpy.datetime64('2016-06-10T20:41:47.000000000'),
     numpy.timedelta64(357,'D'))



<div id="2"><h1><a href="#TOC">2. The analysis</a></h1></div>

<div id="2.0.1"><h2><a href="#TOC">Defining 4-months periods</a></h2></div>


```python
indexed_df = df.set_index(['timestamp'])

df_q1=indexed_df['2015-06-01':'2015-09-30']
df_q2=indexed_df['2015-10-01':'2016-01-31']
df_q3=indexed_df['2016-02-01':]
```

<div id="2.1"><h1><a href="#TOC">2.1. Booters analysis</a></h1></div>

<div id="2.1.1"><h2><a href="#TOC">2.1.1. How many distinct Booters were accessed by users in total? in Q1, Q2 and Q3?</a></h2></div>



```python
total_booters_outliers=len(df0[df0['recordtype']==' Q(Q)']['value'].value_counts())

total_booters=len(df[df['recordtype']==' Q(Q)']['value'].value_counts())
booters_q1=len(df_q1[df_q1['recordtype']==' Q(Q)']['value'].value_counts())
booters_q2=len(df_q2[df_q2['recordtype']==' Q(Q)']['value'].value_counts())
booters_q3=len(df_q3[df_q3['recordtype']==' Q(Q)']['value'].value_counts())

# Printing
total_booters_outliers,total_booters,booters_q1,booters_q2,booters_q3
```




    (228, 194, 108, 120, 98)



- Looks like that the outliers (that are the TOR nodes) accessed 34 Booters never accessed by the other 'normal' users. 

<div id="2.1.2"><h2><a href="#TOC">2.1.2. What is distribution of the number of access to Booters?</a></h2></div>


```python
serie = df['value'][df['recordtype']==' Q(Q)'].value_counts().sort_values()
cum_dist = np.linspace(0.,1.,len(serie))
cdf = pd.Series(cum_dist, index=serie)

serie_q1 = df_q1['value'][df_q1['recordtype']==' Q(Q)'].value_counts().sort_values()
cum_dist_q1 = np.linspace(0.,1.,len(serie_q1))
cdf_q1 = pd.Series(cum_dist_q1, index=serie_q1)

serie_q2 = df_q2['value'][df_q2['recordtype']==' Q(Q)'].value_counts().sort_values()
cum_dist_q2 = np.linspace(0.,1.,len(serie_q2))
cdf_q2 = pd.Series(cum_dist_q2, index=serie_q2)

serie_q3 = df_q3['value'][df_q3['recordtype']==' Q(Q)'].value_counts().sort_values()
cum_dist_q3 = np.linspace(0.,1.,len(serie_q3))
cdf_q3 = pd.Series(cum_dist_q3, index=serie_q3)
```


```python
# Plot
fig = plt.figure(figsize=(8, 5))
fig.subplots_adjust(hspace=0.8,wspace=0.5)

ax1 = plt.subplot2grid((2,3), (0,1))
cdf.plot(lw=2, ax=ax1, drawstyle='steps',color='#4477aa', title="Overall access")
ax1.set_xlabel("# access")
ax1.set_ylabel("CDF of Booters")

ax2 = plt.subplot2grid((2,3), (1,0))
cdf_q1.plot(lw=2, drawstyle='steps',color='r', title="Q1")
ax2.set_xlabel("# access")
ax2.set_ylabel("CDF of Booters")

ax3 = plt.subplot2grid((2,3), (1,1))
cdf_q2.plot(lw=2, drawstyle='steps',color='g', title="Q2")
ax3.set_xlabel("# access")
ax3.set_ylabel("CDF of Booters")

ax4 = plt.subplot2grid((2,3), (1,2))
cdf_q3.plot(lw=2, drawstyle='steps',color='b', title="Q3")
ax4.set_xlabel("# access")
ax4.set_ylabel("CDF of Booters")

fig.show()
plotly.offline.iplot_mpl(fig)

```


<div id="e5e31a9f-3b80-45c8-86bd-2116cb8e5ceb" style="height: 400; width: 640px;" class="plotly-graph-div"></div><script type="text/javascript">require(["plotly"], function(Plotly) { window.PLOTLYENV=window.PLOTLYENV || {};window.PLOTLYENV.BASE_URL="https://plot.ly";Plotly.newPlot("e5e31a9f-3b80-45c8-86bd-2116cb8e5ceb", [{"name": "None", "yaxis": "y1", "mode": "lines", "xaxis": "x1", "y": [0.0, 0.0051813471502590676, 0.010362694300518135, 0.015544041450777202, 0.02072538860103627, 0.02590673575129534, 0.031088082901554404, 0.036269430051813475, 0.04145077720207254, 0.046632124352331605, 0.05181347150259068, 0.05699481865284974, 0.06217616580310881, 0.06735751295336788, 0.07253886010362695, 0.07772020725388601, 0.08290155440414508, 0.08808290155440415, 0.09326424870466321, 0.09844559585492228, 0.10362694300518135, 0.10880829015544041, 0.11398963730569948, 0.11917098445595856, 0.12435233160621761, 0.1295336787564767, 0.13471502590673576, 0.13989637305699482, 0.1450777202072539, 0.15025906735751296, 0.15544041450777202, 0.1606217616580311, 0.16580310880829016, 0.17098445595854922, 0.1761658031088083, 0.18134715025906736, 0.18652849740932642, 0.1917098445595855, 0.19689119170984457, 0.20207253886010362, 0.2072538860103627, 0.21243523316062177, 0.21761658031088082, 0.2227979274611399, 0.22797927461139897, 0.23316062176165803, 0.2383419689119171, 0.24352331606217617, 0.24870466321243523, 0.2538860103626943, 0.2590673575129534, 0.26424870466321243, 0.2694300518134715, 0.2746113989637306, 0.27979274611398963, 0.2849740932642487, 0.2901554404145078, 0.29533678756476683, 0.3005181347150259, 0.305699481865285, 0.31088082901554404, 0.3160621761658031, 0.3212435233160622, 0.32642487046632124, 0.3316062176165803, 0.3367875647668394, 0.34196891191709844, 0.3471502590673575, 0.3523316062176166, 0.35751295336787564, 0.3626943005181347, 0.3678756476683938, 0.37305699481865284, 0.37823834196891193, 0.383419689119171, 0.38860103626943004, 0.39378238341968913, 0.3989637305699482, 0.40414507772020725, 0.40932642487046633, 0.4145077720207254, 0.41968911917098445, 0.42487046632124353, 0.4300518134715026, 0.43523316062176165, 0.44041450777202074, 0.4455958549222798, 0.45077720207253885, 0.45595854922279794, 0.461139896373057, 0.46632124352331605, 0.47150259067357514, 0.4766839378238342, 0.48186528497409326, 0.48704663212435234, 0.4922279792746114, 0.49740932642487046, 0.5025906735751295, 0.5077720207253886, 0.5129533678756477, 0.5181347150259068, 0.5233160621761658, 0.5284974093264249, 0.533678756476684, 0.538860103626943, 0.5440414507772021, 0.5492227979274612, 0.5544041450777202, 0.5595854922279793, 0.5647668393782384, 0.5699481865284974, 0.5751295336787565, 0.5803108808290156, 0.5854922279792746, 0.5906735751295337, 0.5958549222797928, 0.6010362694300518, 0.6062176165803109, 0.61139896373057, 0.616580310880829, 0.6217616580310881, 0.6269430051813472, 0.6321243523316062, 0.6373056994818653, 0.6424870466321244, 0.6476683937823834, 0.6528497409326425, 0.6580310880829016, 0.6632124352331606, 0.6683937823834197, 0.6735751295336788, 0.6787564766839379, 0.6839378238341969, 0.689119170984456, 0.694300518134715, 0.6994818652849741, 0.7046632124352332, 0.7098445595854923, 0.7150259067357513, 0.7202072538860104, 0.7253886010362695, 0.7305699481865285, 0.7357512953367876, 0.7409326424870467, 0.7461139896373057, 0.7512953367875648, 0.7564766839378239, 0.7616580310880829, 0.766839378238342, 0.7720207253886011, 0.7772020725388601, 0.7823834196891192, 0.7875647668393783, 0.7927461139896373, 0.7979274611398964, 0.8031088082901555, 0.8082901554404145, 0.8134715025906736, 0.8186528497409327, 0.8238341968911918, 0.8290155440414508, 0.8341968911917099, 0.8393782383419689, 0.844559585492228, 0.8497409326424871, 0.8549222797927462, 0.8601036269430052, 0.8652849740932643, 0.8704663212435233, 0.8756476683937824, 0.8808290155440415, 0.8860103626943006, 0.8911917098445596, 0.8963730569948187, 0.9015544041450777, 0.9067357512953368, 0.9119170984455959, 0.917098445595855, 0.922279792746114, 0.9274611398963731, 0.9326424870466321, 0.9378238341968912, 0.9430051813471503, 0.9481865284974094, 0.9533678756476685, 0.9585492227979275, 0.9637305699481865, 0.9689119170984456, 0.9740932642487047, 0.9792746113989638, 0.9844559585492229, 0.9896373056994819, 0.9948186528497409, 1.0], "x": [1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 5.0, 5.0, 6.0, 6.0, 6.0, 6.0, 6.0, 7.0, 7.0, 7.0, 8.0, 8.0, 8.0, 8.0, 8.0, 9.0, 9.0, 9.0, 10.0, 10.0, 11.0, 11.0, 12.0, 12.0, 12.0, 12.0, 15.0, 17.0, 17.0, 17.0, 17.0, 18.0, 19.0, 20.0, 21.0, 22.0, 22.0, 22.0, 22.0, 23.0, 24.0, 24.0, 26.0, 26.0, 29.0, 29.0, 29.0, 30.0, 32.0, 33.0, 35.0, 37.0, 38.0, 41.0, 41.0, 48.0, 48.0, 51.0, 55.0, 56.0, 56.0, 57.0, 63.0, 66.0, 66.0, 67.0, 67.0, 89.0, 90.0, 91.0, 97.0, 106.0, 109.0, 113.0, 132.0, 143.0, 153.0, 173.0, 175.0, 176.0, 184.0, 232.0, 234.0, 243.0, 269.0, 280.0, 291.0, 293.0, 301.0, 307.0, 342.0, 348.0, 411.0, 477.0, 662.0, 760.0, 1049.0, 1769.0, 2267.0], "line": {"dash": "solid", "width": 2, "color": "rgba (68, 119, 170, 1)"}, "type": "scatter"}, {"name": "None", "yaxis": "y2", "mode": "lines", "xaxis": "x2", "y": [0.0, 0.009345794392523364, 0.018691588785046728, 0.02803738317757009, 0.037383177570093455, 0.04672897196261682, 0.05607476635514018, 0.06542056074766354, 0.07476635514018691, 0.08411214953271028, 0.09345794392523364, 0.102803738317757, 0.11214953271028036, 0.12149532710280372, 0.1308411214953271, 0.14018691588785046, 0.14953271028037382, 0.1588785046728972, 0.16822429906542055, 0.17757009345794392, 0.18691588785046728, 0.19626168224299065, 0.205607476635514, 0.21495327102803738, 0.22429906542056072, 0.23364485981308408, 0.24299065420560745, 0.2523364485981308, 0.2616822429906542, 0.27102803738317754, 0.2803738317757009, 0.2897196261682243, 0.29906542056074764, 0.308411214953271, 0.3177570093457944, 0.32710280373831774, 0.3364485981308411, 0.34579439252336447, 0.35514018691588783, 0.3644859813084112, 0.37383177570093457, 0.38317757009345793, 0.3925233644859813, 0.40186915887850466, 0.411214953271028, 0.4205607476635514, 0.42990654205607476, 0.4392523364485981, 0.44859813084112143, 0.4579439252336448, 0.46728971962616817, 0.47663551401869153, 0.4859813084112149, 0.49532710280373826, 0.5046728971962616, 0.514018691588785, 0.5233644859813084, 0.5327102803738317, 0.5420560747663551, 0.5514018691588785, 0.5607476635514018, 0.5700934579439252, 0.5794392523364486, 0.5887850467289719, 0.5981308411214953, 0.6074766355140186, 0.616822429906542, 0.6261682242990654, 0.6355140186915887, 0.6448598130841121, 0.6542056074766355, 0.6635514018691588, 0.6728971962616822, 0.6822429906542056, 0.6915887850467289, 0.7009345794392523, 0.7102803738317757, 0.719626168224299, 0.7289719626168224, 0.7383177570093458, 0.7476635514018691, 0.7570093457943925, 0.7663551401869159, 0.7757009345794392, 0.7850467289719626, 0.794392523364486, 0.8037383177570093, 0.8130841121495327, 0.822429906542056, 0.8317757009345794, 0.8411214953271028, 0.8504672897196262, 0.8598130841121495, 0.8691588785046729, 0.8785046728971962, 0.8878504672897196, 0.8971962616822429, 0.9065420560747662, 0.9158878504672896, 0.925233644859813, 0.9345794392523363, 0.9439252336448597, 0.9532710280373831, 0.9626168224299064, 0.9719626168224298, 0.9813084112149532, 0.9906542056074765, 1.0], "x": [1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 4.0, 5.0, 5.0, 6.0, 6.0, 6.0, 6.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 8.0, 9.0, 9.0, 10.0, 10.0, 11.0, 11.0, 12.0, 13.0, 14.0, 15.0, 15.0, 15.0, 15.0, 16.0, 20.0, 20.0, 22.0, 22.0, 23.0, 24.0, 25.0, 25.0, 26.0, 27.0, 28.0, 29.0, 30.0, 31.0, 37.0, 37.0, 38.0, 41.0, 50.0, 53.0, 55.0, 57.0, 59.0, 65.0, 66.0, 74.0, 87.0, 96.0, 96.0, 98.0, 105.0, 123.0, 149.0, 165.0, 194.0, 216.0, 368.0, 453.0], "line": {"dash": "solid", "width": 2, "color": "rgba (255, 0, 0, 1)"}, "type": "scatter"}, {"name": "None", "yaxis": "y3", "mode": "lines", "xaxis": "x3", "y": [0.0, 0.008403361344537815, 0.01680672268907563, 0.025210084033613446, 0.03361344537815126, 0.04201680672268907, 0.05042016806722689, 0.058823529411764705, 0.06722689075630252, 0.07563025210084033, 0.08403361344537814, 0.09243697478991596, 0.10084033613445378, 0.1092436974789916, 0.11764705882352941, 0.12605042016806722, 0.13445378151260504, 0.14285714285714285, 0.15126050420168066, 0.15966386554621848, 0.1680672268907563, 0.1764705882352941, 0.18487394957983191, 0.19327731092436973, 0.20168067226890757, 0.21008403361344538, 0.2184873949579832, 0.226890756302521, 0.23529411764705882, 0.24369747899159663, 0.25210084033613445, 0.26050420168067223, 0.2689075630252101, 0.2773109243697479, 0.2857142857142857, 0.29411764705882354, 0.3025210084033613, 0.31092436974789917, 0.31932773109243695, 0.3277310924369748, 0.3361344537815126, 0.3445378151260504, 0.3529411764705882, 0.36134453781512604, 0.36974789915966383, 0.37815126050420167, 0.38655462184873945, 0.3949579831932773, 0.40336134453781514, 0.4117647058823529, 0.42016806722689076, 0.42857142857142855, 0.4369747899159664, 0.4453781512605042, 0.453781512605042, 0.4621848739495798, 0.47058823529411764, 0.4789915966386554, 0.48739495798319327, 0.49579831932773105, 0.5042016806722689, 0.5126050420168067, 0.5210084033613445, 0.5294117647058824, 0.5378151260504201, 0.5462184873949579, 0.5546218487394958, 0.5630252100840336, 0.5714285714285714, 0.5798319327731092, 0.5882352941176471, 0.5966386554621849, 0.6050420168067226, 0.6134453781512604, 0.6218487394957983, 0.6302521008403361, 0.6386554621848739, 0.6470588235294117, 0.6554621848739496, 0.6638655462184874, 0.6722689075630252, 0.680672268907563, 0.6890756302521008, 0.6974789915966386, 0.7058823529411764, 0.7142857142857143, 0.7226890756302521, 0.7310924369747899, 0.7394957983193277, 0.7478991596638656, 0.7563025210084033, 0.7647058823529411, 0.7731092436974789, 0.7815126050420168, 0.7899159663865546, 0.7983193277310924, 0.8067226890756303, 0.8151260504201681, 0.8235294117647058, 0.8319327731092436, 0.8403361344537815, 0.8487394957983193, 0.8571428571428571, 0.8655462184873949, 0.8739495798319328, 0.8823529411764706, 0.8907563025210083, 0.8991596638655461, 0.907563025210084, 0.9159663865546218, 0.9243697478991596, 0.9327731092436974, 0.9411764705882353, 0.9495798319327731, 0.9579831932773109, 0.9663865546218487, 0.9747899159663865, 0.9831932773109243, 0.9915966386554621, 1.0], "x": [1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 5.0, 5.0, 6.0, 6.0, 6.0, 6.0, 7.0, 7.0, 8.0, 8.0, 8.0, 8.0, 9.0, 9.0, 9.0, 12.0, 12.0, 12.0, 13.0, 13.0, 13.0, 14.0, 14.0, 15.0, 15.0, 16.0, 16.0, 21.0, 22.0, 24.0, 28.0, 30.0, 30.0, 31.0, 34.0, 36.0, 46.0, 46.0, 50.0, 53.0, 65.0, 67.0, 75.0, 84.0, 92.0, 94.0, 99.0, 107.0, 127.0, 133.0, 133.0, 143.0, 149.0, 183.0, 184.0, 225.0, 232.0, 252.0, 277.0, 313.0, 371.0, 907.0, 927.0], "line": {"dash": "solid", "width": 2, "color": "rgba (0, 127, 0, 1)"}, "type": "scatter"}, {"name": "None", "yaxis": "y4", "mode": "lines", "xaxis": "x4", "y": [0.0, 0.010309278350515464, 0.020618556701030927, 0.030927835051546393, 0.041237113402061855, 0.05154639175257732, 0.061855670103092786, 0.07216494845360824, 0.08247422680412371, 0.09278350515463918, 0.10309278350515463, 0.1134020618556701, 0.12371134020618557, 0.13402061855670103, 0.14432989690721648, 0.15463917525773196, 0.16494845360824742, 0.17525773195876287, 0.18556701030927836, 0.1958762886597938, 0.20618556701030927, 0.21649484536082475, 0.2268041237113402, 0.23711340206185566, 0.24742268041237114, 0.25773195876288657, 0.26804123711340205, 0.27835051546391754, 0.28865979381443296, 0.29896907216494845, 0.30927835051546393, 0.31958762886597936, 0.32989690721649484, 0.3402061855670103, 0.35051546391752575, 0.36082474226804123, 0.3711340206185567, 0.38144329896907214, 0.3917525773195876, 0.4020618556701031, 0.41237113402061853, 0.422680412371134, 0.4329896907216495, 0.44329896907216493, 0.4536082474226804, 0.4639175257731959, 0.4742268041237113, 0.4845360824742268, 0.4948453608247423, 0.5051546391752577, 0.5154639175257731, 0.5257731958762887, 0.5360824742268041, 0.5463917525773195, 0.5567010309278351, 0.5670103092783505, 0.5773195876288659, 0.5876288659793815, 0.5979381443298969, 0.6082474226804123, 0.6185567010309279, 0.6288659793814433, 0.6391752577319587, 0.6494845360824743, 0.6597938144329897, 0.6701030927835051, 0.6804123711340206, 0.6907216494845361, 0.7010309278350515, 0.711340206185567, 0.7216494845360825, 0.7319587628865979, 0.7422680412371134, 0.7525773195876289, 0.7628865979381443, 0.7731958762886598, 0.7835051546391752, 0.7938144329896907, 0.8041237113402062, 0.8144329896907216, 0.8247422680412371, 0.8350515463917526, 0.845360824742268, 0.8556701030927835, 0.865979381443299, 0.8762886597938144, 0.8865979381443299, 0.8969072164948454, 0.9072164948453608, 0.9175257731958762, 0.9278350515463918, 0.9381443298969072, 0.9484536082474226, 0.9587628865979382, 0.9690721649484536, 0.979381443298969, 0.9896907216494846, 1.0], "x": [1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 4.0, 4.0, 4.0, 4.0, 5.0, 6.0, 6.0, 7.0, 7.0, 7.0, 8.0, 8.0, 8.0, 8.0, 9.0, 10.0, 10.0, 11.0, 11.0, 12.0, 14.0, 14.0, 16.0, 18.0, 19.0, 27.0, 37.0, 41.0, 41.0, 41.0, 42.0, 42.0, 43.0, 43.0, 70.0, 71.0, 71.0, 79.0, 81.0, 87.0, 91.0, 101.0, 128.0, 134.0, 136.0, 140.0, 143.0, 224.0, 243.0, 253.0, 494.0, 581.0, 887.0], "line": {"dash": "solid", "width": 2, "color": "rgba (0, 0, 255, 1)"}, "type": "scatter"}], {"autosize": false, "width": 640, "xaxis4": {"tickfont": {"size": 10.0}, "domain": [0.75, 1.0], "title": "# access", "ticks": "inside", "showgrid": false, "range": [1.0, 887.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 10, "type": "linear", "anchor": "y4", "side": "bottom"}, "xaxis3": {"tickfont": {"size": 10.0}, "domain": [0.375, 0.625], "title": "# access", "ticks": "inside", "showgrid": false, "range": [1.0, 927.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 6, "type": "linear", "anchor": "y3", "side": "bottom"}, "xaxis2": {"tickfont": {"size": 10.0}, "domain": [0.0, 0.24999999999999997], "title": "# access", "ticks": "inside", "showgrid": false, "range": [1.0, 453.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 6, "type": "linear", "anchor": "y2", "side": "bottom"}, "xaxis1": {"tickfont": {"size": 10.0}, "domain": [0.375, 0.625], "title": "# access", "ticks": "inside", "showgrid": false, "range": [1.0, 2267.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 6, "type": "linear", "anchor": "y1", "side": "bottom"}, "height": 400, "yaxis1": {"tickfont": {"size": 10.0}, "domain": [0.6428571428571428, 1.0], "title": "CDF of Booters", "ticks": "inside", "showgrid": false, "range": [0.0, 1.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 6, "type": "linear", "anchor": "x1", "side": "left"}, "yaxis2": {"tickfont": {"size": 10.0}, "domain": [0.0, 0.35714285714285715], "title": "CDF of Booters", "ticks": "inside", "showgrid": false, "range": [0.0, 1.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 6, "type": "linear", "anchor": "x2", "side": "left"}, "yaxis3": {"tickfont": {"size": 10.0}, "domain": [0.0, 0.35714285714285715], "title": "CDF of Booters", "ticks": "inside", "showgrid": false, "range": [0.0, 1.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 6, "type": "linear", "anchor": "x3", "side": "left"}, "yaxis4": {"tickfont": {"size": 10.0}, "domain": [0.0, 0.35714285714285715], "title": "CDF of Booters", "ticks": "inside", "showgrid": false, "range": [0.0, 1.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 6, "type": "linear", "anchor": "x4", "side": "left"}, "hovermode": "closest", "showlegend": false, "margin": {"b": 50, "r": 63, "pad": 0, "t": 39, "l": 80}, "annotations": [{"yanchor": "bottom", "xref": "paper", "xanchor": "center", "yref": "paper", "text": "Overall access", "y": 1.0325116112897463, "x": 0.49899396378269617, "font": {"color": "#000000", "size": 12.0}, "showarrow": false}, {"yanchor": "bottom", "xref": "paper", "xanchor": "center", "yref": "paper", "text": "Q1", "y": 0.3917215332006329, "x": 0.12474849094567404, "font": {"color": "#000000", "size": 12.0}, "showarrow": false}, {"yanchor": "bottom", "xref": "paper", "xanchor": "center", "yref": "paper", "text": "Q2", "y": 0.3917215332006329, "x": 0.49899396378269617, "font": {"color": "#000000", "size": 12.0}, "showarrow": false}, {"yanchor": "bottom", "xref": "paper", "xanchor": "center", "yref": "paper", "text": "Q3", "y": 0.3917215332006329, "x": 0.8732394366197183, "font": {"color": "#000000", "size": 12.0}, "showarrow": false}]}, {"linkText": "Export to plot.ly", "showLink": true})});</script>


<div id="2.1.3"><h2><a href="#TOC">2.1.3. What are the most accessed Booters? and for each 4-months period?</a></h2></div>


```python
top10booters_count = df['value'][df['recordtype']==' Q(Q)'].value_counts()[:10]
top10booters=top10booters_count.index.tolist()

top10booters_q1_count = df_q1['value'][df_q1['recordtype']==' Q(Q)'] .value_counts()[:10]
top10booters_q1=top10booters_q1_count.index.tolist()

top10booters_q2_count = df_q2['value'][df_q2['recordtype']==' Q(Q)'].value_counts()[:10]
top10booters_q2=top10booters_q2_count.index.tolist()

top10booters_q3_count = df_q3['value'][df_q3['recordtype']==' Q(Q)'] .value_counts()[:10]
top10booters_q3=top10booters_q3_count.index.tolist()

q1=pd.DataFrame(top10booters_q1)
q2=pd.DataFrame(top10booters_q2)
q3=pd.DataFrame(top10booters_q3)
alltop10=list(pd.concat([q1,q2,q3]).drop_duplicates().values.flatten())

booters_perweek = pd.pivot_table(df[df['recordtype']==' Q(Q)'],\
                    index=pd.Grouper(key='timestamp',freq='7D'), \
                    columns='value', \
                    values='unit', \
                    aggfunc=np.sum, \
                    fill_value=0)

booters_permonth = pd.pivot_table(df[df['recordtype']==' Q(Q)'],\
                    index=pd.Grouper(key='timestamp',freq='1M'), \
                    columns='value', \
                    values='unit', \
                    aggfunc=np.sum, \
                    fill_value=0)
```


```python
fig = plt.figure(figsize=(10, 10))
fig.subplots_adjust(hspace=2,wspace=0.3)

ax1 = plt.subplot2grid((4,3), (0,0), colspan=3)
booters_perweek[alltop10].plot(ax=ax1,style=mystyle, title="WEEKLY access to the most accessed Booters (from top 10 Booters in Q1, Q2, and Q3)", legend=False)
ax1.legend(loc='center left',prop={'size':10},bbox_to_anchor=(1.01, -1.7),numpoints=1)
ax1.set_xlabel("")

ax2 = plt.subplot2grid((4,3), (1,0), colspan=3)
booters_permonth[alltop10].plot(ax=ax2,style=mystyle, title="MONTHLY access to the most accessed Booters (from top 10 Booters in Q1, Q2, and Q3)", legend=False)
ax2.set_xlabel("")

ax3 = plt.subplot2grid((4,3), (2,0))
top10booters_q1_count.plot.bar(color=mycolors12,ax=ax3, title="Q1 [Jun/Sep]",ylim=(0,1000))

ax4 = plt.subplot2grid((4,3), (2,1))
top10booters_q2_count.plot.bar(color=mycolors12,ax=ax4, title="Q2 [Oct/Jan]")

ax5 = plt.subplot2grid((4,3), (2,2))
top10booters_q3_count.plot.bar(color=mycolors12,ax=ax5, title="Q3 [Feb/May]",ylim=(0,1000))

ax6 = plt.subplot2grid((4,3), (3,0), colspan=3)
merged = pd.concat([top10booters_count, top10booters_q1_count,top10booters_q2_count,top10booters_q3_count], axis=1)
merged.columns = ['Overall','Q1', 'Q2','Q3']
merged.sort_values(['Overall','Q1'], ascending=[False,True]).plot.bar(color=mycolors4,ax=ax6,logy=True,width = .8)
ax6.legend(loc='center left',prop={'size':10},bbox_to_anchor=(1.01, 0.5),numpoints=1)

# fig.savefig('~/Desktop/booters_per_quarter.eps', bbox_inches='tight',format='eps', dpi=1200)
```




    <matplotlib.legend.Legend at 0x7ffa588f31d0>




![png](output_29_1.png)


<div id="2.1.4"><h2><a href="#TOC">2.1.4. What are the medians of access to the most accessed Booters?</a></h2></div>


```python
medians = pd.concat([booters_perweek[alltop10].median(),booters_permonth[alltop10].median()], axis=1)
medians.columns = ['Weekly','Monthly']
medians.sort_values(['Monthly','Weekly'], ascending=[False,True])
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Weekly</th>
      <th>Monthly</th>
    </tr>
    <tr>
      <th>value</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>booter.xyz.</th>
      <td>44.5</td>
      <td>185.0</td>
    </tr>
    <tr>
      <th>mostwantedhf.info.</th>
      <td>28.0</td>
      <td>138.0</td>
    </tr>
    <tr>
      <th>inboot.me.</th>
      <td>17.0</td>
      <td>91.0</td>
    </tr>
    <tr>
      <th>ipstresser.com.</th>
      <td>13.0</td>
      <td>58.0</td>
    </tr>
    <tr>
      <th>vbooter.org.</th>
      <td>11.5</td>
      <td>49.0</td>
    </tr>
    <tr>
      <th>quezstresser.com.</th>
      <td>7.0</td>
      <td>38.0</td>
    </tr>
    <tr>
      <th>ragebooter.net.</th>
      <td>3.0</td>
      <td>26.0</td>
    </tr>
    <tr>
      <th>ddos.city.</th>
      <td>3.0</td>
      <td>16.0</td>
    </tr>
    <tr>
      <th>powerstresser.com.</th>
      <td>2.0</td>
      <td>14.0</td>
    </tr>
    <tr>
      <th>titaniumstresser.net.</th>
      <td>2.0</td>
      <td>14.0</td>
    </tr>
    <tr>
      <th>boot4free.com.</th>
      <td>1.0</td>
      <td>13.0</td>
    </tr>
    <tr>
      <th>vdos-s.com.</th>
      <td>2.5</td>
      <td>12.0</td>
    </tr>
    <tr>
      <th>iddos.net.</th>
      <td>0.0</td>
      <td>9.0</td>
    </tr>
    <tr>
      <th>ragebooter.com.</th>
      <td>1.5</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>ipstresstest.com.</th>
      <td>0.0</td>
      <td>6.0</td>
    </tr>
    <tr>
      <th>purestress.net.</th>
      <td>0.0</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>xplodestresser.pw.</th>
      <td>0.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>booter.eu.</th>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>



<div id="2.2."><h1><a href="#TOC">2.2. Users Analysis</a></h1></div>

<div id="2.2.1."><h2><a href="#TOC">2.2.1. How many users are in the data?</a></h2></div>


```python
total_users_outliers=len(df0[df0['recordtype']==' Q(Q)']['client'].value_counts())

total_users=len(df[df['recordtype']==' Q(Q)']['client'].value_counts())
users_q1=len(df_q1[df_q1['recordtype']==' Q(Q)']['client'].value_counts())
users_q2=len(df_q2[df_q2['recordtype']==' Q(Q)']['client'].value_counts())
users_q3=len(df_q3[df_q3['recordtype']==' Q(Q)']['client'].value_counts())

# Printing
total_users_outliers,total_users,users_q1,users_q2,users_q3
```




    (450, 448, 180, 245, 207)



<div id="2.2.2."><h2><a href="#TOC">2.2.2. What is the cumulative distribution of the requests per users?</a></h2></div>


```python
serie = df['client'][df['recordtype']==' Q(Q)'].value_counts().sort_values()
cum_dist = np.linspace(0.,1.,len(serie))
cdf = pd.Series(cum_dist, index=serie)

serie_q1 = df_q1['client'][df_q1['recordtype']==' Q(Q)'].value_counts().sort_values()
cum_dist_q1 = np.linspace(0.,1.,len(serie_q1))
cdf_q1 = pd.Series(cum_dist_q1, index=serie_q1)

serie_q2 = df_q2['client'][df_q2['recordtype']==' Q(Q)'].value_counts().sort_values()
cum_dist_q2 = np.linspace(0.,1.,len(serie_q2))
cdf_q2 = pd.Series(cum_dist_q2, index=serie_q2)

serie_q3 = df_q3['client'][df_q3['recordtype']==' Q(Q)'].value_counts().sort_values()
cum_dist_q3 = np.linspace(0.,1.,len(serie_q3))
cdf_q3 = pd.Series(cum_dist_q3, index=serie_q3)
```


```python
# Plot
fig = plt.figure(figsize=(8, 5))
fig.subplots_adjust(hspace=0.6,wspace=0.5)

ax1 = plt.subplot2grid((2,3), (0,1))
cdf.plot(lw=2, ax=ax1, drawstyle='steps',color='#4477aa', title="Overall access")
ax1.set_xlabel("# access")
ax1.set_ylabel("CDF of users")

ax2 = plt.subplot2grid((2,3), (1,0))
cdf_q1.plot(lw=2, drawstyle='steps',color='r', title="Q2")
ax2.set_xlabel("# access")
ax2.set_ylabel("CDF of users")

ax3 = plt.subplot2grid((2,3), (1,1))
cdf_q2.plot(lw=2, drawstyle='steps',color='g', title="Q1")
ax3.set_xlabel("# access")
ax3.set_ylabel("CDF of users")

ax4 = plt.subplot2grid((2,3), (1,2))
cdf_q3.plot(lw=2, drawstyle='steps',color='b', title="Q3")
ax4.set_xlabel("# access")
ax4.set_ylabel("CDF of users")

fig.show()
plotly.offline.iplot_mpl(fig )
```


<div id="3d81af37-f6f6-4ba3-abad-7d52b5e6dd3e" style="height: 400; width: 640px;" class="plotly-graph-div"></div><script type="text/javascript">require(["plotly"], function(Plotly) { window.PLOTLYENV=window.PLOTLYENV || {};window.PLOTLYENV.BASE_URL="https://plot.ly";Plotly.newPlot("3d81af37-f6f6-4ba3-abad-7d52b5e6dd3e", [{"name": "None", "yaxis": "y1", "mode": "lines", "xaxis": "x1", "y": [0.0, 0.0022371364653243847, 0.0044742729306487695, 0.006711409395973154, 0.008948545861297539, 0.011185682326621923, 0.013422818791946308, 0.015659955257270694, 0.017897091722595078, 0.020134228187919462, 0.022371364653243846, 0.024608501118568233, 0.026845637583892617, 0.029082774049217, 0.03131991051454139, 0.03355704697986577, 0.035794183445190156, 0.03803131991051454, 0.040268456375838924, 0.04250559284116331, 0.04474272930648769, 0.04697986577181208, 0.049217002237136466, 0.05145413870246085, 0.053691275167785234, 0.05592841163310962, 0.058165548098434, 0.060402684563758385, 0.06263982102908278, 0.06487695749440715, 0.06711409395973154, 0.06935123042505592, 0.07158836689038031, 0.0738255033557047, 0.07606263982102908, 0.07829977628635347, 0.08053691275167785, 0.08277404921700224, 0.08501118568232661, 0.087248322147651, 0.08948545861297538, 0.09172259507829977, 0.09395973154362416, 0.09619686800894854, 0.09843400447427293, 0.10067114093959731, 0.1029082774049217, 0.10514541387024608, 0.10738255033557047, 0.10961968680089486, 0.11185682326621924, 0.11409395973154363, 0.116331096196868, 0.1185682326621924, 0.12080536912751677, 0.12304250559284116, 0.12527964205816555, 0.12751677852348994, 0.1297539149888143, 0.1319910514541387, 0.1342281879194631, 0.13646532438478748, 0.13870246085011184, 0.14093959731543623, 0.14317673378076062, 0.14541387024608501, 0.1476510067114094, 0.14988814317673377, 0.15212527964205816, 0.15436241610738255, 0.15659955257270694, 0.1588366890380313, 0.1610738255033557, 0.16331096196868009, 0.16554809843400448, 0.16778523489932887, 0.17002237136465323, 0.17225950782997762, 0.174496644295302, 0.1767337807606264, 0.17897091722595077, 0.18120805369127516, 0.18344519015659955, 0.18568232662192394, 0.18791946308724833, 0.1901565995525727, 0.19239373601789708, 0.19463087248322147, 0.19686800894854586, 0.19910514541387025, 0.20134228187919462, 0.203579418344519, 0.2058165548098434, 0.2080536912751678, 0.21029082774049215, 0.21252796420581654, 0.21476510067114093, 0.21700223713646533, 0.21923937360178972, 0.22147651006711408, 0.22371364653243847, 0.22595078299776286, 0.22818791946308725, 0.23042505592841162, 0.232662192393736, 0.2348993288590604, 0.2371364653243848, 0.23937360178970918, 0.24161073825503354, 0.24384787472035793, 0.24608501118568232, 0.2483221476510067, 0.2505592841163311, 0.25279642058165547, 0.2550335570469799, 0.25727069351230425, 0.2595078299776286, 0.26174496644295303, 0.2639821029082774, 0.26621923937360176, 0.2684563758389262, 0.27069351230425054, 0.27293064876957496, 0.2751677852348993, 0.2774049217002237, 0.2796420581655481, 0.28187919463087246, 0.2841163310961969, 0.28635346756152125, 0.2885906040268456, 0.29082774049217003, 0.2930648769574944, 0.2953020134228188, 0.2975391498881432, 0.29977628635346754, 0.30201342281879195, 0.3042505592841163, 0.30648769574944074, 0.3087248322147651, 0.31096196868008946, 0.3131991051454139, 0.31543624161073824, 0.3176733780760626, 0.319910514541387, 0.3221476510067114, 0.3243847874720358, 0.32662192393736017, 0.32885906040268453, 0.33109619686800895, 0.3333333333333333, 0.33557046979865773, 0.3378076062639821, 0.34004474272930646, 0.3422818791946309, 0.34451901565995524, 0.34675615212527966, 0.348993288590604, 0.3512304250559284, 0.3534675615212528, 0.35570469798657717, 0.35794183445190153, 0.36017897091722595, 0.3624161073825503, 0.36465324384787473, 0.3668903803131991, 0.36912751677852346, 0.3713646532438479, 0.37360178970917224, 0.37583892617449666, 0.378076062639821, 0.3803131991051454, 0.3825503355704698, 0.38478747203579416, 0.3870246085011186, 0.38926174496644295, 0.3914988814317673, 0.39373601789709173, 0.3959731543624161, 0.3982102908277405, 0.4004474272930649, 0.40268456375838924, 0.40492170022371365, 0.407158836689038, 0.4093959731543624, 0.4116331096196868, 0.41387024608501116, 0.4161073825503356, 0.41834451901565994, 0.4205816554809843, 0.4228187919463087, 0.4250559284116331, 0.4272930648769575, 0.42953020134228187, 0.43176733780760623, 0.43400447427293065, 0.436241610738255, 0.43847874720357943, 0.4407158836689038, 0.44295302013422816, 0.4451901565995526, 0.44742729306487694, 0.44966442953020136, 0.4519015659955257, 0.4541387024608501, 0.4563758389261745, 0.45861297539149887, 0.46085011185682323, 0.46308724832214765, 0.465324384787472, 0.46756152125279643, 0.4697986577181208, 0.47203579418344516, 0.4742729306487696, 0.47651006711409394, 0.47874720357941836, 0.4809843400447427, 0.4832214765100671, 0.4854586129753915, 0.48769574944071586, 0.4899328859060403, 0.49217002237136465, 0.494407158836689, 0.4966442953020134, 0.4988814317673378, 0.5011185682326622, 0.5033557046979865, 0.5055928411633109, 0.5078299776286354, 0.5100671140939598, 0.5123042505592841, 0.5145413870246085, 0.5167785234899329, 0.5190156599552572, 0.5212527964205816, 0.5234899328859061, 0.5257270693512304, 0.5279642058165548, 0.5302013422818792, 0.5324384787472035, 0.5346756152125279, 0.5369127516778524, 0.5391498881431768, 0.5413870246085011, 0.5436241610738255, 0.5458612975391499, 0.5480984340044742, 0.5503355704697986, 0.5525727069351231, 0.5548098434004474, 0.5570469798657718, 0.5592841163310962, 0.5615212527964206, 0.5637583892617449, 0.5659955257270693, 0.5682326621923938, 0.5704697986577181, 0.5727069351230425, 0.5749440715883669, 0.5771812080536912, 0.5794183445190156, 0.5816554809843401, 0.5838926174496644, 0.5861297539149888, 0.5883668903803132, 0.5906040268456376, 0.5928411633109619, 0.5950782997762863, 0.5973154362416108, 0.5995525727069351, 0.6017897091722595, 0.6040268456375839, 0.6062639821029082, 0.6085011185682326, 0.610738255033557, 0.6129753914988815, 0.6152125279642058, 0.6174496644295302, 0.6196868008948546, 0.6219239373601789, 0.6241610738255033, 0.6263982102908278, 0.6286353467561521, 0.6308724832214765, 0.6331096196868009, 0.6353467561521252, 0.6375838926174496, 0.639821029082774, 0.6420581655480985, 0.6442953020134228, 0.6465324384787472, 0.6487695749440716, 0.6510067114093959, 0.6532438478747203, 0.6554809843400448, 0.6577181208053691, 0.6599552572706935, 0.6621923937360179, 0.6644295302013423, 0.6666666666666666, 0.668903803131991, 0.6711409395973155, 0.6733780760626398, 0.6756152125279642, 0.6778523489932886, 0.6800894854586129, 0.6823266219239373, 0.6845637583892618, 0.6868008948545861, 0.6890380313199105, 0.6912751677852349, 0.6935123042505593, 0.6957494407158836, 0.697986577181208, 0.7002237136465325, 0.7024608501118568, 0.7046979865771812, 0.7069351230425056, 0.7091722595078299, 0.7114093959731543, 0.7136465324384788, 0.7158836689038031, 0.7181208053691275, 0.7203579418344519, 0.7225950782997763, 0.7248322147651006, 0.727069351230425, 0.7293064876957495, 0.7315436241610738, 0.7337807606263982, 0.7360178970917226, 0.7382550335570469, 0.7404921700223713, 0.7427293064876958, 0.7449664429530202, 0.7472035794183445, 0.7494407158836689, 0.7516778523489933, 0.7539149888143176, 0.756152125279642, 0.7583892617449665, 0.7606263982102908, 0.7628635346756152, 0.7651006711409396, 0.7673378076062639, 0.7695749440715883, 0.7718120805369127, 0.7740492170022372, 0.7762863534675615, 0.7785234899328859, 0.7807606263982103, 0.7829977628635346, 0.785234899328859, 0.7874720357941835, 0.7897091722595078, 0.7919463087248322, 0.7941834451901566, 0.796420581655481, 0.7986577181208053, 0.8008948545861297, 0.8031319910514542, 0.8053691275167785, 0.8076062639821029, 0.8098434004474273, 0.8120805369127516, 0.814317673378076, 0.8165548098434005, 0.8187919463087248, 0.8210290827740492, 0.8232662192393736, 0.825503355704698, 0.8277404921700223, 0.8299776286353467, 0.8322147651006712, 0.8344519015659955, 0.8366890380313199, 0.8389261744966443, 0.8411633109619686, 0.843400447427293, 0.8456375838926175, 0.8478747203579419, 0.8501118568232662, 0.8523489932885906, 0.854586129753915, 0.8568232662192393, 0.8590604026845637, 0.8612975391498882, 0.8635346756152125, 0.8657718120805369, 0.8680089485458613, 0.8702460850111856, 0.87248322147651, 0.8747203579418344, 0.8769574944071589, 0.8791946308724832, 0.8814317673378076, 0.883668903803132, 0.8859060402684563, 0.8881431767337807, 0.8903803131991052, 0.8926174496644295, 0.8948545861297539, 0.8970917225950783, 0.8993288590604027, 0.901565995525727, 0.9038031319910514, 0.9060402684563759, 0.9082774049217002, 0.9105145413870246, 0.912751677852349, 0.9149888143176733, 0.9172259507829977, 0.9194630872483222, 0.9217002237136465, 0.9239373601789709, 0.9261744966442953, 0.9284116331096197, 0.930648769574944, 0.9328859060402684, 0.9351230425055929, 0.9373601789709172, 0.9395973154362416, 0.941834451901566, 0.9440715883668903, 0.9463087248322147, 0.9485458612975392, 0.9507829977628636, 0.9530201342281879, 0.9552572706935123, 0.9574944071588367, 0.959731543624161, 0.9619686800894854, 0.9642058165548099, 0.9664429530201342, 0.9686800894854586, 0.970917225950783, 0.9731543624161073, 0.9753914988814317, 0.9776286353467561, 0.9798657718120806, 0.9821029082774049, 0.9843400447427293, 0.9865771812080537, 0.988814317673378, 0.9910514541387024, 0.9932885906040269, 0.9955257270693512, 0.9977628635346756, 1.0], "x": [1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 8.0, 8.0, 8.0, 8.0, 8.0, 8.0, 8.0, 8.0, 8.0, 9.0, 10.0, 10.0, 11.0, 11.0, 11.0, 11.0, 11.0, 11.0, 12.0, 12.0, 12.0, 12.0, 13.0, 13.0, 13.0, 13.0, 14.0, 14.0, 14.0, 15.0, 15.0, 15.0, 15.0, 15.0, 16.0, 16.0, 16.0, 16.0, 17.0, 17.0, 17.0, 18.0, 18.0, 19.0, 20.0, 20.0, 20.0, 20.0, 20.0, 21.0, 21.0, 21.0, 22.0, 23.0, 24.0, 24.0, 25.0, 25.0, 25.0, 25.0, 26.0, 26.0, 27.0, 29.0, 29.0, 30.0, 30.0, 30.0, 30.0, 32.0, 34.0, 34.0, 35.0, 42.0, 42.0, 48.0, 49.0, 55.0, 59.0], "line": {"dash": "solid", "width": 2, "color": "rgba (68, 119, 170, 1)"}, "type": "scatter"}, {"name": "None", "yaxis": "y2", "mode": "lines", "xaxis": "x2", "y": [0.0, 0.00558659217877095, 0.0111731843575419, 0.01675977653631285, 0.0223463687150838, 0.02793296089385475, 0.0335195530726257, 0.03910614525139665, 0.0446927374301676, 0.05027932960893855, 0.0558659217877095, 0.06145251396648045, 0.0670391061452514, 0.07262569832402235, 0.0782122905027933, 0.08379888268156425, 0.0893854748603352, 0.09497206703910614, 0.1005586592178771, 0.10614525139664804, 0.111731843575419, 0.11731843575418995, 0.1229050279329609, 0.12849162011173185, 0.1340782122905028, 0.13966480446927373, 0.1452513966480447, 0.15083798882681565, 0.1564245810055866, 0.16201117318435754, 0.1675977653631285, 0.17318435754189945, 0.1787709497206704, 0.18435754189944134, 0.18994413407821228, 0.19553072625698326, 0.2011173184357542, 0.20670391061452514, 0.2122905027932961, 0.21787709497206706, 0.223463687150838, 0.22905027932960895, 0.2346368715083799, 0.24022346368715083, 0.2458100558659218, 0.25139664804469275, 0.2569832402234637, 0.26256983240223464, 0.2681564245810056, 0.2737430167597765, 0.27932960893854747, 0.28491620111731847, 0.2905027932960894, 0.29608938547486036, 0.3016759776536313, 0.30726256983240224, 0.3128491620111732, 0.31843575418994413, 0.3240223463687151, 0.329608938547486, 0.335195530726257, 0.34078212290502796, 0.3463687150837989, 0.35195530726256985, 0.3575418994413408, 0.36312849162011174, 0.3687150837988827, 0.3743016759776536, 0.37988826815642457, 0.38547486033519557, 0.3910614525139665, 0.39664804469273746, 0.4022346368715084, 0.40782122905027934, 0.4134078212290503, 0.41899441340782123, 0.4245810055865922, 0.4301675977653631, 0.4357541899441341, 0.44134078212290506, 0.446927374301676, 0.45251396648044695, 0.4581005586592179, 0.46368715083798884, 0.4692737430167598, 0.4748603351955307, 0.48044692737430167, 0.48603351955307267, 0.4916201117318436, 0.49720670391061456, 0.5027932960893855, 0.5083798882681564, 0.5139664804469274, 0.5195530726256984, 0.5251396648044693, 0.5307262569832403, 0.5363128491620112, 0.5418994413407822, 0.547486033519553, 0.553072625698324, 0.5586592178770949, 0.5642458100558659, 0.5698324022346369, 0.5754189944134078, 0.5810055865921788, 0.5865921787709497, 0.5921787709497207, 0.5977653631284916, 0.6033519553072626, 0.6089385474860335, 0.6145251396648045, 0.6201117318435755, 0.6256983240223464, 0.6312849162011174, 0.6368715083798883, 0.6424581005586593, 0.6480446927374302, 0.6536312849162011, 0.659217877094972, 0.664804469273743, 0.670391061452514, 0.6759776536312849, 0.6815642458100559, 0.6871508379888268, 0.6927374301675978, 0.6983240223463687, 0.7039106145251397, 0.7094972067039106, 0.7150837988826816, 0.7206703910614526, 0.7262569832402235, 0.7318435754189945, 0.7374301675977654, 0.7430167597765364, 0.7486033519553073, 0.7541899441340782, 0.7597765363128491, 0.7653631284916201, 0.7709497206703911, 0.776536312849162, 0.782122905027933, 0.7877094972067039, 0.7932960893854749, 0.7988826815642458, 0.8044692737430168, 0.8100558659217877, 0.8156424581005587, 0.8212290502793297, 0.8268156424581006, 0.8324022346368716, 0.8379888268156425, 0.8435754189944135, 0.8491620111731844, 0.8547486033519553, 0.8603351955307262, 0.8659217877094972, 0.8715083798882682, 0.8770949720670391, 0.8826815642458101, 0.888268156424581, 0.893854748603352, 0.8994413407821229, 0.9050279329608939, 0.9106145251396648, 0.9162011173184358, 0.9217877094972068, 0.9273743016759777, 0.9329608938547487, 0.9385474860335196, 0.9441340782122906, 0.9497206703910615, 0.9553072625698324, 0.9608938547486033, 0.9664804469273743, 0.9720670391061453, 0.9776536312849162, 0.9832402234636872, 0.9888268156424581, 0.9944134078212291, 1.0], "x": [1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 7.0, 7.0, 8.0, 8.0, 8.0, 8.0, 8.0, 9.0, 9.0, 10.0, 10.0, 11.0, 11.0, 12.0, 13.0, 13.0, 13.0, 13.0, 15.0, 15.0, 16.0, 16.0, 17.0, 18.0, 18.0, 18.0, 18.0, 18.0, 18.0, 19.0, 20.0, 21.0, 21.0, 22.0, 23.0, 24.0, 24.0, 25.0, 26.0, 27.0, 28.0, 30.0, 31.0, 33.0, 34.0, 37.0, 39.0, 40.0, 43.0, 48.0, 49.0, 51.0, 63.0, 67.0, 69.0, 87.0, 90.0, 102.0, 102.0, 104.0, 120.0, 132.0, 224.0, 277.0, 305.0, 370.0], "line": {"dash": "solid", "width": 2, "color": "rgba (255, 0, 0, 1)"}, "type": "scatter"}, {"name": "None", "yaxis": "y3", "mode": "lines", "xaxis": "x3", "y": [0.0, 0.004098360655737705, 0.00819672131147541, 0.012295081967213115, 0.01639344262295082, 0.020491803278688527, 0.02459016393442623, 0.028688524590163935, 0.03278688524590164, 0.036885245901639344, 0.04098360655737705, 0.045081967213114756, 0.04918032786885246, 0.05327868852459017, 0.05737704918032787, 0.06147540983606558, 0.06557377049180328, 0.06967213114754099, 0.07377049180327869, 0.0778688524590164, 0.0819672131147541, 0.0860655737704918, 0.09016393442622951, 0.09426229508196722, 0.09836065573770492, 0.10245901639344263, 0.10655737704918034, 0.11065573770491804, 0.11475409836065574, 0.11885245901639345, 0.12295081967213116, 0.12704918032786885, 0.13114754098360656, 0.13524590163934427, 0.13934426229508198, 0.1434426229508197, 0.14754098360655737, 0.15163934426229508, 0.1557377049180328, 0.1598360655737705, 0.1639344262295082, 0.16803278688524592, 0.1721311475409836, 0.1762295081967213, 0.18032786885245902, 0.18442622950819673, 0.18852459016393444, 0.19262295081967215, 0.19672131147540983, 0.20081967213114754, 0.20491803278688525, 0.20901639344262296, 0.21311475409836067, 0.21721311475409838, 0.2213114754098361, 0.22540983606557377, 0.22950819672131148, 0.2336065573770492, 0.2377049180327869, 0.2418032786885246, 0.24590163934426232, 0.25, 0.2540983606557377, 0.2581967213114754, 0.26229508196721313, 0.26639344262295084, 0.27049180327868855, 0.27459016393442626, 0.27868852459016397, 0.2827868852459017, 0.2868852459016394, 0.29098360655737704, 0.29508196721311475, 0.29918032786885246, 0.30327868852459017, 0.3073770491803279, 0.3114754098360656, 0.3155737704918033, 0.319672131147541, 0.3237704918032787, 0.3278688524590164, 0.33196721311475413, 0.33606557377049184, 0.34016393442622955, 0.3442622950819672, 0.3483606557377049, 0.3524590163934426, 0.35655737704918034, 0.36065573770491804, 0.36475409836065575, 0.36885245901639346, 0.3729508196721312, 0.3770491803278689, 0.3811475409836066, 0.3852459016393443, 0.389344262295082, 0.39344262295081966, 0.3975409836065574, 0.4016393442622951, 0.4057377049180328, 0.4098360655737705, 0.4139344262295082, 0.4180327868852459, 0.42213114754098363, 0.42622950819672134, 0.43032786885245905, 0.43442622950819676, 0.43852459016393447, 0.4426229508196722, 0.44672131147540983, 0.45081967213114754, 0.45491803278688525, 0.45901639344262296, 0.46311475409836067, 0.4672131147540984, 0.4713114754098361, 0.4754098360655738, 0.4795081967213115, 0.4836065573770492, 0.4877049180327869, 0.49180327868852464, 0.49590163934426235, 0.5, 0.5040983606557378, 0.5081967213114754, 0.5122950819672132, 0.5163934426229508, 0.5204918032786886, 0.5245901639344263, 0.5286885245901639, 0.5327868852459017, 0.5368852459016393, 0.5409836065573771, 0.5450819672131147, 0.5491803278688525, 0.5532786885245902, 0.5573770491803279, 0.5614754098360656, 0.5655737704918034, 0.569672131147541, 0.5737704918032788, 0.5778688524590164, 0.5819672131147541, 0.5860655737704918, 0.5901639344262295, 0.5942622950819673, 0.5983606557377049, 0.6024590163934427, 0.6065573770491803, 0.6106557377049181, 0.6147540983606558, 0.6188524590163935, 0.6229508196721312, 0.6270491803278689, 0.6311475409836066, 0.6352459016393442, 0.639344262295082, 0.6434426229508197, 0.6475409836065574, 0.6516393442622951, 0.6557377049180328, 0.6598360655737705, 0.6639344262295083, 0.6680327868852459, 0.6721311475409837, 0.6762295081967213, 0.6803278688524591, 0.6844262295081968, 0.6885245901639344, 0.6926229508196722, 0.6967213114754098, 0.7008196721311476, 0.7049180327868853, 0.709016393442623, 0.7131147540983607, 0.7172131147540984, 0.7213114754098361, 0.7254098360655739, 0.7295081967213115, 0.7336065573770493, 0.7377049180327869, 0.7418032786885246, 0.7459016393442623, 0.75, 0.7540983606557378, 0.7581967213114754, 0.7622950819672132, 0.7663934426229508, 0.7704918032786886, 0.7745901639344263, 0.778688524590164, 0.7827868852459017, 0.7868852459016393, 0.7909836065573771, 0.7950819672131147, 0.7991803278688525, 0.8032786885245902, 0.8073770491803279, 0.8114754098360656, 0.8155737704918034, 0.819672131147541, 0.8237704918032788, 0.8278688524590164, 0.8319672131147542, 0.8360655737704918, 0.8401639344262295, 0.8442622950819673, 0.8483606557377049, 0.8524590163934427, 0.8565573770491803, 0.8606557377049181, 0.8647540983606558, 0.8688524590163935, 0.8729508196721312, 0.8770491803278689, 0.8811475409836066, 0.8852459016393444, 0.889344262295082, 0.8934426229508197, 0.8975409836065574, 0.9016393442622951, 0.9057377049180328, 0.9098360655737705, 0.9139344262295083, 0.9180327868852459, 0.9221311475409837, 0.9262295081967213, 0.9303278688524591, 0.9344262295081968, 0.9385245901639345, 0.9426229508196722, 0.9467213114754098, 0.9508196721311476, 0.9549180327868853, 0.959016393442623, 0.9631147540983607, 0.9672131147540984, 0.9713114754098361, 0.9754098360655739, 0.9795081967213115, 0.9836065573770493, 0.9877049180327869, 0.9918032786885247, 0.9959016393442623, 1.0], "x": [1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 7.0, 8.0, 8.0, 8.0, 8.0, 8.0, 8.0, 8.0, 8.0, 8.0, 8.0, 8.0, 8.0, 8.0, 9.0, 9.0, 9.0, 9.0, 10.0, 10.0, 10.0, 10.0, 10.0, 10.0, 10.0, 10.0, 11.0, 11.0, 11.0, 11.0, 12.0, 12.0, 12.0, 13.0, 14.0, 16.0, 17.0, 18.0, 18.0, 19.0, 19.0, 19.0, 19.0, 19.0, 19.0, 20.0, 20.0, 20.0, 21.0, 21.0, 22.0, 22.0, 22.0, 23.0, 23.0, 23.0, 23.0, 24.0, 25.0, 25.0, 26.0, 26.0, 27.0, 28.0, 29.0, 30.0, 31.0, 32.0, 39.0, 40.0, 41.0, 43.0, 44.0, 44.0, 48.0, 49.0, 51.0, 51.0, 52.0, 55.0, 57.0, 66.0, 67.0, 71.0, 71.0, 75.0, 75.0, 76.0, 79.0, 81.0, 87.0, 89.0, 123.0, 154.0, 160.0, 164.0, 172.0, 200.0, 203.0, 210.0, 219.0, 291.0, 323.0, 399.0, 554.0], "line": {"dash": "solid", "width": 2, "color": "rgba (0, 127, 0, 1)"}, "type": "scatter"}, {"name": "None", "yaxis": "y4", "mode": "lines", "xaxis": "x4", "y": [0.0, 0.0048543689320388345, 0.009708737864077669, 0.014563106796116504, 0.019417475728155338, 0.02427184466019417, 0.029126213592233007, 0.03398058252427184, 0.038834951456310676, 0.04368932038834951, 0.04854368932038834, 0.05339805825242718, 0.058252427184466014, 0.06310679611650485, 0.06796116504854369, 0.07281553398058252, 0.07766990291262135, 0.08252427184466019, 0.08737864077669902, 0.09223300970873785, 0.09708737864077668, 0.10194174757281553, 0.10679611650485436, 0.1116504854368932, 0.11650485436893203, 0.12135922330097086, 0.1262135922330097, 0.13106796116504854, 0.13592233009708737, 0.1407766990291262, 0.14563106796116504, 0.15048543689320387, 0.1553398058252427, 0.16019417475728154, 0.16504854368932037, 0.1699029126213592, 0.17475728155339804, 0.17961165048543687, 0.1844660194174757, 0.18932038834951453, 0.19417475728155337, 0.19902912621359223, 0.20388349514563106, 0.2087378640776699, 0.21359223300970873, 0.21844660194174756, 0.2233009708737864, 0.22815533980582522, 0.23300970873786406, 0.2378640776699029, 0.24271844660194172, 0.24757281553398056, 0.2524271844660194, 0.25728155339805825, 0.2621359223300971, 0.2669902912621359, 0.27184466019417475, 0.2766990291262136, 0.2815533980582524, 0.28640776699029125, 0.2912621359223301, 0.2961165048543689, 0.30097087378640774, 0.3058252427184466, 0.3106796116504854, 0.31553398058252424, 0.3203883495145631, 0.3252427184466019, 0.33009708737864074, 0.3349514563106796, 0.3398058252427184, 0.34466019417475724, 0.3495145631067961, 0.3543689320388349, 0.35922330097087374, 0.36407766990291257, 0.3689320388349514, 0.37378640776699024, 0.37864077669902907, 0.3834951456310679, 0.38834951456310673, 0.3932038834951456, 0.39805825242718446, 0.4029126213592233, 0.4077669902912621, 0.41262135922330095, 0.4174757281553398, 0.4223300970873786, 0.42718446601941745, 0.4320388349514563, 0.4368932038834951, 0.44174757281553395, 0.4466019417475728, 0.4514563106796116, 0.45631067961165045, 0.4611650485436893, 0.4660194174757281, 0.47087378640776695, 0.4757281553398058, 0.4805825242718446, 0.48543689320388345, 0.4902912621359223, 0.4951456310679611, 0.49999999999999994, 0.5048543689320388, 0.5097087378640777, 0.5145631067961165, 0.5194174757281553, 0.5242718446601942, 0.529126213592233, 0.5339805825242718, 0.5388349514563107, 0.5436893203883495, 0.5485436893203883, 0.5533980582524272, 0.558252427184466, 0.5631067961165048, 0.5679611650485437, 0.5728155339805825, 0.5776699029126213, 0.5825242718446602, 0.587378640776699, 0.5922330097087378, 0.5970873786407767, 0.6019417475728155, 0.6067961165048543, 0.6116504854368932, 0.616504854368932, 0.6213592233009708, 0.6262135922330097, 0.6310679611650485, 0.6359223300970873, 0.6407766990291262, 0.645631067961165, 0.6504854368932038, 0.6553398058252426, 0.6601941747572815, 0.6650485436893203, 0.6699029126213591, 0.674757281553398, 0.6796116504854368, 0.6844660194174756, 0.6893203883495145, 0.6941747572815533, 0.6990291262135921, 0.703883495145631, 0.7087378640776698, 0.7135922330097086, 0.7184466019417475, 0.7233009708737863, 0.7281553398058251, 0.733009708737864, 0.7378640776699028, 0.7427184466019416, 0.7475728155339805, 0.7524271844660193, 0.7572815533980581, 0.762135922330097, 0.7669902912621358, 0.7718446601941746, 0.7766990291262135, 0.7815533980582524, 0.7864077669902912, 0.7912621359223301, 0.7961165048543689, 0.8009708737864077, 0.8058252427184466, 0.8106796116504854, 0.8155339805825242, 0.8203883495145631, 0.8252427184466019, 0.8300970873786407, 0.8349514563106796, 0.8398058252427184, 0.8446601941747572, 0.8495145631067961, 0.8543689320388349, 0.8592233009708737, 0.8640776699029126, 0.8689320388349514, 0.8737864077669902, 0.8786407766990291, 0.8834951456310679, 0.8883495145631067, 0.8932038834951456, 0.8980582524271844, 0.9029126213592232, 0.9077669902912621, 0.9126213592233009, 0.9174757281553397, 0.9223300970873786, 0.9271844660194174, 0.9320388349514562, 0.9368932038834951, 0.9417475728155339, 0.9466019417475727, 0.9514563106796116, 0.9563106796116504, 0.9611650485436892, 0.9660194174757281, 0.9708737864077669, 0.9757281553398057, 0.9805825242718446, 0.9854368932038834, 0.9902912621359222, 0.9951456310679611, 1.0], "x": [1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 8.0, 8.0, 8.0, 8.0, 8.0, 8.0, 9.0, 9.0, 9.0, 9.0, 10.0, 10.0, 10.0, 10.0, 11.0, 11.0, 11.0, 11.0, 11.0, 12.0, 12.0, 12.0, 13.0, 13.0, 14.0, 14.0, 14.0, 15.0, 15.0, 16.0, 17.0, 19.0, 20.0, 21.0, 22.0, 23.0, 23.0, 24.0, 24.0, 25.0, 25.0, 28.0, 29.0, 30.0, 30.0, 35.0, 36.0, 37.0, 38.0, 38.0, 39.0, 40.0, 41.0, 41.0, 42.0, 42.0, 46.0, 48.0, 48.0, 49.0, 50.0, 52.0, 56.0, 59.0, 75.0, 78.0, 85.0, 85.0, 86.0, 90.0, 92.0, 110.0, 115.0, 122.0, 127.0, 130.0, 133.0, 143.0, 157.0, 202.0, 213.0, 368.0, 380.0], "line": {"dash": "solid", "width": 2, "color": "rgba (0, 0, 255, 1)"}, "type": "scatter"}], {"autosize": false, "width": 640, "xaxis4": {"tickfont": {"size": 10.0}, "domain": [0.75, 1.0], "title": "# access", "ticks": "inside", "showgrid": false, "range": [1.0, 380.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 9, "type": "linear", "anchor": "y4", "side": "bottom"}, "xaxis3": {"tickfont": {"size": 10.0}, "domain": [0.375, 0.625], "title": "# access", "ticks": "inside", "showgrid": false, "range": [1.0, 554.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 7, "type": "linear", "anchor": "y3", "side": "bottom"}, "xaxis2": {"tickfont": {"size": 10.0}, "domain": [0.0, 0.24999999999999997], "title": "# access", "ticks": "inside", "showgrid": false, "range": [1.0, 370.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 9, "type": "linear", "anchor": "y2", "side": "bottom"}, "xaxis1": {"tickfont": {"size": 10.0}, "domain": [0.375, 0.625], "title": "# access", "ticks": "inside", "showgrid": false, "range": [1.0, 59.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 7, "type": "linear", "anchor": "y1", "side": "bottom"}, "height": 400, "yaxis1": {"tickfont": {"size": 10.0}, "domain": [0.6153846153846154, 1.0], "title": "CDF of users", "ticks": "inside", "showgrid": false, "range": [0.0, 1.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 6, "type": "linear", "anchor": "x1", "side": "left"}, "yaxis2": {"tickfont": {"size": 10.0}, "domain": [0.0, 0.3846153846153846], "title": "CDF of users", "ticks": "inside", "showgrid": false, "range": [0.0, 1.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 6, "type": "linear", "anchor": "x2", "side": "left"}, "yaxis3": {"tickfont": {"size": 10.0}, "domain": [0.0, 0.3846153846153846], "title": "CDF of users", "ticks": "inside", "showgrid": false, "range": [0.0, 1.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 6, "type": "linear", "anchor": "x3", "side": "left"}, "yaxis4": {"tickfont": {"size": 10.0}, "domain": [0.0, 0.3846153846153846], "title": "CDF of users", "ticks": "inside", "showgrid": false, "range": [0.0, 1.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 6, "type": "linear", "anchor": "x4", "side": "left"}, "hovermode": "closest", "showlegend": false, "margin": {"b": 50, "r": 63, "pad": 0, "t": 39, "l": 80}, "annotations": [{"yanchor": "bottom", "xref": "paper", "xanchor": "center", "yref": "paper", "text": "Overall access", "y": 1.0325116112897463, "x": 0.49899396378269617, "font": {"color": "#000000", "size": 12.0}, "showarrow": false}, {"yanchor": "bottom", "xref": "paper", "xanchor": "center", "yref": "paper", "text": "Q2", "y": 0.4191057245719626, "x": 0.12474849094567404, "font": {"color": "#000000", "size": 12.0}, "showarrow": false}, {"yanchor": "bottom", "xref": "paper", "xanchor": "center", "yref": "paper", "text": "Q1", "y": 0.4191057245719626, "x": 0.49899396378269617, "font": {"color": "#000000", "size": 12.0}, "showarrow": false}, {"yanchor": "bottom", "xref": "paper", "xanchor": "center", "yref": "paper", "text": "Q3", "y": 0.4191057245719626, "x": 0.8732394366197183, "font": {"color": "#000000", "size": 12.0}, "showarrow": false}]}, {"linkText": "Export to plot.ly", "showLink": true})});</script>


<div id="2.2.3."><h2><a href="#TOC">2.2.3. Which are the top10 clients?and how many times each top10 client request for a Booter?</a></h2></div>


```python
top10users_count = df['client'][df['recordtype']==' Q(Q)'].value_counts()[:10]
top10users=top10users_count.index.tolist()

top10users_q1_count = df_q1['client'][df_q1['recordtype']==' Q(Q)'] .value_counts()[:10]
top10users_q1=top10users_q1_count.index.tolist()

top10users_q2_count = df_q2['client'][df_q2['recordtype']==' Q(Q)'].value_counts()[:10]
top10users_q2=top10users_q2_count.index.tolist()

top10users_q3_count = df_q3['client'][df_q3['recordtype']==' Q(Q)'] .value_counts()[:10]
top10users_q3=top10users_q3_count.index.tolist()

q1=pd.DataFrame(top10users_q1)
q2=pd.DataFrame(top10users_q2)
q3=pd.DataFrame(top10users_q3)
alltop10=list(pd.concat([q1,q2,q3]).drop_duplicates().values.flatten())

users_perweek = pd.pivot_table(df[df['recordtype']==' Q(Q)'],\
                    index=pd.Grouper(key='timestamp',freq='7D'), \
                    columns='client', \
                    values='unit', \
                    aggfunc=np.sum, \
                    fill_value=0)

users_permonth = pd.pivot_table(df[df['recordtype']==' Q(Q)'],\
                    index=pd.Grouper(key='timestamp',freq='1M'), \
                    columns='client', \
                    values='unit', \
                    aggfunc=np.sum, \
                    fill_value=0)
```


```python
fig = plt.figure(figsize=(10, 10))
fig.subplots_adjust(hspace=0.8,wspace=0.3)

ax1 = plt.subplot2grid((4,3), (0,0), colspan=3)
users_perweek[alltop10].plot(ax=ax1,style=mystyle, title="WEEKLY access to the most accessed Booters (from top 10 Booters in Q1, Q2, and Q3)", legend=False)
ax1.legend(loc='center left',prop={'size':10},bbox_to_anchor=(1.01, -1.1),numpoints=1)
ax1.set_xlabel("")

ax2 = plt.subplot2grid((4,3), (1,0), colspan=3)
users_permonth[alltop10].plot(ax=ax2,style=mystyle, title="MONTHLY access to the most accessed Booters (from top 10 Booters in Q1, Q2, and Q3)", legend=False)
ax2.set_xlabel("")

# ax3 = plt.subplot2grid((4,3), (2,0))
# top10users_q1_count.plot.bar(color=mycolors12,ax=ax3, title="Q1 [Jun/Sep]",ylim=(0,1000))

# ax4 = plt.subplot2grid((4,3), (2,1))
# top10users_q2_count.plot.bar(color=mycolors12,ax=ax4, title="Q2 [Oct/Jan]")

# ax5 = plt.subplot2grid((4,3), (2,2))
# top10users_q3_count.plot.bar(color=mycolors12,ax=ax5, title="Q3 [Feb/May]",ylim=(0,1000))

ax6 = plt.subplot2grid((4,3), (2,0), colspan=3, rowspan=2) 
merged = pd.concat([top10users_count, top10users_q1_count,top10users_q2_count,top10users_q3_count], axis=1)
merged.columns = ['Overall','Q1', 'Q2','Q3']
merged.sort_values(['Overall','Q1'], ascending=[False,True]).plot.bar(color=mycolors4,ax=ax6,logy=True,width = .8)
ax6.legend(prop={'size':10})

```




    <matplotlib.legend.Legend at 0x7ffa2fe82110>




![png](output_40_1.png)


<div id="2.2.4."><h2><a href="#TOC">2.2.4. How many accesses to Booters each top 10 client performed? How is the overall distribution of access to different Booters each client performed?</a></h2></div>


```python
# DATA (for the main plot)
ser = df[df['recordtype']==' Q(Q)'].groupby(['client', 'value']).size().reset_index().groupby('client').count().ix[:,0].sort_values()
cum_dist = np.linspace(0.,1.,len(ser))
cdf = pd.Series(cum_dist, index=ser)

# PLOT (main)
fig = plt.figure()
ax = fig.add_subplot(111)
cdf.plot(lw=2,ax=ax,drawstyle='steps',color='#4477aa', title="").set_xlabel("")
ax.set_xlabel("# distinct Booters accessed")
ax.set_ylabel("CDF of users")

# DATA (for the subplot inside)
clients_permonth = pd.pivot_table(df[df['recordtype']==' Q(Q)'], index='client', columns='value', values='unit', aggfunc=np.sum, fill_value=0)

# PLOT 
box = ax.get_position()
fig2 = plt.gcf()
fig2.transFigure.inverted().transform(ax.transAxes.transform([0,0])) ##doesn't matter this numbers
x=0.35
y=0.3
width=box.width*0.67
height=box.height*0.6
ax1 = fig2.add_axes([x,y,width,height],axisbg='w') #x,y,width,height
clients_permonth.ix[top10users].plot(kind='bar',ax=ax1, stacked=True, legend=False, color=mycolors12, title="")
ax1.axes.get_xaxis().set_ticks([])
ax1.set_xlabel("Top 10 users")
ax1.set_ylabel("# acesses to Booters")
ax1.xaxis.set_tick_params(labelsize=10)
ax1.yaxis.set_tick_params(labelsize=10)

fig.show()
#fig.savefig('/Users/santannajj/Desktop/booters_accessed_top10users.eps', bbox_inches='tight',format='eps', dpi=1200)
plotly.offline.iplot_mpl(fig )
```


<div id="a417c822-ede6-4d6a-91c2-c27621439a93" style="height: 320; width: 480px;" class="plotly-graph-div"></div><script type="text/javascript">require(["plotly"], function(Plotly) { window.PLOTLYENV=window.PLOTLYENV || {};window.PLOTLYENV.BASE_URL="https://plot.ly";Plotly.newPlot("a417c822-ede6-4d6a-91c2-c27621439a93", [{"name": "None", "yaxis": "y1", "mode": "lines", "xaxis": "x1", "y": [0.0, 0.0022371364653243847, 0.0044742729306487695, 0.006711409395973154, 0.008948545861297539, 0.011185682326621923, 0.013422818791946308, 0.015659955257270694, 0.017897091722595078, 0.020134228187919462, 0.022371364653243846, 0.024608501118568233, 0.026845637583892617, 0.029082774049217, 0.03131991051454139, 0.03355704697986577, 0.035794183445190156, 0.03803131991051454, 0.040268456375838924, 0.04250559284116331, 0.04474272930648769, 0.04697986577181208, 0.049217002237136466, 0.05145413870246085, 0.053691275167785234, 0.05592841163310962, 0.058165548098434, 0.060402684563758385, 0.06263982102908278, 0.06487695749440715, 0.06711409395973154, 0.06935123042505592, 0.07158836689038031, 0.0738255033557047, 0.07606263982102908, 0.07829977628635347, 0.08053691275167785, 0.08277404921700224, 0.08501118568232661, 0.087248322147651, 0.08948545861297538, 0.09172259507829977, 0.09395973154362416, 0.09619686800894854, 0.09843400447427293, 0.10067114093959731, 0.1029082774049217, 0.10514541387024608, 0.10738255033557047, 0.10961968680089486, 0.11185682326621924, 0.11409395973154363, 0.116331096196868, 0.1185682326621924, 0.12080536912751677, 0.12304250559284116, 0.12527964205816555, 0.12751677852348994, 0.1297539149888143, 0.1319910514541387, 0.1342281879194631, 0.13646532438478748, 0.13870246085011184, 0.14093959731543623, 0.14317673378076062, 0.14541387024608501, 0.1476510067114094, 0.14988814317673377, 0.15212527964205816, 0.15436241610738255, 0.15659955257270694, 0.1588366890380313, 0.1610738255033557, 0.16331096196868009, 0.16554809843400448, 0.16778523489932887, 0.17002237136465323, 0.17225950782997762, 0.174496644295302, 0.1767337807606264, 0.17897091722595077, 0.18120805369127516, 0.18344519015659955, 0.18568232662192394, 0.18791946308724833, 0.1901565995525727, 0.19239373601789708, 0.19463087248322147, 0.19686800894854586, 0.19910514541387025, 0.20134228187919462, 0.203579418344519, 0.2058165548098434, 0.2080536912751678, 0.21029082774049215, 0.21252796420581654, 0.21476510067114093, 0.21700223713646533, 0.21923937360178972, 0.22147651006711408, 0.22371364653243847, 0.22595078299776286, 0.22818791946308725, 0.23042505592841162, 0.232662192393736, 0.2348993288590604, 0.2371364653243848, 0.23937360178970918, 0.24161073825503354, 0.24384787472035793, 0.24608501118568232, 0.2483221476510067, 0.2505592841163311, 0.25279642058165547, 0.2550335570469799, 0.25727069351230425, 0.2595078299776286, 0.26174496644295303, 0.2639821029082774, 0.26621923937360176, 0.2684563758389262, 0.27069351230425054, 0.27293064876957496, 0.2751677852348993, 0.2774049217002237, 0.2796420581655481, 0.28187919463087246, 0.2841163310961969, 0.28635346756152125, 0.2885906040268456, 0.29082774049217003, 0.2930648769574944, 0.2953020134228188, 0.2975391498881432, 0.29977628635346754, 0.30201342281879195, 0.3042505592841163, 0.30648769574944074, 0.3087248322147651, 0.31096196868008946, 0.3131991051454139, 0.31543624161073824, 0.3176733780760626, 0.319910514541387, 0.3221476510067114, 0.3243847874720358, 0.32662192393736017, 0.32885906040268453, 0.33109619686800895, 0.3333333333333333, 0.33557046979865773, 0.3378076062639821, 0.34004474272930646, 0.3422818791946309, 0.34451901565995524, 0.34675615212527966, 0.348993288590604, 0.3512304250559284, 0.3534675615212528, 0.35570469798657717, 0.35794183445190153, 0.36017897091722595, 0.3624161073825503, 0.36465324384787473, 0.3668903803131991, 0.36912751677852346, 0.3713646532438479, 0.37360178970917224, 0.37583892617449666, 0.378076062639821, 0.3803131991051454, 0.3825503355704698, 0.38478747203579416, 0.3870246085011186, 0.38926174496644295, 0.3914988814317673, 0.39373601789709173, 0.3959731543624161, 0.3982102908277405, 0.4004474272930649, 0.40268456375838924, 0.40492170022371365, 0.407158836689038, 0.4093959731543624, 0.4116331096196868, 0.41387024608501116, 0.4161073825503356, 0.41834451901565994, 0.4205816554809843, 0.4228187919463087, 0.4250559284116331, 0.4272930648769575, 0.42953020134228187, 0.43176733780760623, 0.43400447427293065, 0.436241610738255, 0.43847874720357943, 0.4407158836689038, 0.44295302013422816, 0.4451901565995526, 0.44742729306487694, 0.44966442953020136, 0.4519015659955257, 0.4541387024608501, 0.4563758389261745, 0.45861297539149887, 0.46085011185682323, 0.46308724832214765, 0.465324384787472, 0.46756152125279643, 0.4697986577181208, 0.47203579418344516, 0.4742729306487696, 0.47651006711409394, 0.47874720357941836, 0.4809843400447427, 0.4832214765100671, 0.4854586129753915, 0.48769574944071586, 0.4899328859060403, 0.49217002237136465, 0.494407158836689, 0.4966442953020134, 0.4988814317673378, 0.5011185682326622, 0.5033557046979865, 0.5055928411633109, 0.5078299776286354, 0.5100671140939598, 0.5123042505592841, 0.5145413870246085, 0.5167785234899329, 0.5190156599552572, 0.5212527964205816, 0.5234899328859061, 0.5257270693512304, 0.5279642058165548, 0.5302013422818792, 0.5324384787472035, 0.5346756152125279, 0.5369127516778524, 0.5391498881431768, 0.5413870246085011, 0.5436241610738255, 0.5458612975391499, 0.5480984340044742, 0.5503355704697986, 0.5525727069351231, 0.5548098434004474, 0.5570469798657718, 0.5592841163310962, 0.5615212527964206, 0.5637583892617449, 0.5659955257270693, 0.5682326621923938, 0.5704697986577181, 0.5727069351230425, 0.5749440715883669, 0.5771812080536912, 0.5794183445190156, 0.5816554809843401, 0.5838926174496644, 0.5861297539149888, 0.5883668903803132, 0.5906040268456376, 0.5928411633109619, 0.5950782997762863, 0.5973154362416108, 0.5995525727069351, 0.6017897091722595, 0.6040268456375839, 0.6062639821029082, 0.6085011185682326, 0.610738255033557, 0.6129753914988815, 0.6152125279642058, 0.6174496644295302, 0.6196868008948546, 0.6219239373601789, 0.6241610738255033, 0.6263982102908278, 0.6286353467561521, 0.6308724832214765, 0.6331096196868009, 0.6353467561521252, 0.6375838926174496, 0.639821029082774, 0.6420581655480985, 0.6442953020134228, 0.6465324384787472, 0.6487695749440716, 0.6510067114093959, 0.6532438478747203, 0.6554809843400448, 0.6577181208053691, 0.6599552572706935, 0.6621923937360179, 0.6644295302013423, 0.6666666666666666, 0.668903803131991, 0.6711409395973155, 0.6733780760626398, 0.6756152125279642, 0.6778523489932886, 0.6800894854586129, 0.6823266219239373, 0.6845637583892618, 0.6868008948545861, 0.6890380313199105, 0.6912751677852349, 0.6935123042505593, 0.6957494407158836, 0.697986577181208, 0.7002237136465325, 0.7024608501118568, 0.7046979865771812, 0.7069351230425056, 0.7091722595078299, 0.7114093959731543, 0.7136465324384788, 0.7158836689038031, 0.7181208053691275, 0.7203579418344519, 0.7225950782997763, 0.7248322147651006, 0.727069351230425, 0.7293064876957495, 0.7315436241610738, 0.7337807606263982, 0.7360178970917226, 0.7382550335570469, 0.7404921700223713, 0.7427293064876958, 0.7449664429530202, 0.7472035794183445, 0.7494407158836689, 0.7516778523489933, 0.7539149888143176, 0.756152125279642, 0.7583892617449665, 0.7606263982102908, 0.7628635346756152, 0.7651006711409396, 0.7673378076062639, 0.7695749440715883, 0.7718120805369127, 0.7740492170022372, 0.7762863534675615, 0.7785234899328859, 0.7807606263982103, 0.7829977628635346, 0.785234899328859, 0.7874720357941835, 0.7897091722595078, 0.7919463087248322, 0.7941834451901566, 0.796420581655481, 0.7986577181208053, 0.8008948545861297, 0.8031319910514542, 0.8053691275167785, 0.8076062639821029, 0.8098434004474273, 0.8120805369127516, 0.814317673378076, 0.8165548098434005, 0.8187919463087248, 0.8210290827740492, 0.8232662192393736, 0.825503355704698, 0.8277404921700223, 0.8299776286353467, 0.8322147651006712, 0.8344519015659955, 0.8366890380313199, 0.8389261744966443, 0.8411633109619686, 0.843400447427293, 0.8456375838926175, 0.8478747203579419, 0.8501118568232662, 0.8523489932885906, 0.854586129753915, 0.8568232662192393, 0.8590604026845637, 0.8612975391498882, 0.8635346756152125, 0.8657718120805369, 0.8680089485458613, 0.8702460850111856, 0.87248322147651, 0.8747203579418344, 0.8769574944071589, 0.8791946308724832, 0.8814317673378076, 0.883668903803132, 0.8859060402684563, 0.8881431767337807, 0.8903803131991052, 0.8926174496644295, 0.8948545861297539, 0.8970917225950783, 0.8993288590604027, 0.901565995525727, 0.9038031319910514, 0.9060402684563759, 0.9082774049217002, 0.9105145413870246, 0.912751677852349, 0.9149888143176733, 0.9172259507829977, 0.9194630872483222, 0.9217002237136465, 0.9239373601789709, 0.9261744966442953, 0.9284116331096197, 0.930648769574944, 0.9328859060402684, 0.9351230425055929, 0.9373601789709172, 0.9395973154362416, 0.941834451901566, 0.9440715883668903, 0.9463087248322147, 0.9485458612975392, 0.9507829977628636, 0.9530201342281879, 0.9552572706935123, 0.9574944071588367, 0.959731543624161, 0.9619686800894854, 0.9642058165548099, 0.9664429530201342, 0.9686800894854586, 0.970917225950783, 0.9731543624161073, 0.9753914988814317, 0.9776286353467561, 0.9798657718120806, 0.9821029082774049, 0.9843400447427293, 0.9865771812080537, 0.988814317673378, 0.9910514541387024, 0.9932885906040269, 0.9955257270693512, 0.9977628635346756, 1.0], "x": [1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 8.0, 8.0, 8.0, 8.0, 8.0, 8.0, 8.0, 8.0, 8.0, 9.0, 10.0, 10.0, 11.0, 11.0, 11.0, 11.0, 11.0, 11.0, 12.0, 12.0, 12.0, 12.0, 13.0, 13.0, 13.0, 13.0, 14.0, 14.0, 14.0, 15.0, 15.0, 15.0, 15.0, 15.0, 16.0, 16.0, 16.0, 16.0, 17.0, 17.0, 17.0, 18.0, 18.0, 19.0, 20.0, 20.0, 20.0, 20.0, 20.0, 21.0, 21.0, 21.0, 22.0, 23.0, 24.0, 24.0, 25.0, 25.0, 25.0, 25.0, 26.0, 26.0, 27.0, 29.0, 29.0, 30.0, 30.0, 30.0, 30.0, 32.0, 34.0, 34.0, 35.0, 42.0, 42.0, 48.0, 49.0, 55.0, 59.0], "line": {"dash": "solid", "width": 2, "color": "rgba (68, 119, 170, 1)"}, "type": "scatter"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#332288", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#6699CC", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#88CCEE", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#44AA99", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#117733", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#999933", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 3.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#DDCC77", "line": {"width": 1.0}}, "xaxis": "x2", "y": [2.0, 5.0, 1.0, 2.0, 0.0, 1.0, 0.0, 34.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#661100", "line": {"width": 1.0}}, "xaxis": "x2", "y": [4.0, 0.0, 2.0, 3.0, 6.0, 0.0, 0.0, 0.0, 4.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#CC6677", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4466", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 1.0, 0.0, 1.0, 1.0, 0.0, 0.0, 0.0, 0.0, 2.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#882255", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4499", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#332288", "line": {"width": 1.0}}, "xaxis": "x2", "y": [1.0, 1.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#6699CC", "line": {"width": 1.0}}, "xaxis": "x2", "y": [2.0, 0.0, 1.0, 5.0, 2.0, 0.0, 0.0, 0.0, 1.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#88CCEE", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 1.0, 6.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#44AA99", "line": {"width": 1.0}}, "xaxis": "x2", "y": [1.0, 3.0, 2.0, 0.0, 1.0, 3.0, 0.0, 0.0, 3.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#117733", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 36.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#999933", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#DDCC77", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#661100", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#CC6677", "line": {"width": 1.0}}, "xaxis": "x2", "y": [2.0, 5.0, 4.0, 0.0, 1.0, 3.0, 0.0, 0.0, 4.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4466", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#882255", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4499", "line": {"width": 1.0}}, "xaxis": "x2", "y": [6.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#332288", "line": {"width": 1.0}}, "xaxis": "x2", "y": [39.0, 5.0, 9.0, 11.0, 5.0, 1.0, 9.0, 2.0, 5.0, 2.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#6699CC", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 44.0, 5.0, 0.0, 25.0, 2.0, 0.0, 0.0, 0.0, 50.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#88CCEE", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#44AA99", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 1.0, 0.0, 0.0, 0.0, 3.0, 0.0, 17.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#117733", "line": {"width": 1.0}}, "xaxis": "x2", "y": [11.0, 5.0, 5.0, 8.0, 2.0, 2.0, 3.0, 0.0, 0.0, 8.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#999933", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 2.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#DDCC77", "line": {"width": 1.0}}, "xaxis": "x2", "y": [176.0, 108.0, 314.0, 81.0, 73.0, 75.0, 54.0, 50.0, 61.0, 45.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#661100", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#CC6677", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4466", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#882255", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 1.0, 0.0, 2.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4499", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#332288", "line": {"width": 1.0}}, "xaxis": "x2", "y": [4.0, 2.0, 4.0, 3.0, 1.0, 3.0, 2.0, 45.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#6699CC", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 21.0, 0.0, 0.0, 0.0, 44.0, 0.0, 0.0, 2.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#88CCEE", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 2.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#44AA99", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#117733", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#999933", "line": {"width": 1.0}}, "xaxis": "x2", "y": [8.0, 4.0, 0.0, 5.0, 1.0, 11.0, 0.0, 0.0, 4.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#DDCC77", "line": {"width": 1.0}}, "xaxis": "x2", "y": [9.0, 3.0, 11.0, 2.0, 5.0, 3.0, 3.0, 0.0, 6.0, 1.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#661100", "line": {"width": 1.0}}, "xaxis": "x2", "y": [7.0, 1.0, 1.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#CC6677", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4466", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#882255", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4499", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#332288", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#6699CC", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 1.0, 2.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#88CCEE", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#44AA99", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#117733", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#999933", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#DDCC77", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#661100", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#CC6677", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4466", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#882255", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4499", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#332288", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#6699CC", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#88CCEE", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#44AA99", "line": {"width": 1.0}}, "xaxis": "x2", "y": [8.0, 4.0, 0.0, 1.0, 1.0, 2.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#117733", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#999933", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#DDCC77", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 3.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 1.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#661100", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#CC6677", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4466", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#882255", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4499", "line": {"width": 1.0}}, "xaxis": "x2", "y": [7.0, 0.0, 1.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#332288", "line": {"width": 1.0}}, "xaxis": "x2", "y": [1.0, 0.0, 12.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#6699CC", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#88CCEE", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#44AA99", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#117733", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#999933", "line": {"width": 1.0}}, "xaxis": "x2", "y": [16.0, 0.0, 0.0, 0.0, 0.0, 10.0, 0.0, 3.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#DDCC77", "line": {"width": 1.0}}, "xaxis": "x2", "y": [7.0, 25.0, 8.0, 6.0, 5.0, 2.0, 0.0, 0.0, 20.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#661100", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#CC6677", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 4.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4466", "line": {"width": 1.0}}, "xaxis": "x2", "y": [1.0, 1.0, 5.0, 0.0, 2.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#882255", "line": {"width": 1.0}}, "xaxis": "x2", "y": [16.0, 55.0, 27.0, 70.0, 33.0, 2.0, 0.0, 0.0, 21.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4499", "line": {"width": 1.0}}, "xaxis": "x2", "y": [69.0, 163.0, 65.0, 29.0, 27.0, 24.0, 17.0, 73.0, 44.0, 18.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#332288", "line": {"width": 1.0}}, "xaxis": "x2", "y": [3.0, 1.0, 0.0, 5.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#6699CC", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 24.0, 1.0, 15.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#88CCEE", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#44AA99", "line": {"width": 1.0}}, "xaxis": "x2", "y": [68.0, 29.0, 29.0, 27.0, 33.0, 22.0, 6.0, 2.0, 13.0, 10.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#117733", "line": {"width": 1.0}}, "xaxis": "x2", "y": [8.0, 45.0, 26.0, 40.0, 33.0, 11.0, 0.0, 0.0, 18.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#999933", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#DDCC77", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 1.0, 1.0, 0.0, 0.0, 51.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#661100", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#CC6677", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 1.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4466", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#882255", "line": {"width": 1.0}}, "xaxis": "x2", "y": [4.0, 2.0, 3.0, 10.0, 6.0, 0.0, 0.0, 0.0, 5.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4499", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#332288", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#6699CC", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#88CCEE", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 2.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#44AA99", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 16.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#117733", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#999933", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#DDCC77", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#661100", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#CC6677", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4466", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#882255", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4499", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#332288", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#6699CC", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#88CCEE", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#44AA99", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#117733", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#999933", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#DDCC77", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#661100", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#CC6677", "line": {"width": 1.0}}, "xaxis": "x2", "y": [2.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4466", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#882255", "line": {"width": 1.0}}, "xaxis": "x2", "y": [111.0, 251.0, 49.0, 54.0, 42.0, 46.0, 15.0, 0.0, 20.0, 32.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4499", "line": {"width": 1.0}}, "xaxis": "x2", "y": [2.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#332288", "line": {"width": 1.0}}, "xaxis": "x2", "y": [6.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 3.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#6699CC", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#88CCEE", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#44AA99", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#117733", "line": {"width": 1.0}}, "xaxis": "x2", "y": [10.0, 5.0, 5.0, 18.0, 17.0, 8.0, 1.0, 33.0, 16.0, 3.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#999933", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#DDCC77", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#661100", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#CC6677", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4466", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#882255", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4499", "line": {"width": 1.0}}, "xaxis": "x2", "y": [12.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#332288", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#6699CC", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#88CCEE", "line": {"width": 1.0}}, "xaxis": "x2", "y": [23.0, 4.0, 6.0, 5.0, 3.0, 1.0, 2.0, 0.0, 0.0, 3.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#44AA99", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#117733", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#999933", "line": {"width": 1.0}}, "xaxis": "x2", "y": [13.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#DDCC77", "line": {"width": 1.0}}, "xaxis": "x2", "y": [4.0, 0.0, 0.0, 0.0, 1.0, 3.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#661100", "line": {"width": 1.0}}, "xaxis": "x2", "y": [5.0, 0.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#CC6677", "line": {"width": 1.0}}, "xaxis": "x2", "y": [11.0, 26.0, 10.0, 21.0, 12.0, 14.0, 5.0, 0.0, 6.0, 7.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4466", "line": {"width": 1.0}}, "xaxis": "x2", "y": [1.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#882255", "line": {"width": 1.0}}, "xaxis": "x2", "y": [29.0, 8.0, 9.0, 8.0, 6.0, 3.0, 48.0, 0.0, 6.0, 2.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4499", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 1.0, 3.0, 8.0, 3.0, 0.0, 0.0, 0.0, 6.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#332288", "line": {"width": 1.0}}, "xaxis": "x2", "y": [38.0, 30.0, 14.0, 23.0, 7.0, 17.0, 11.0, 14.0, 2.0, 5.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#6699CC", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#88CCEE", "line": {"width": 1.0}}, "xaxis": "x2", "y": [7.0, 2.0, 2.0, 0.0, 2.0, 0.0, 46.0, 0.0, 3.0, 60.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#44AA99", "line": {"width": 1.0}}, "xaxis": "x2", "y": [18.0, 4.0, 8.0, 1.0, 4.0, 1.0, 74.0, 35.0, 4.0, 30.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#117733", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 2.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#999933", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#DDCC77", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#661100", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#CC6677", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4466", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#882255", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4499", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#332288", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#6699CC", "line": {"width": 1.0}}, "xaxis": "x2", "y": [15.0, 10.0, 2.0, 3.0, 2.0, 0.0, 1.0, 0.0, 2.0, 2.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#88CCEE", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#44AA99", "line": {"width": 1.0}}, "xaxis": "x2", "y": [6.0, 2.0, 4.0, 4.0, 0.0, 0.0, 5.0, 1.0, 2.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#117733", "line": {"width": 1.0}}, "xaxis": "x2", "y": [11.0, 0.0, 2.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 3.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#999933", "line": {"width": 1.0}}, "xaxis": "x2", "y": [9.0, 0.0, 1.0, 0.0, 0.0, 4.0, 0.0, 0.0, 20.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#DDCC77", "line": {"width": 1.0}}, "xaxis": "x2", "y": [1.0, 2.0, 2.0, 0.0, 13.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#661100", "line": {"width": 1.0}}, "xaxis": "x2", "y": [16.0, 11.0, 11.0, 9.0, 8.0, 3.0, 9.0, 0.0, 8.0, 7.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#CC6677", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4466", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 1.0, 0.0, 2.0, 6.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#882255", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4499", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#332288", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#6699CC", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#88CCEE", "line": {"width": 1.0}}, "xaxis": "x2", "y": [11.0, 15.0, 3.0, 13.0, 24.0, 14.0, 0.0, 0.0, 12.0, 1.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#44AA99", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 8.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#117733", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#999933", "line": {"width": 1.0}}, "xaxis": "x2", "y": [8.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#DDCC77", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#661100", "line": {"width": 1.0}}, "xaxis": "x2", "y": [1.0, 0.0, 0.0, 0.0, 1.0, 4.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#CC6677", "line": {"width": 1.0}}, "xaxis": "x2", "y": [2.0, 0.0, 0.0, 0.0, 1.0, 1.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4466", "line": {"width": 1.0}}, "xaxis": "x2", "y": [102.0, 19.0, 22.0, 14.0, 10.0, 24.0, 1.0, 0.0, 2.0, 36.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#882255", "line": {"width": 1.0}}, "xaxis": "x2", "y": [29.0, 4.0, 4.0, 49.0, 4.0, 9.0, 3.0, 0.0, 23.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4499", "line": {"width": 1.0}}, "xaxis": "x2", "y": [1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#332288", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#6699CC", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#88CCEE", "line": {"width": 1.0}}, "xaxis": "x2", "y": [1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#44AA99", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 1.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#117733", "line": {"width": 1.0}}, "xaxis": "x2", "y": [1.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#999933", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#DDCC77", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#661100", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 1.0, 0.0, 0.0, 4.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#CC6677", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 2.0, 2.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4466", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#882255", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#AA4499", "line": {"width": 1.0}}, "xaxis": "x2", "y": [4.0, 0.0, 0.0, 5.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#332288", "line": {"width": 1.0}}, "xaxis": "x2", "y": [1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}, {"opacity": 1, "orientation": "v", "yaxis": "y2", "marker": {"color": "#6699CC", "line": {"width": 1.0}}, "xaxis": "x2", "y": [0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0], "x": [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0], "type": "bar"}], {"autosize": false, "width": 480, "xaxis2": {"domain": [0.29032258064516125, 0.9603225806451614], "title": "Top 10 users", "ticks": "inside", "showgrid": false, "range": [-0.5, 9.5], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 0, "type": "linear", "anchor": "y2", "side": "bottom"}, "xaxis1": {"tickfont": {"size": 10.0}, "domain": [0.0, 1.0], "title": "# distinct Booters accessed", "ticks": "inside", "showgrid": false, "range": [1.0, 59.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 7, "type": "linear", "anchor": "y1", "side": "bottom"}, "height": 320, "yaxis1": {"tickfont": {"size": 10.0}, "domain": [0.0, 1.0], "title": "CDF of users", "ticks": "inside", "showgrid": false, "range": [0.0, 1.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 6, "type": "linear", "anchor": "x1", "side": "left"}, "yaxis2": {"tickfont": {"size": 10.0}, "domain": [0.2258064516129032, 0.8258064516129031], "title": "# acesses to Booters", "ticks": "inside", "showgrid": false, "range": [0.0, 1000.0], "titlefont": {"color": "#000000", "size": 10.0}, "mirror": "ticks", "zeroline": false, "showline": true, "nticks": 6, "type": "linear", "anchor": "x2", "side": "left"}, "bargap": 0.5, "barmode": "stack", "showlegend": false, "margin": {"b": 40, "r": 47, "pad": 0, "t": 31, "l": 60}, "hovermode": "x"}, {"linkText": "Export to plot.ly", "showLink": true})});</script>


<div id="2.2.5"><h2><a href="#TOC">2.2.5. What are the medians of access to the most accessed Booters?</a></h2></div>


```python
medians = pd.concat([users_perweek[alltop10].median(),users_permonth[alltop10].median()], axis=1)
medians.columns = ['Weekly','Monthly']
medians.sort_values(['Monthly','Weekly'], ascending=[False,True])
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Weekly</th>
      <th>Monthly</th>
    </tr>
    <tr>
      <th>client</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>7b4f45b838b71c0b422ac872c22b31650d8a8765afcc003b8b3b6ca5b2cbed55</th>
      <td>14.5</td>
      <td>80.0</td>
    </tr>
    <tr>
      <th>561e20341386f1b637011983cd6b800e4e77496fd22c14843bfe8521706e68ad</th>
      <td>13.0</td>
      <td>72.0</td>
    </tr>
    <tr>
      <th>18f9eec93778aa08e31a56b1f416f0d8704db9a56d36f3eebefeac80c7cb2cc5</th>
      <td>10.5</td>
      <td>49.0</td>
    </tr>
    <tr>
      <th>8acd6af477c9b86d6ef54ee4dbf9a4910396bc7c9ad64d4c22b89e77626bf72a</th>
      <td>3.5</td>
      <td>27.0</td>
    </tr>
    <tr>
      <th>0f972ef7b4abd26a505e0d0aa8d997f01a9ac75a5a4b0f8ec31852fd8979e74e</th>
      <td>2.5</td>
      <td>23.0</td>
    </tr>
    <tr>
      <th>6936a119cf12e9551e0f08ba9e71750108f4807cc522fbb23de6b6240e07eb38</th>
      <td>4.0</td>
      <td>23.0</td>
    </tr>
    <tr>
      <th>4778e1a1a9a899ad68cbf91e7566860ac9c94cce707fc8127e6739841f03587e</th>
      <td>0.5</td>
      <td>18.0</td>
    </tr>
    <tr>
      <th>2fe0390dd55d56e6c2889f5ccd5361b233a78d7415ab4fd2a08a9fc0c0f42b80</th>
      <td>2.5</td>
      <td>17.0</td>
    </tr>
    <tr>
      <th>a13133036389ac9dd4762142ea36d3405bd78e12402f62df483d0f91fb427db6</th>
      <td>1.0</td>
      <td>16.0</td>
    </tr>
    <tr>
      <th>6a08a0ed68df7495e74392068398313951d40c84b7f000d5aeedc01d86d0e1f3</th>
      <td>1.5</td>
      <td>16.0</td>
    </tr>
    <tr>
      <th>0fcb07b661671c1dd54f2cab5aaa44340e0f07c8812dd9a75a7c2588dcd49b27</th>
      <td>2.0</td>
      <td>16.0</td>
    </tr>
    <tr>
      <th>666ccb31666bd54d14ab078ab73d96cf04a01ec113a12dcd02e5a759019400ba</th>
      <td>1.5</td>
      <td>15.0</td>
    </tr>
    <tr>
      <th>f9a6b7ec0ccef96a70e8548e081fb92a4f281193c4a2eedf8bd554921783cc50</th>
      <td>2.0</td>
      <td>14.0</td>
    </tr>
    <tr>
      <th>6efadc240e0549394684390a59ab01912441ec3319bf7015ed2819db66edac6a</th>
      <td>2.5</td>
      <td>10.0</td>
    </tr>
    <tr>
      <th>3d871e5389b8434ca72af572b8885e3c9104897cb7ce0f5d5623e4c6bdcac15f</th>
      <td>0.0</td>
      <td>9.0</td>
    </tr>
    <tr>
      <th>f6f7fe38d406f181d374f7e4475639e95a93e3f1559c5229b3d43431aae50b05</th>
      <td>0.0</td>
      <td>5.0</td>
    </tr>
    <tr>
      <th>7f79658a9cc4e7e34f0c8ca40de8bfa2f76fa33f6653df351da3325c9e3c8781</th>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>37543c4e575dec6cfbb511cf26bde8c3e07542cf347441c5f09a5f56d0db2196</th>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>6f8914d9ef7e3200bbf91aeb493b3d458ecad654300068f6b5fece7a42ad6fec</th>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>6e23b99069b86c590dbff05d70859ca094f6bb0111f983958e2f84557e484dcb</th>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>


