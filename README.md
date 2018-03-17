## Data Analysis with Python. Pivot tables with Pandas

One of the first post in my blog was about Pivot tables. I'd created a library to pivot tables in my PHP scripts. The library is not very beautiful (it throws a lot of warnings), but it works. These days I'm playing with Python Data Analysis and I'm using Pandas. The purpose of this post is something that I like a lot: Learn by doing. So I want to do the same operations that I did eight years ago in the post but now with Pandas. Let's start.

I'll start with the same datasource that I used almost ten years ago. One simple recordset with cliks and number of users

I create a dataframe with this data
```python
import numpy as np
import pandas as pd

data = pd.DataFrame([
    {'host': 1, 'country': 'fr', 'year': 2010, 'month': 1, 'clicks': 123, 'users': 4},
    {'host': 1, 'country': 'fr', 'year': 2010, 'month': 2, 'clicks': 134, 'users': 5},
    {'host': 1, 'country': 'fr', 'year': 2010, 'month': 3, 'clicks': 341, 'users': 2},
    {'host': 1, 'country': 'es', 'year': 2010, 'month': 1, 'clicks': 113, 'users': 4},
    {'host': 1, 'country': 'es', 'year': 2010, 'month': 2, 'clicks': 234, 'users': 5},
    {'host': 1, 'country': 'es', 'year': 2010, 'month': 3, 'clicks': 421, 'users': 2},
    {'host': 1, 'country': 'es', 'year': 2010, 'month': 4, 'clicks': 22, 'users': 3},
    {'host': 2, 'country': 'es', 'year': 2010, 'month': 1, 'clicks': 111, 'users': 2},
    {'host': 2, 'country': 'es', 'year': 2010, 'month': 2, 'clicks': 2, 'users': 4},
    {'host': 3, 'country': 'es', 'year': 2010, 'month': 3, 'clicks': 34, 'users': 2},
    {'host': 3, 'country': 'es', 'year': 2010, 'month': 4, 'clicks': 1, 'users': 1}
])
```

```
clicks	country	host	month	users	year
0	123	fr	1	1	4	2010
1	134	fr	1	2	5	2010
2	341	fr	1	3	2	2010
3	113	es	1	1	4	2010
4	234	es	1	2	5	2010
5	421	es	1	3	2	2010
6	22	es	1	4	3	2010
7	111	es	2	1	2	2010
8	2	es	2	2	4	2010
9	34	es	3	3	2	2010
10	1	es	3	4	1	2010
```

Now we want to do a simple pivot operation. We want to pivot on host

```python
pd.pivot_table(data,
   index=['host'], 
   values=['users', 'clicks'], 
   columns=['year', 'month'],
   fill_value=''
  )
```

```
	clicks	users
year	2010	2010
month	1	2	3	4	1	2	3	4
host								
1	118	184	381	22	4	5	2	3
2	111	2			2	4		
3			34	1			2	1
```

We can add totals

```python
pd.pivot_table(data,
               index=['host'], 
               values=['users', 'clicks'], 
               columns=['year', 'month'],
               fill_value='',
               aggfunc=np.sum, 
               margins=True, 
               margins_name='Total'
              )
```

```
	clicks	users
year	2010	Total	2010	Total
month	1	2	3	4		1	2	3	4	
host										
1	236	368	762	22	1388	8	10	4	3	25
2	111	2			113	2	4			6
3			34	1	35			2	1	3
Total	347	370	796	23	1536	10	14	6	4	34
```

We can also pivot on more than one column. For example host and country

```python
pd.pivot_table(data,
               index=['host', 'country'], 
               values=['users', 'clicks'], 
               columns=['year', 'month'],
               fill_value=''
              )
```

```
	clicks	users
year	2010	2010
month	1	2	3	4	1	2	3	4
host	country								
1	es	113	234	421	22	4	5	2	3
fr	123	134	341		4	5	2	
2	es	111	2			2	4		
3	es			34	1			2	1

```

and also with totals

```python
pd.pivot_table(data,
               index=['host', 'country'], 
               values=['users', 'clicks'], 
               columns=['year', 'month'],
               aggfunc=np.sum, 
               fill_value='',
               margins=True, 
               margins_name='Total'
              )
```

```
		clicks	users
year	2010	Total	2010	Total
month	1	2	3	4		1	2	3	4	
host	country										
1	es	113	234	421	22	790	4	5	2	3	14
fr	123	134	341		598	4	5	2		11
2	es	111	2			113	2	4			6
3	es			34	1	35			2	1	3
Total		347	370	796	23	1536	10	14	6	4	34
```

We can group by dataframe and calculate subtotals
```python
data.groupby(['host', 'country'])[('clicks', 'users')].sum()
```

```
		clicks	users
host	country		
1	es	790	14
fr	598	11
2	es	113	6
3	es	35	3
```

```python
data.groupby(['host', 'country'])[('clicks', 'users')].mean()
```

```
clicks	users
host	country		
1	es	197.500000	3.500000
fr	199.333333	3.666667
2	es	56.500000	3.000000
3	es	17.500000	1.500000
```

And finally we can mix totals and subtotals.

```python
out = data.groupby('host').apply(lambda sub: sub.pivot_table(
    index=['host', 'country'], 
    values=['users', 'clicks'], 
    columns=['year', 'month'],
    aggfunc=np.sum, 
    margins=True,
    margins_name='SubTotal',
))

out.loc[('', 'Max', '')] = out.max()
out.loc[('', 'Min', '')] = out.min()
out.loc[('', 'Total', '')] = out.sum()

out.index = out.index.droplevel(0)

out.fillna('', inplace=True)
out
```

And that's all. A lot of to learn yet about data analysis, but Pandas will be definitely a good friend of mine.