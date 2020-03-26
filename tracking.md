---
permalink: /projects/tracking
---
Gathering and Analyzing Personal Data
======================================

I was interested to find out how much personal data was out there and how it could be potentially be used. The first step was to find out what interesting data I could collect. I wear a fitbit which records heart rate, activity level and sleep at pretty much all times. I also have a calendar and and android phone which tracks location data by default. These datasets seemed like a good place to start and could potentially be used to analyze my lifestyle.
For this project, I use a variety of API's and data source to collect, assemble, clean and report lifestyle data.

Here is a visual representation for the busy.
![image](img/tracker_project.png)


Fitbit.
-------------------------------

Firbit requires you to register on their site before you can use the web api. Once you do that, you are given an ID and password that you can use to login to the API.
Once I had the credentials you can access the api. I made a package with functions to access the data. 

First was to login to the client.

{% highlight python %}

from __future__ import print_function
import fitbit
import pandas as pd 
from fitbit import gather_keys_oauth2 as Oauth2
import datetime
import pickle
import os.path
from googleapiclient.discovery import build
from google_auth_oauthlib.flow import InstalledAppFlow
from google.auth.transport.requests import Request
import json
SCOPES = ['https://www.googleapis.com/auth/calendar.readonly']

def fitbit_client(id,secret):
    #Accepts id, secret credentials
    #uses id, secret to log into fitbit client
    #retrieves access token,refresh token
    #returns client

    server = Oauth2.OAuth2Server(id, secret)
    server.browser_authorize()
    accesstoken = str(server.fitbit.client.session.token['access_token'])
    refreshtoken = str(server.fitbit.client.session.token['refresh_token'])
    client = fitbit.Fitbit(id, secret, oauth2=True, access_token=accesstoken, refresh_token=refreshtoken)
    return client
  
{% endhighlight %}

Then I made functions to send requests to the api and parse the responses into dataframes. 
{% highlight python %}

    
def collect_heart(date,client):
    #Accepts date to access, fitbit client
    #returns time and heart rate from day requested in pandas dataframe

    time_list = []
    val_list = []
    heart = client.intraday_time_series('activities/heart', base_date=date, detail_level='1min')
    for i in heart['activities-heart-intraday']['dataset']:
        val_list.append(i['value'])
        time_list.append(i['time'])
    heartdf = pd.DataFrame({'Date':date,'Heart Rate':val_list,'Time':time_list})
    return heartdf
    

def collect_sleep(date,client):      
    #Accepts date to access, fitbit client
    #returns summary sleep data from day requested in pandas dataframe
     
    sleep = client.sleep(date=date)['sleep'][0]
    sleepdf = pd.DataFrame({'Date':sleep['dateOfSleep'],
               'MainSleep':sleep['isMainSleep'],
               'Efficiency':sleep['efficiency'],
               'Duration':sleep['duration'],
               'Minutes Asleep':sleep['minutesAsleep'],
               'Minutes Awake':sleep['minutesAwake'],
               'Awakenings':sleep['awakeCount'],
               'Restless Count':sleep['restlessCount'],
               'Restless Duration':sleep['restlessDuration'],
               'Time in Bed':sleep['timeInBed'],'start':sleep['startTime'],'end':sleep['endTime']
                        } ,index=[0])
    return sleepdf
    
{% endhighlight %}


Heartdf contains the date, time and heart rate for each minute

| Date|Heart rate|Time 
| 2020-03-20 | 59 | 00:00:00
| 2020-03-20 | 57 | 00:01:00|

sleepdf contains the date, start of sleep, end of sleep among other information

| Date | MainSleep | Efficiency | Duration | Minutes Asleep | Minutes Awake |Awakenings | Restless Count | Restless Duration | Time in Bed | start | end 
| 2020-03-20 | TRUE | 93 | 30720000 | 476 | 36 | 0 | 20 | 36 | 512 | 2020-03-19T23:55:00.000 | 2020-03-20T8:27:00.000|

Google Calendar
-------------------------------

Calendars allow you to schedule events, so calendar or other schedule software seem like a good place to record and retrieve events in the day. Google calendar also requires you to allow the api to access your data and to login the first time you access the api.

{% highlight python %}

def collect_calendar(date1,date2):
    #Accepts start date, end date
    #Uses code google https://developers.google.com/calendar/quickstart/python to login to api
    #uses and stores credentials as token on computer
    #returns list of calendar events in pandas dataframe

    creds = None
    if os.path.exists('token.pickle'):
        with open('token.pickle', 'rb') as token:
            creds = pickle.load(token)
    # If there are no (valid) credentials available, let the user log in.
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file(
                'credentials.json', SCOPES)
            creds = flow.run_local_server(port=0)
        # Save the credentials for the next run
        with open('token.pickle', 'wb') as token:
            pickle.dump(creds, token)

    service = build('calendar', 'v3', credentials=creds)
        # Call the Calendar API
    
    events_result = service.events().list(calendarId='primary', timeMin=date1,timeMax=date2,
                                        maxResults=100, singleEvents=True,
                                        orderBy='startTime').execute()
    events = events_result.get('items', [])
    
    #returns list of calendar events in pandas dataframe
    start_list = []
    end_list = []
    name_list=[]
    desc_list=[]
    for i in events:
        start_list.append(i['start']['dateTime'])
        end_list.append(i['end']['dateTime'])
        name_list.append(i['summary'])
        #desc_list.append(i['description'])
    eventdf=pd.DataFrame({'Date':date1,
                    'start':start_list,
                    'end':end_list,
                    'event':name_list})
    return eventdf

