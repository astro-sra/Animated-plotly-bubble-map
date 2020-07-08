# Animated-plotly-bubble-map
Animated plotly bubble map fro COVID-19 spread

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns


#Datasource - John Hokins COVID-19 data repository
#Github project - CSSEGISandData/COVID-19

#Cases confirmed
url_confirmed = 'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv'
df_confirmed = pd.read_csv(url_confirmed)


#Cases Deaths
url_death = 'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_deaths_global.csv'
df_death = pd.read_csv(url_death)


#Cases Recovered
url_recovered = 'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_recovered_global.csv'
df_recovered = pd.read_csv(url_recovered)

# As of 5 April'20
df_confirmed.shape   #(259, 78)
df_death.shape       #(259, 78)
df_recovered.shape   # (245, 78)

#dropping state column
df_confirmed.drop(columns= ['Province/State'])
df_death.drop(columns= ['Province/State'])
df_recovered.drop(columns= ['Province/State'])

#Group by (country-wise)
group_country = df_confirmed.groupby(['Country/Region'])
df_confirmed_grp = group_country.aggregate(np.sum) 

group_country = df_death.groupby(['Country/Region'])
df_death_grp = group_country.aggregate(np.sum) 

group_country = df_recovered.groupby(['Country/Region'])
df_recovered_grp = group_country.aggregate(np.sum) 



group_country = df_confirmed.groupby(['Country/Region'])
df_confirmed_geo = group_country.aggregate(np.mean) 
df_confirmed_grp['Lat'] = df_confirmed_geo['Lat']
df_confirmed_grp['Long'] = df_confirmed_geo['Long']


df_confirmed_grp['Long']['United Kingdom'] = -3.4360
df_confirmed_grp['Lat']['United Kingdom'] = 55.3781
df_confirmed_grp['Long']['Netherlands'] = 5.2913
df_confirmed_grp['Lat']['Netherlands'] = 52.1326
df_confirmed_grp['Long']['France'] = 2.2137
df_confirmed_grp['Lat']['France'] = 46.2276


import plotly.graph_objects as go
dataset = df_confirmed_grp

dates = df_confirmed_grp.iloc[:,2:-1].columns.tolist()
max_size = df_confirmed_grp.iloc[:,2:-1].max().max()

date = df_confirmed_grp.iloc[:,-2].name


df_death_grp['Lat'] = df_confirmed_grp['Lat']
df_death_grp['Long'] = df_confirmed_grp['Long']
df_recovered_grp['Lat'] = df_confirmed_grp['Lat']
df_recovered_grp['Long'] = df_confirmed_grp['Long']


# make data
data=[]

dataset_by_day = dataset[['Lat','Long']+[date]]
dataset_by_day_recovered = df_recovered_grp[['Lat','Long']+[date]]
dataset_by_day_death = df_death_grp[['Lat','Long']+[date]]
total_cases = dataset_by_day[date].sum()

data_dict = {
    "lon": list(dataset_by_day["Long"]),
    "lat": list(dataset_by_day["Lat"]),
    "mode": "markers",
    "text": list(dataset_by_day.index),
    "marker": {
        "sizemode": "area",
        "sizeref": 2*max_size/(120**2),
        "size": list(dataset_by_day[date]),
        "line":{"width":0}
    },
    "type":"scattergeo",
    "name":"Reported",
    "hoverinfo":"text+name"
    }
data_dict_recovered = {
    "lon": list(dataset_by_day_recovered["Long"]),
    "lat": list(dataset_by_day_recovered["Lat"]),
    "mode": "markers",
    "text": list(dataset_by_day_recovered.index),
    "marker": {
        "sizemode": "area",
        "sizeref": 2*max_size/(120**2),
        "size": list(dataset_by_day_recovered[date]),
        "color":"lightgreen",
        "line":{"width":0}
    },
    "type":"scattergeo",
    "name":"Recovered",
    "hoverinfo":"text+name"
    }
data_dict_death =  {
    "lon": list(dataset_by_day_death["Long"]),
    "lat": list(dataset_by_day_death["Lat"]),
    "mode": "markers",
    "text": list(dataset_by_day_death.index),
    "marker": {
        "sizemode": "area",
        "sizeref": 2*max_size/(120**2),
        "size": list(dataset_by_day_death[date]),
        "color":"#EF553B",
        "line":{"width":0}
    },
    "type":"scattergeo",
    "name":"Deaths",
    "hoverinfo":"text+name"
 
   }

