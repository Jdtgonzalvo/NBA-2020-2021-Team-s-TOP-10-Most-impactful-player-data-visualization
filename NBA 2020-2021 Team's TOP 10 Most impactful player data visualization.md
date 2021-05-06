```python
import pandas as pd
import requests
from bs4 import BeautifulSoup
import matplotlib.pyplot as plt
import matplotlib as mpl
from urllib.parse import urljoin
import re
from pywaffle import Waffle
from highlight_text import fig_text
```

## 1) Extracting the links via BeautifulSoup


```python
url =  'https://tv5.espn.com/nba/teams'
r = requests.get(url)
html_data = r.text
```


```python
soup = BeautifulSoup(html_data,'html5lib')
```


```python
base =  'https://tv5.espn.com'
url_list = []
name_list = []
teams_dict = {i:None for i in range(30)}

i = 1
for a in soup.find_all('a', href=re.compile("team/stats")):
    print ("Found the URL: #{}".format(i), a['href'])
    temp = a['href'].replace('/nba/team/stats/_/name/','')
    temp_split = temp.split('/')
    name_list.append(temp_split[1])
    url = urljoin(base, a['href'])
    url_list.append(url)
    i+=1
```

    Found the URL: #1 /nba/team/stats/_/name/bos/boston-celtics
    Found the URL: #2 /nba/team/stats/_/name/bkn/brooklyn-nets
    Found the URL: #3 /nba/team/stats/_/name/ny/new-york-knicks
    Found the URL: #4 /nba/team/stats/_/name/phi/philadelphia-76ers
    Found the URL: #5 /nba/team/stats/_/name/tor/toronto-raptors
    Found the URL: #6 /nba/team/stats/_/name/chi/chicago-bulls
    Found the URL: #7 /nba/team/stats/_/name/cle/cleveland-cavaliers
    Found the URL: #8 /nba/team/stats/_/name/det/detroit-pistons
    Found the URL: #9 /nba/team/stats/_/name/ind/indiana-pacers
    Found the URL: #10 /nba/team/stats/_/name/mil/milwaukee-bucks
    Found the URL: #11 /nba/team/stats/_/name/den/denver-nuggets
    Found the URL: #12 /nba/team/stats/_/name/min/minnesota-timberwolves
    Found the URL: #13 /nba/team/stats/_/name/okc/oklahoma-city-thunder
    Found the URL: #14 /nba/team/stats/_/name/por/portland-trail-blazers
    Found the URL: #15 /nba/team/stats/_/name/utah/utah-jazz
    Found the URL: #16 /nba/team/stats/_/name/gs/golden-state-warriors
    Found the URL: #17 /nba/team/stats/_/name/lac/la-clippers
    Found the URL: #18 /nba/team/stats/_/name/lal/los-angeles-lakers
    Found the URL: #19 /nba/team/stats/_/name/phx/phoenix-suns
    Found the URL: #20 /nba/team/stats/_/name/sac/sacramento-kings
    Found the URL: #21 /nba/team/stats/_/name/atl/atlanta-hawks
    Found the URL: #22 /nba/team/stats/_/name/cha/charlotte-hornets
    Found the URL: #23 /nba/team/stats/_/name/mia/miami-heat
    Found the URL: #24 /nba/team/stats/_/name/orl/orlando-magic
    Found the URL: #25 /nba/team/stats/_/name/wsh/washington-wizards
    Found the URL: #26 /nba/team/stats/_/name/dal/dallas-mavericks
    Found the URL: #27 /nba/team/stats/_/name/hou/houston-rockets
    Found the URL: #28 /nba/team/stats/_/name/mem/memphis-grizzlies
    Found the URL: #29 /nba/team/stats/_/name/no/new-orleans-pelicans
    Found the URL: #30 /nba/team/stats/_/name/sa/san-antonio-spurs
    

## 2) Automate the ETL process using a function.


