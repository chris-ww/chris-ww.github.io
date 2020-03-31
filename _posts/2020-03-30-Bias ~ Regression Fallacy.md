---
---
Human Beings are good at picking up patterns even if they don't exist. One example of this is the regression fallacy, where extreme events are often followed by moderate events (regression to the mean).

Random distributions
---------------------------------

Many processes have variability which means that they will be some randomness that cannot be predicted. One example is a stocks earning, there may be some predictible behaviour in stocks but a lot of the fluctuation is almost random.
Example: Lets say I interested in improving my investments. I have gererated random data to represent the change in value of my portfolio over 20 days.
![alt]({{ site.url }}{{ site.baseurl }}/img/earnings.png)
My earnings fluctuate through the days because of inherent variability.


Example
---------------------------------

Lets say I want to take action to correct the stocks on poor performing days. On days that my portfolio loses more than $100, I will throw a quarter in the well and wish for better earnings tommorow. Here are the same 20 days with my intervention.
![alt]({{ site.url }}{{ site.baseurl }}/img/intervention.png)
It seemed like throwing quarters in the well improved my stock performance. After every intervention, my stocks did better than they did before.

Regression to the Mean
---------------------------------

In reality, me losing over $100 was an unlikely event caused by the inherent variability. Most days my stocks are expected to do better than losing $100 and will tend to be near to the average value. 

The apparent correlation with the intervention is reversed. the large losses are causing me to perform the intervention. This fallacy can show up in a variety of situations like taking a pain remedy when you feel the worst and giving punishments when somebody performs the worst.\

This can make it difficult to determine if an intervention is actually working.
This phenomenon, and others that misattribute causality are common and show why we should be carefull with analysis.

Learn more:
-----------------------
Code Used for graphs: https://github.com/chris-ww/Posts/blob/master/2020-03-30.R

Book: Thinking Fast and Slow by Daniel Kahneman
Wiki: https://en.wikipedia.org/wiki/Regression_fallacy
Software: https://www.r-project.org/