cicrle1 = {
         "lon":[-130], 
         "lat":[0], 
         "showlegend":False,
         "text":"1000 K",
         "textposition":"middle right",
         "mode":"markers+text",
         "marker": {
         "sizemode": "area",
         "sizeref": 2*max_size/(120**2),
         "size": [1000000],
         "color" : "lightgray",
         "line":{"width":1,"color":"black"}
         },
         "hoverinfo":"none",
         "type":"scattergeo"
         }
cicrle2 = {
         "lon":[-150], 
         "lat":[-30], 
         "showlegend":False,
         "text":"<b>100 K<b>",
         "textposition":"bottom center",
         "mode":"markers+text",
         "marker": {
         "sizemode": "area",
         "sizeref": 2*max_size/(120**2),
         "size": [100000],
         "color" : "lightgray",
         "line":{"width":1,"color":"black"}
         },
         "hoverinfo":"none",
         "type":"scattergeo"
         }
cicrle3 = {
         "lon":[-150], 
         "lat":[-50], 
         "showlegend":False,
         "text":"<b>10 K<b>",
         "textposition":"bottom center",
         "mode":"markers+text",
         "marker": {
         "sizemode": "area",
         "sizeref": 2*max_size/(120**2),
         "size": [10000],
         "color" : "lightgray",
         "line":{"width":1,"color":"black"}
         },
         "hoverinfo":"none",
         "type":"scattergeo"
         }

data= [data_dict, data_dict_recovered, data_dict_death,cicrle1,cicrle2,cicrle3]


layout = {
    "title":{"text": "<b>Spread of COVID-19<b>","x":0.5,"y":1},
    "hovermode": "closest",
    "geo": {
        "scope":"world"
        },
    "margin":{"l":0, "r":0, "t":20, "b":0,"pad":0},
    "updatemenus" : [
    {
        "buttons": [
            {
                "args": [None, {"frame": {"duration": 50, "redraw": True},
                                "fromcurrent": True, "transition": {"duration": 50,
                                                                    "easing": "quadratic-in-out"}}],
                "label": "Play",
                "method": "animate"
            },
            {
                "args": [[None], {"frame": {"duration": 0, "redraw": False},
                                  "mode": "immediate",
                                  "transition": {"duration": 0}}],
                "label": "Pause",
                "method": "animate"
            }
        ],
        "direction": "left",
        "pad": {"r": 00, "t": 10},
        "showactive": True,
        "type": "buttons",
        "x": 0.1,
        "xanchor": "right",
        "y": 0,
        "yanchor": "top",
        "active":2
    }
],
    "sliders" : [{
    "active": 0,
    "yanchor": "top",
    "xanchor": "left",
    "currentvalue": {
        "font": {"size": 20},
        "prefix": "Date:",
        "visible": True,
        "xanchor": "right"
    },
    "transition": {"duration": 50, "easing": "cubic-in-out"},
    "pad": {"b": 10, "t": 10,"r":0,"l":0},
    "len": 0.9,
    "x": 0.1,
    "y": 0,
    "steps": []
}],
    "annotations":[ {
        "text":"<b>World-wide Reported Cases:<br>"+str('{:,}'.format(total_cases))+"<b>",
        "align":"left",
        "visible":True,
        "showarrow":False,
        "font":{"color":'#1f77b4'},
        "xref":"paper",
        "yref":"paper",
        "x":1.07,
        "y":0.5,
        "bordercolor":None,
        "borderwidth":0}]
    
    }