```python
def create_waffle(url_number,minimum_games=24):
    url =  url_list[url_number]
    r = requests.get(url)
    html_data = r.text

    new_soup = BeautifulSoup(html_data,'html5lib')
    tables= new_soup.find_all('table')

    df1, = pd.read_html(str(tables[0]), flavor = 'bs4')
    df2, = pd.read_html(str(tables[1]), flavor = 'bs4')
    new_df = df1.join(df2)
    teams_dict[url_number] = new_df
    
    new_df = new_df[new_df.GP >=24]
    new_df= new_df.sort_values(by=['MIN','PER'],ascending=False).head(10)
    main_stat = new_df[['Name','PTS','REB','AST']].sort_values(by=['PTS'],ascending=False)
    
    main_stat = main_stat.T
    main_stat.columns = main_stat.iloc[0]
    main_stat.drop('Name',inplace=True)
    my_list = main_stat.columns.values.tolist()
    
    main_stat = pd.DataFrame(main_stat,columns=my_list)
    fig = plt.figure(
     FigureClass=Waffle,
    columns =7,
     figsize = (20,10),
     plots= {
         (2,5,1):{
             'values':main_stat.iloc[:,0],
             'title':{
                 'label':my_list[0],
                 'fontsize':20,
                 'color':'lightgrey',
                 'loc':'left'
             }
         },
         (2,5,2):{
             'values':main_stat.iloc[:,1],
             'title':{
                 'label':my_list[1],
                 'fontsize':20,
                 'color':'lightgrey',
                 'loc':'left'
             }
         },
         (2,5,3):{
             'values':main_stat.iloc[:,2],
             'title':{
                 'label':my_list[2],
                 'color':'lightgrey',
                 'fontsize':20,
                 'loc':'left'
             }
         },
         (2,5,4):{
             'values':main_stat.iloc[:,3],
             'title':{
                 'label':my_list[3],
                 'fontsize':20,
                 'color':'lightgrey',
                 'loc':'left'
             }
         },
         (2,5,5):{
             'values':main_stat.iloc[:,4],
             'title':{
                 'label':my_list[4],
                 'fontsize':20,
                 'color':'lightgrey',
                 'loc':'left'
             }
         },
         (2,5,6):{
             'values':main_stat.iloc[:,5],
             'title':{
                 'label':my_list[5],
                 'fontsize':20,
                 'color':'lightgrey',
                 'loc':'left'
             }
         },
         (2,5,7):{
             'values':main_stat.iloc[:,6],
             'title':{
                 'label':my_list[6],
                 'fontsize':20,
                 'color':'lightgrey',
                 'loc':'left'
             }
         },
         (2,5,8):{
             'values':main_stat.iloc[:,7],
             'title':{
                 'label':my_list[7],
                 'color':'lightgrey',
                 'fontsize':20,
                 'loc':'left'
             }
         },
         (2,5,9):{
             'values':main_stat.iloc[:,8],
             'title':{
                 'label':my_list[8],
                 'fontsize':20,
                 'color':'lightgrey',
                 'loc':'left'
             }
         },
         (2,5,10):{
             'values':main_stat.iloc[:,9],
             'title':{
                 'label':my_list[9],
                 'fontsize':20,
                 'color':'lightgrey',
                 'loc':'left'
             }
         }
     },
    colors = ('#ff7c43','#de425b','#ffa600')
        )

    fig_text(
    s=f'{name_list[url_number].upper()} \n <POINTS> <REBOUNDS> <ASSIST>',
    x= 0.01,
    y=0.99,
    fontsize=25,
    color ='lightgrey',
    highlight_textprops=[{"color": '#ff7c43'},{"color": '#de425b'},
                              {"color": '#ffa600'}]
    )
    fig.patch.set_facecolor('#101820FF')
```

## 3) Create Visualization for the 30 NBA teams

The top 10 impactful players are selected by their player efficiency rating and the number of games playes. 
So players with high efficiency rating but haven't played atleast one-third of the season's games would not be selected. 
Each of the 10 players's main stats(Points, Rebounds and Assist) are visualized via a waffle chart. 


```python
#removes matplotlib memory restriction warning

mpl.rc('figure', max_open_warning = 0)
```


```python
for i in range(30):
    create_waffle(i)
```


    
![png](output_9_0.png)
    



    
![png](output_9_1.png)
    



    
![png](output_9_2.png)
    



    
![png](output_9_3.png)
    



    
![png](output_9_4.png)
    



    
![png](output_9_5.png)
    



    
![png](output_9_6.png)
    



    
![png](output_9_7.png)
    



    
![png](output_9_8.png)
    



    
![png](output_9_9.png)
    



    
![png](output_9_10.png)
    



    
![png](output_9_11.png)
    



    
![png](output_9_12.png)
    



    
![png](output_9_13.png)
    



    
![png](output_9_14.png)
    



    
![png](output_9_15.png)
    



    
![png](output_9_16.png)
    



    
![png](output_9_17.png)
    



    
![png](output_9_18.png)
    



    
![png](output_9_19.png)
    



    
![png](output_9_20.png)
    



    
![png](output_9_21.png)
    



    
![png](output_9_22.png)
    



    
![png](output_9_23.png)
    



    
![png](output_9_24.png)
    



    
![png](output_9_25.png)
    



    
![png](output_9_26.png)
    



    
![png](output_9_27.png)
    



    
![png](output_9_28.png)
    



    
![png](output_9_29.png)
    



```python

```