{% endhighlight %}

eventdf contains event name, start and end times

| Date | start | end | event 
|2020-03-20T04:57:31.046307Z |2020-03-20T10:00:00-04:00 | 2020-03-20T16:00:00-04:00 | work |

Google Location
-------------------------------
There wasn't an api to get the location data so I have to request a download from google takeout as a json document.

{% highlight python %}

def collect_location(file):
    #Accepts file name
    #Opens Json file and extracts locations,activities and times
    #returns pandas dataframe
  
    with open(file) as f:
        data = json.load(f)
        event_list = []
        start_list=[]
        end_list=[]
        for i in data['timelineObjects']:      
            if 'activitySegment' in i:
                event_list.append(i['activitySegment']['activityType'])
                start_list.append(i['activitySegment']['duration']['startTimestampMs'])
                end_list.append(i['activitySegment']['duration']['endTimestampMs'])
            if 'placeVisit' in i:     
                event_list.append(i['placeVisit']['location']['name'])
                start_list.append(i['placeVisit']['duration']['startTimestampMs'])
                end_list.append(i['placeVisit']['duration']['endTimestampMs'])
            
        locdf = pd.DataFrame({'start':start_list,'end':end_list,'location':event_list}) 
        return locdf        
    
{% endhighlight %}

locdf contained the activity/location, start time in milliseconds, end time in milliseconds.

| start | end | location
| 1583106934722 | 1583109205332 | IN_PASSENGER_VEHICLE
| 1583109205332 | 1583111388806 | Fit4Less |


Saving CSV files
-------------------------------
I made another python file to save collect and save the data as csv files 

{% highlight python %}

import pandas as pd 
import datetime
import sys
sys.path.append('sites/tracker')
from packages.importer import collect_sleep,collect_activity,fitbit_client,collect_heart,collect_calendar,collect_location

#Dates in multiple formats 
yesterday = str((datetime.datetime.now() - datetime.timedelta(days=1)).strftime("%Y%m%d"))
yesterday2 = str((datetime.datetime.now() - datetime.timedelta(days=1)).strftime("%Y-%m-%d"))
yesterday3 = datetime.datetime.today() - datetime.timedelta(hours=40)
yesterday3=yesterday3.isoformat()+'Z'
today = str(datetime.datetime.now().strftime("%Y%m%d"))
today2 = str((datetime.datetime.now() - datetime.timedelta(days=0)).strftime("%Y-%m-%d"))
today3 = datetime.datetime.today() - datetime.timedelta(hours=16)
today3=today3.isoformat()+'Z'

#using credentials saved on computer
codes=pd.read_csv("code.csv")
client=fitbit_client(codes.id[0],codes.secret[0])

#collecting data
heartdf=collect_heart(yesterday2,client)
sleepdf=collect_sleep(today2,client)
activitydf=collect_activity(yesterday2,client)
calendardf=collect_calendar(yesterday3,today3)
locdf=collect_location("sites/tracker/data/MAR2020LOC.json")

#saving data in csv
heartdf.to_csv('data/heart.csv',mode='a', header=True, index=False)
sleepdf.to_csv('data/sleep.csv', mode='a', header=False,index=False)
activitydf.to_csv('data/activity.csv', mode='a', header=True,index=False)
calendardf.to_csv('data/calendar.csv', mode='a', header=True,index=False)
locdf.to_csv('data/location.csv', mode='a', header=True,index=False)


{% endhighlight %}

Examining schedule
-------------------------------
I decided to use R to look at the data. I had to covert the dates/times because there were a variety of different formats.

{% highlight R %}

library(blogdown)
library(tidyr)
library(dplyr)
library(DiagrammeR)
library(plotly)

#collects data from csv files
sleepdf=read.csv('data/sleep.csv',header=TRUE)
heartdf=read.csv('data/heart.csv',header=TRUE)
eventdf=read.csv('data/calendar.csv',header=TRUE)
locdf=read.csv('data/location.csv',header=TRUE)
loctype=read.csv('data/location_types.csv',header=TRUE,fileEncoding="UTF-8-BOM")

#converts dates to consistent format
sleepdf$start=as.POSIXct(sleepdf$start, format = "%Y-%m-%dT%H:%M:%S")
sleepdf$end=as.POSIXct(sleepdf$end, format = "%Y-%m-%dT%H:%M:%S")
eventdf$start=as.POSIXct(eventdf$start, format = "%Y-%m-%dT%H:%M:%S")
eventdf$end=as.POSIXct(eventdf$end, format = "%Y-%m-%dT%H:%M:%S")
locdf$start=as.POSIXct(locdf$start/1000, origin="1970-01-01", tz='EST')
locdf$end=as.POSIXct(locdf$end/1000, origin="1970-01-01", tz='EST')

