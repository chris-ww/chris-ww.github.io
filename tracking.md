---
permalink: /projects/tracking/
---

Project Scope
----------------------

The goal of this project is to collect personal data and discover its value in learning about and improve quality of life.

- **Research:** 
    - Discover personal data that can be collected and Analysed
- **Data Engineering**: 
    - Extract Data from Api's and Csv's with python
    - Store Data in local database
    - Set up automatic update schedule with airflow,and backfill previous days data
- **Data Cleaning:**
    - Transform Data to consistent date formats
    - Map important features in the data
- **Data Visualisation**
    - Plot daily data as metrics with goals on bullet graphs
    - PLot historical data to discover any trends
- **Data Analysis**
    - Find correlations from data that has already been collected
- **Planning/Assessing:**
    - Discuss potential future data sources
    - Discuss potential future data analysis
    - Discuss strategies that could have the most practical impact
    - Discuss social/technological potential
    
Research - Data Sources
-------------------------------

I found tons of sources of personal data. I wanted to use data sources that fit a few criteria.

- **Relevant/Add Value**
    - Data provides information that could be useful. e.g. Sleep quality is known to be related to physical and mental health, emails in your spam folder are probably not.
- **Easy to record**
    - Data being collected automatically is alot more likely to be recorded and takes less time than manual recording.
- **Easy to extract**
    - Some apps have API's that make it easy to extract the data and can be done with an automatic process.
- **Accurate/consistent**
    - As the saying goes garbage in garbage out.
- **Free or cheap**
    - I don't want to have to buy extra equipment

**Sources of Data I used**

- **Excercies Tracker(fitbit)**
    - Activity (steps, calories burned, active minutes)
    - Sleep Data (length, restlessness, time got up)
- **Food Tracker(My Fitness Pal)**
    - Food eaten (calories, protein etc)
- **Weight/Bodyfat**
    - Smart scale
- **Computer Usage(Rescue time)**
    - minutes on computer/phone using productive/unproductive applications and websites
- **Location tracker(Google Location history)**
    - times at home/buisinesses/driving
- **Schedule(google calendar/DoItNow app)**
    - event/task times
    - task productivity/challenge level
- **Manual/Spreadsheets**
    - Can use a speadsheet to record any other information I would like. 
    - Success assessments/metrics, other important data that can't be recorded automatically
    - Data about jobs/applications/interviews
    - Subjective life satisfaction values
    
**Other Possible sources**
- Financial data, where my money is going
- Workout data, specific exercises/ repititions
- Emails and communication
- Weather data
- online shopping

Data Engineering
-------------------------------
   
**Steps**
- Extract and parse data from various sources with python
- Store in database
- Automate collection Schedule with airflow

I wrote a python package with functions that extract and parse data from the various sources "importer.py".
- **API's**
    - Fitbit and rescuetime have API's. 
    - Credentials have to be collected from your account online. 
    - an API requests is sent with credentials and information you want to request.
    - A response object is returned that can be parsed and turned into a dataframe.
-** Through other apps**
    - MyFitness pal has a private api that requires permission
    - Fitbit app has persmission,so I set the fitbit app to collect from MyFitnessPal and the smart scale
- **Local** - Some apps/services require you to download or save directly to your computer
    - Google location data - download json file form their website
    - DoItNow - database file that can be automatically saved on dropbox
    - Manual inputed Excel/CSV files - already easy to use


Data was stored in a local database file "personal_data.db" with SQLite

![image](img/sqlite.PNG)

I used airflow to automate the daily data collection with the "collect_fitbit.py" and "collect_rescue.py" files. Then I backfilled to collect from the days that have already past.

![image](img/airflow.PNG)

Data Visualisation
---------------------------

In R I made some visualisations for the data that has already been collect "Plot.R". Some of the data has just started to be collected now, so there are only a few points to look at.

I made bullet graph to show possible way to view daily "performance" goal metrics. The blue bar is the current value, the background colours show bad/ok/good values and the black vertical line is a pace line to reach the good region.

![image](img/metrics.png)

Here is a graph of sleeping/restless duration by day. 

![image](img/sleep.png)

Here is a graph of sleeping/restless duration by day. 

![image](img/active.png)

Here is a graph of slightly/fairly/very active minutes by day.
![image](img/weight.png)

Here is a graph of time on computer time on productive/unproductive applications by day. It only has a few days so far, so there is not much information.

![image](img/productivity plot.png)

There is a fair amount of variability in the graphs but there doesn't appear to be any long term trends except slightly lower weight.

Data analysis
----------------------------------------------


