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

The code below is easy enough to modify, changing the 'names' list to include as many friends as one wishes, I have included 4 friends for demonstration purposes.


{% highlight python %}
hours_list = ['00:00','01:00','02:00','03:00','04:00','05:00',
             '06:00','07:00','08:00','09:00','10:00','11:00',
             '12:00','13:00','14:00','15:00','16:00','17:00',
             '18:00','19:00','20:00','21:00','22:00','23:00']


def hours_of_messages(name):
  search=str(name)
  name_len = len(search)

  indexes_of_name = [m.start() for m in re.finditer(search, fullstring)]
  
  find = re.compile('^(.*?\UTC)')
  counter = 0
  
  listo = []
  for i in range(0,len(indexes_of_name)-1):
    line =  fullstring[indexes_of_name[i]+name_len : indexes_of_name[i+1] ]
  
    try:
      x=  re.search(find, line).group(0)
        
      try:
        datetime_of_msg = (dt.datetime.strptime(x, '%A, %d %B %Y at %H:%M %Z'))
        listo.append(datetime_of_msg.hour)
        #writer.writerow([name,datetime_of_msg,counter])
      except:
        pass
      
    except:
      continue

  list2 = sorted(listo)

  k = [(x, list2.count(x)) for x in set(list2)]

  return k

names = ['Adam Butterfield','Joseph Noaman','Ben Jeffrey','Madhu Vanthi']

z = []

for name in names:

    new_row = []
    hours = hours_of_messages(name)
    print hours
    totalnumber_messages = sum(n for _, n in hours)
    b = np.zeros(24)
    for index, hour_msgs in enumerate(hours):
      b[hour_msgs[0]] = (float(hours[index][1]) / totalnumber_messages)*100

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



