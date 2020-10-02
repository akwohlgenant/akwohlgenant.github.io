---
title: "Shiny Web App: US Rig Counts by State"
date: 2020-09-03
tags: [shiny, plotly, oil, drilling]
header:
 image: "/images/worksite-ltd-kw9LLON5lO8-unsplash.jpg"
excerpt: "This is an interactive plot of rig counts in U.S. states built as a web app with Shiny in R."
mathjax: "true"
---

## U.S. Land Drilling Rig Counts by State and Date Range

First off, this app isn't going to look very good on a mobile device, especially a phone.  It's marginally better if you click the link below that takes you to the shinyapps.io site where this app is hosted.  It looks better there on any device; the embedded version here doesn't really have enough space, but it at least it gives you an inkling of what the app can do.

That said, this is an interactive plot of rig counts in U.S. states.  The data are provided by Baker Hughes as an indicator of oil and gas drilling activity.  Here is some more information on the data from the Baker Hughes website:

"*Baker Hughes has issued the rotary rig counts as a service to the petroleum industry since 1944, when Baker Hughes Tool Company began weekly counts of U.S. and Canadian drilling activity.  Baker Hughes initiated the monthly international rig count in 1975. The North American rig count is released weekly at noon Central Time on the last day of the work week. Starting in February 2020, the international rig count will be released on the last working day of the first week of the month.*

*The Baker Hughes Rig Counts are an important business barometer for the drilling industry and its suppliers. When drilling rigs are active they consume products and services produced by the oil service industry. The active rig count acts as a leading indicator of demand for products used in drilling, completing, producing and processing hydrocarbons.*"

 (<https://rigcount.bakerhughes.com/na-rig-count>)
 
To use the interactive plot, simply select a state or multiple states and a range of dates using the slider to see the line plots rendered in the plot output. Colorado, North Dakota, and Wyoming are selected by default; feel free to unselect them and choose the states you want to see in the plot.
 
<center>

<iframe src="https://andro-wohlgenant.shinyapps.io/US_RigCountsByState/" width="150%" height="500px" frameborder="0" allowfullscreen allow="geolocation"></iframe>


</center>

To view this web app in full-screen on the shinyapps.io website, click the link below:

 (<https://andro-wohlgenant.shinyapps.io/US_RigCountsByState/>)

I'm still learning shiny and I realize this is a pretty simple little app.  Feel free to pass along any tips or ideas to make this app better.  I plan to update the source code to download the data from Baker Hughes more frequently; I realize this dataset is a little dated now.  I also hope to modify this post to include the source code to generate the app; look for that update soon!


<span>Photo by <a href="https://unsplash.com/@worksite?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">WORKSITE Ltd.</a> on <a href="https://unsplash.com/s/photos/oil-rig?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
