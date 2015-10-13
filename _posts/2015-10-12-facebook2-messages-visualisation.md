---
layout:     post
title:      Heatmap of Facebook message Distribution with Plotly.
date:       2015-10-12 21:26:00
summary:    Facebook message distribution Heatmap.
categories: data analysis
thumbnail: cogs
tags:

---

Expanding on my earlier facebook visualisation post, here is a python script, which uses Plotly to graph the distribution of messages recieved by various friends. 

The only dependencies are numpy and plotly.

The code below is easy enough to modify, changing the 'names' list to include as many friends as one wishes, I have included 4 friends for demonstration purposes, with over 40,000 messages over 4 years.


{% highlight python %}

import numpy as np
import plotly.plotly as py
import plotly.graph_objs as go
from HTMLParser import HTMLParser
import re
import datetime as dt
import numpy as np


class MLStripper(HTMLParser):
  #Strips HTML tags
    def __init__(self):
        self.reset()
        self.fed = []
    def handle_data(self, d):
        self.fed.append(d)
    def get_data(self):
        return ''.join(self.fed)

def strip_tags(html):
    s = MLStripper()
    s.feed(html)
    return s.get_data()

txt = open('messages.htm','r')
#messages.htm directory in same file as script
text = txt.read()


fullstring = strip_tags(str(text))



py.sign_in('USERNAME', 'API KEY')
# input your plotly username and API-Key here



hours_list = ['00:00','01:00','02:00','03:00','04:00','05:00',
             '06:00','07:00','08:00','09:00','10:00','11:00',
             '12:00','13:00','14:00','15:00','16:00','17:00',
             '18:00','19:00','20:00','21:00','22:00','23:00']


def hours_of_messages(name):
  search=str(name)
  name_len = len(search)

  indexes_of_name = [m.start() for m in re.finditer(search, fullstring)]
  #searches name
  find = re.compile('^(.*?\UTC)')
  #regex finds datestamps (which filters out mentions of name without datestamp)
  
  list_times = []
  for i in range(0,len(indexes_of_name)-1):
    line =  fullstring[indexes_of_name[i]+name_len : indexes_of_name[i+1] ]
  
    try:
      x=  re.search(find, line).group(0)
        
      try:
        datetime_of_msg = (dt.datetime.strptime(x, '%A, %d %B %Y at %H:%M %Z'))
        list_times.append(datetime_of_msg.hour)
      except:
        pass
      
    except:
      continue

  list_times = sorted(list_times)
  #sorts times chronologically

  k = [(x, list_times.count(x)) for x in set(list_times)]
  # returns list of tuples of each hour and messages recieved in that hour

  return k

#names = ['Adam Butterfield','Joseph Noaman','Ben Jeffrey','Madhu Vanthi']

def heatmap_gen(names):
  # input names as list eg: names = ['John Doe','Bob Jones',...]

  z = []

  for name in names:

    new_row = []
    hours = hours_of_messages(name)
    print hours
    totalnumber_messages = sum(n for _, n in hours)
    b = np.zeros(24)
    for index, hour_msgs in enumerate(hours):
      b[hour_msgs[0]] = (float(hours[index][1]) / totalnumber_messages)*100
      #returns percent of total messages recieved per hour

    z.append(b)

  data = go.Data([
      go.Heatmap(
        z=z,
        x=hours_list,
        y=names,
        colorscale='Viridis',
      )
    ])


  layout = go.Layout( title='Percentage of Total Facebook Messages Recieved by Hour',
                  xaxis = dict( ticks='', nticks=24 ),
                  yaxis = dict( ticks='' ) )

  fig = go.Figure( data=data, layout=layout )

  url = py.plot(fig, filename='facebook-hourly', validate=False)   

{% endhighlight %}


<iframe width=950 height=500 frameborder="0" seamless="seamless" scrolling="no" src="https://plot.ly/~varunbalupuri/24/percentage-of-total-facebook-messages-recieved-per-hour/"> </iframe> 



