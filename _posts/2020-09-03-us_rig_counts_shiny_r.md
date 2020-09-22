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

First off, this isn't going to look very good or work very well on a mobile device.  It's marginally better if you click the link below that takes you to the shinyapps.io site where this app is hosted.  It looks better there on any device; the embedded version here doesn't really have enough space, but it at least it gives you an inkling of what the app can do.

That said, this is an interactive plot of rig counts in U.S. states.  The data are provided by Baker Hughes as an indicator of oil and gas drilling activity.  Here is some more information on the data from the Baker Hughes website:

"Baker Hughes has issued the rotary rig counts as a service to the petroleum industry since 1944, when Baker Hughes Tool Company began weekly counts of U.S. and Canadian drilling activity.  Baker Hughes initiated the monthly international rig count in 1975. The North American rig count is released weekly at noon Central Time on the last day of the work week. Starting in February 2020, the international rig count will be released on the last working day of the first week of the month.

The Baker Hughes Rig Counts are an important business barometer for the drilling industry and its suppliers. When drilling rigs are active they consume products and services produced by the oil service industry. The active rig count acts as a leading indicator of demand for products used in drilling, completing, producing and processing hydrocarbons. built as a web app with Shiny in R."

 (<https://rigcount.bakerhughes.com/na-rig-count>)
 
To use the interactive plot, simply select a state or multiple states and a range of dates using the slider to see the line plots rendered in the plot output. Colorado is selected by default; feel free to unselect it and choose the states you want to see in the plot.
 
<center>

<iframe src="https://andro-wohlgenant.shinyapps.io/US_RigCountsByState/" width="150%" height="500px" frameborder="0" allowfullscreen allow="geolocation"></iframe>


</center>

To view this web app in full-screen on the shinyapps.io website, click the link below:

 (<https://andro-wohlgenant.shinyapps.io/US_RigCountsByState/>)


<span>Photo by <a href="https://unsplash.com/@worksite?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">WORKSITE Ltd.</a> on <a href="https://unsplash.com/s/photos/oil-rig?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
