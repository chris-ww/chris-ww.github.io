---
---

Most people have alot of data personal data on the internet that could be collected and analyzed. 

Examples
--------------------

Phones take location data, we keep calendars/schedules/day planners/food and excercise loggers, we use fitness trackers and every site we visit is recorded in device histories. Many site allow for this data to be downloaded and many have api's(application program interface) that allow programs to access the data. They usually involve your program sending a request with parameters indicating what you want and their servers sending back a response. With the api's you can make a program to automatically download and automatically perform procedures like visualizations. Apps/sites will have documentation that shows how to access and use their api. There are a variety of ways that access api's but I used python. There are packages to simplify the process for many popular api services.

One place i collected data from was from fitbit. Data can be downloaded on the web from your account or you can use their api. You first register an app on their website and then you are given a id and secret to use to access your data. Then i used the fitbit package in python to create a script to download the data. The script downloads sleep quality data as well as activity data for the day, but there is tons more data that you can download that is documented on their website including the ability to get heart rate from every second of the day.