# make frames
frames =[]
for date in dates:
    frame = {"data": [], "name": str(date),"layout":{"annotations":[]}}
    dataset_by_day = dataset[['Lat','Long']+[date]]
    dataset_by_day_recovered = df_recovered_grp[['Lat','Long']+[date]]
    dataset_by_day_death = df_death_grp[['Lat','Long']+[date]]
    total_cases = dataset_by_day[date].sum()
    total_recoveries = dataset_by_day_recovered[date].sum()
    total_deaths = dataset_by_day_death[date].sum()
    total_countries = len(dataset_by_day[dataset_by_day[date]>0])

    data_dict = {
        "lon": list(dataset_by_day["Long"]),
        "lat": list(dataset_by_day["Lat"]),
        "mode": "markers",
        "text": list(zip(dataset_by_day.index,dataset_by_day[date])),
        "marker": {
            "sizemode": "area",
            "sizeref": 2*max_size/(120**2),
            "size": list(dataset_by_day[date]),
            "line":{"width":0}
        },
        "type":"scattergeo",
        "name":"Reported",
        "hoverinfo":"text+name"
    }
    data_dict_recovered = {
        "lon": list(dataset_by_day_recovered["Long"]),
        "lat": list(dataset_by_day_recovered["Lat"]),
        "mode": "markers",
        "text": list(zip(dataset_by_day_recovered.index,dataset_by_day_recovered[date])),
        "marker": {
            "sizemode": "area",
            "sizeref": 2*max_size/(120**2),
            "size": list(dataset_by_day_recovered[date]),
            "color" : "lightgreen",
            "line":{"width":0}
        },
        "type":"scattergeo",
        "name":"Recovered",
        "hoverinfo":"text+name"
    }
    data_dict_death ={
        "lon": list(dataset_by_day_death["Long"]),
        "lat": list(dataset_by_day_death["Lat"]),
        "mode": "markers",
        "text": list(zip(dataset_by_day_death.index,dataset_by_day_death[date])),
        "marker": {
            "sizemode": "area",
            "sizeref": 2*max_size/(120**2),
            "size": list(dataset_by_day_death[date]),
            "color" : "#EF553B",
            "line":{"width":0}
        },
        "type":"scattergeo",
        "name":"Deaths",
        "hoverinfo":"text+name"
    }
    cicrle1 = {
             "lon":[-150], 
             "lat":[0], 
             "showlegend":False,
             "text":"<b>1000 K<b>",
             "textposition":"bottom center",
             "mode":"markers+text",
             "marker": {
             "sizemode": "area",
             "sizeref": 2*max_size/(120**2),
             "size": [1000000],
             "color" : "lightgray",
             "line":{"width":1,"color":"black"}
             },
             "hoverinfo":"none",
             "type":"scattergeo"
             }
    cicrle2 = {
             "lon":[-150], 
             "lat":[-30], 
             "showlegend":False,
             "text":"<b>100 K<b>",
             "textposition":"bottom center",
             "mode":"markers+text",
             "marker": {
             "sizemode": "area",
             "sizeref": 2*max_size/(120**2),
             "size": [100000],
             "color" : "lightgray",
             "line":{"width":1,"color":"black"}
             },
             "hoverinfo":"none",
             "type":"scattergeo"
             }
    cicrle3 = {
             "lon":[-150], 
             "lat":[-50], 
             "showlegend":False,
             "text":"<b>10 K<b>",
             "textposition":"bottom center",
             "mode":"markers+text",
             "marker": {
             "sizemode": "area",
             "sizeref": 2*max_size/(120**2),
             "size": [10000],
             "color" : "lightgray",
             "line":{"width":1,"color":"black"}
             },
             "hoverinfo":"none",
             "type":"scattergeo"
             }
    
    frame["data"] = [data_dict, data_dict_recovered, data_dict_death,cicrle1,cicrle2,cicrle3]
    
    annote1 = {
        "text":"<b>World-wide Reported Cases:<br>"+str('{:,}'.format(total_cases))+"<b>",
        "align":"left",
        "visible":True,
        "showarrow":False,
        "font":{"color":'#1f77b4'},
        "xref":"paper",
        "yref":"paper",
        "x":1.07,
        "y":0.5,
        "bordercolor":None,
        "borderwidth":0}
    
    annote2 = {
        "text":"<b>World-wide Recovered Cases:<br>"+str('{:,}'.format(total_recoveries))+"<b>",
        "align":"left",
        "visible":True,
        "showarrow":False,
        "font":{"color":'lightgreen'},
        "xref":"paper",
        "yref":"paper",
        "x":1.08,
        "y":0.4,
        "bordercolor":None,
        "borderwidth":0}
    
    annote3 = {
        "text":"<b>World-wide Deceased Cases:<br>"+str('{:,}'.format(total_deaths))+"<b>",
        "align":"left",
        "visible":True,
        "showarrow":False,
        "font":{"color":'#EF553B'},
        "xref":"paper",
        "yref":"paper",
        "x":1.075,
        "y":0.27,
        "bordercolor":None,
        "borderwidth":0}
    
    annote4 = {
        "text":"<b>Affected Countries:<br>"+str('{:,}'.format(total_countries))+"<b>",
        "align":"left",
        "visible":True,
        "showarrow":False,
        "font":{"color":'black'},
        "xref":"paper",
        "yref":"paper",
        "x":1.02,
        "y":0.6,
        "bordercolor":None,
        "borderwidth":0}
    
    frame["layout"]["annotations"] = [annote1,annote2,annote3,annote4]
    
    
    frames.append(frame)
  
    slider_step = {"args": [
        [date],
        {"frame": {"duration": 50, "redraw": True},
          "mode": "immediate",
          "transition": {"duration": 50}},
    ],    
        "label": date,
        "method": "animate"}
    layout["sliders"][0]["steps"].append(slider_step)
    

fig = go.Figure(data =data,layout = layout, frames =frames)

plot(fig, auto_open=True)

fig.write_html("animated_bubble_map.html")

