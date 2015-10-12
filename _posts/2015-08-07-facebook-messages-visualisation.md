---
layout:     post
title:      Facebook Message Analysis.
date:       2015-08-7 21:01:19
summary:    Facebook message analysis.
categories: data analysis
thumbnail: cogs
tags:

---

Facebook allows users to download their full data. Within this data is a full html page of every message sent by you and to you. Out of interest, I wanted to chart when certain people message me and how many messages I've recieved from them

Using python and matplotlib, this is fairly simple to do.


First, we strip all html tags. My messages file totals over 100,000 messages and is quite a large file (for text), our first task is to strip html tags.

{% highlight python %}
from HTMLParser import HTMLParser
from sys import argv
import re
import datetime as dt
import csv
import numpy as np
from matplotlib import pyplot as plt

class MLStripper(HTMLParser):
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

txt = open('messages.htm','r') #reading the messages file
text = txt.read()

fullstring = strip_tags(str(text))
{% endhighlight %}

Next, we create a function which, when fed with a friends name outputs a date sorted list of every message.
Using a simple regex, we can filter out references to names within messages and only look at those with a timestamp immediately after them as these are the messages sent by them to me.

Facebook message times are of the form:

    '18 January 2014 at 20:47 UTC'

We convert this to a datetime object for our visualisation.

Then, we write the datetime of every message sent, along with a counter to a CSV file

{% highlight python %}

def dates_times_of_msgs(name):
  search=str(name) # Search for all mentions of name
  name_len = len(search)

  indexes_of_name = [m.start() for m in re.finditer(search, fullstring)] # indexed of name
  
  find = re.compile('^(.*?\UTC)') # 
  counter = 0
  
  listo = []
  for i in range(0,len(indexes_of_name)-1):
    line =  fullstring[indexes_of_name[i]+name_len : indexes_of_name[i+1] ]
  
    try:
      x=  re.search(find, line).group(0)
        
      try:
        datetime_of_msg = (dt.datetime.strptime(x, '%A, %d %B %Y at %H:%M %Z')) # Converts to datetime object
        listo.append(datetime_of_msg)
        writer.writerow([name,datetime_of_msg,counter])
      except:
        pass
      
    except:
      continue
  return sorted(listo)
{% endhighlight %}


Now, define a simple plotter function to draw our names on a line graph:

{% highlight python %}
def plotter(name,col):
  a = dates_times_of_msgs(name)
  days=[]
  c=[]

  for i in range(len(a)):
    days.append(a[i])
    c.append(i+1)

  plt.plot(days,c,lw=2,label=name,c=col)
{% endhighlight %}

Now we can call the plotter function with name and desired colours as many times as we like!

{% highlight python %}
plotter('person1','orange')
plotter('person2','blue')
plotter('person3','red')
plotter('person3','green')
{% endhighlight %}


Note: I have modified names here for anonymity reasons.

Adding some styling options for the graph and displaying it below:


{% highlight python %}

plt.title('Cumulative Number of Facebook Messages Recieved From Person Since Collection Began',fontsize=20)
plt.xlabel('Date',fontsize=18)
plt.ylabel('Number of Messages Recieved',fontsize=18)
plt.legend(bbox_to_anchor=(1.0, 1), loc=2, borderaxespad=0.)
plt.gca().xaxis.grid(True)
plt.gca().yaxis.grid(True)

plt.show()

{% endhighlight %}

<img src="http://i.imgur.com/yam6c47.png" width="200" height="200" />


> [You can view a high res image of my results with a few selected friends here][1]


Note: Red deleted her account for a while around May 2014 explaining the gap.

Stay tuned for more message analysis coming soon, just whacked this out today, got a lot more work to do on it!


[1]: http://i.imgur.com/yam6c47.png