{% endhighlight %}

{% highlight R %}

#Labels locations by type
#example  loaction x-home
#         location y-work
#         labels based on data/location_types.csv
locdf$location2 <- loctype$location[match(locdf$location, loctype$value)]
locdf$event <- loctype$event[match(locdf$location, loctype$value)]


{% endhighlight %}

I decided to record exercise periods as extended times with heart rate over 100. I recorded exercise periods as events with start time and end time like the sleep/ location/ calendar data. I filled in up to 5 minute gaps in exercise incase heart rate drops below for a short period. 
{% highlight R %}

#records periods of high heart rate as exercise events
#fills in gaps if less then or eqaul to 5 minutes
#records as dataframe activitydf
workout_start =numeric(10)
workout_end = numeric(10)
heartdf$Heart.Rate=as.numeric(heartdf$Heart.Rate)
active=FALSE
count=0
workout_n=1
for(i in 1:(nrow(heartdf))){
  if(heartdf$Heart.Rate[i]>100){
    count=5
    if(active==FALSE){
      workout_start[workout_n]=i
      active=TRUE
    }
  }
  else{
    count=count-1
    if(active==TRUE){
      if(count<1){
        workout_end[workout_n]=i
        workout_n=workout_n+1
        active=FALSE
      }
    }
  }
}
activitydf<-data.frame(as.POSIXct(paste(heartdf$Date[workout_start],heartdf$Time[workout_start]), format="%Y-%m-%d %H:%M:%S"),
                       as.POSIXct(paste(heartdf$Date[workout_end],heartdf$Time[workout_end]), format="%Y-%m-%d %H:%M:%S"),
                       "workout")

names(activitydf)=c("start","end","group")


#labels events of activity as exercise and sleep as sleep
activitydf$event="exercise"
sleepdf$event="sleep"

{% endhighlight %}

I decided to look at the data as a Gantt chart. So I used the plotly package and drew bars for events and looked at 1 day as an example.

{% highlight R %}

#create figure
fig <- plot_ly()

tograph<-function(fig,data,colour,level){
  #accepts figure, dataset, colour and level
  #adds box for each event in data
  #start and end are when box starts/ends
  #colour can be set, or custom where it gives events different colour
  #level is height
  #returns figure
  
  for(i in 1:(nrow(data) )){
    fig = add_trace(fig,
      x = c(data$start[i], data$end[i]),  
      y = c(level, level),
      type="scatter",
      mode = "lines",
      line = list(color = colour, width = 100),
      showlegend = F, 
      text = paste("Event: ", data$event[i], "<br>") )
  }
  return(fig)
}

#creating dataframe for driving and loactions that aren't "home"
drivingdf=locdf[locdf$event=="driving",]
otherdf=locdf[!locdf$event=="driving"&!locdf$location2=="home",]

#adding data to the figure
fig=tograph(fig,sleepdf,"blue",0)
fig=tograph(fig,activitydf,"orange",1)
fig=tograph(fig,eventdf,"red",4)
fig=tograph(fig,drivingdf,"purple",2)
fig=tograph(fig,otherdf,"green",3)

#Looking at march 20, 2020
fig=fig %>% layout(title="Schedule March 20, 2020",margin=list(t=50),xaxis=list(range=c("2020-03-20","2020-03-21"),type="date"), 
                    yaxis = list(showgrid = F,tickmode = "array", tickvals = c(0,1,2,3,4), 
                                 ticktext = c("Sleep","exercise","drive","other","event"),tickfont=list(size=15)))
#print figure
fig


{% endhighlight %}


Example Day: ![image](img/schedule.png)

I will have to wait a little while start collecting the data, if I want to use it for analysis.

Future work/Discussion
-------------------------------
There are a variety of ways to add to this project

###  Scheduled events   ###
i could add schedule ETL process or automatic update to interesting metrics/ plots. I could have a program that displays the schedule and updates it everyday.

###  Other data   ###
There is alot more data that could be added to this dataset, including steps taken, food logs, device history and quality of day indicators.

###  Quality of day data   ###
If you record metrics to rate how good the day went, then you would be able to analyze and learn what improves your life.

###  Experiments   ###
If you are recording quality of day metrics, then you can also include experiments (like changing exercise/diet/lifestyle) to see if it improves your life.

###  Analysis   ###
When all of the data comes in it will be possible to do an analyis and learn about lifestyle data.



Thanks for reading,

Chris Werner-Westoby

References
-------------------------------

https://dev.fitbit.com/

https://dev.fitbit.com/build/reference/web-api/

https://python-fitbit.readthedocs.io/en/latest/

https://towardsdatascience.com/collect-your-own-fitbit-data-with-python-ff145fa10873

https://developers.google.com/calendar

https://takeout.google.com/settings/takeout

https://developers.google.com/calendar/quickstart/python

https://pandas.pydata.org/pandas-docs/stable/index.html

https://plot.ly/r/

https://www.python.org/

https://www.r-project.org/

Links
-------------------------------

[Link to code](https://github.com/chris-ww/personal-data)

[Back](/)
