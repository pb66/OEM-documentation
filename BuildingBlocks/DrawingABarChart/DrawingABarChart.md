## Drawing a myelectric style bar chart

Both the myelectric emoncms module and the soon to be myelectric android app have a bar chart that is written using the javascript 2d graphics canvas on emoncms and the java 2d graphics canvas on the android app.

This short guide details how the bar chart is built. Writting your own graphs is really not that complicated and once you have a grasp of 2d graphics canvas sometimes its easier to write your own graph to get the style that you want rather than figuring out how to use and then hack a library.

The myelectric graph was written from scratch in order to achieve a precice look: simple color scheme, bar height values overlayed on the bar, kWh label in the top-left corner.

![Bar chart](files/bargraphic.png)

The essential part of creating a graph is mapping the data time:value coordinates on to the 2d graphics canvas x:y pixel coordinates. 

Lets start by taking an example dataset for 7 days of electricity consumption in kWh/d:

  time, value
  1401235200, 4.8
  1401321600, 5.8
  1401408000, 5.0
  1401494400, 7.6
  1401580800, 4.4
  1401667200, 2.3
  1401753600, 5.0





