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

First off, this isn't going to look very good or work very well on a mobile device.  It's marginally better on a mobile device if you click the link below that takes you to the shinyapps.io site where this app is hosted.  It looks better there on any device; the embedded version here doesn't really have enough space, but it at least it gives you an inkling of what the app can do.

That said, this is an interactive plot of rig counts in U.S. states built as a web app with Shiny in R. The data were downloaded from Baker Hughes.

 (<https://rigcount.bakerhughes.com/na-rig-count>)
 
Select a state or multiple states and a range of dates using the slider to see the line plots rendered. Colorado is selected by default; feel free to unselect it and choose the states you want to see in the plot.
 
<center>

<iframe src="https://andro-wohlgenant.shinyapps.io/US_RigCountsByState/" width="150%" height="500px" frameborder="0" allowfullscreen allow="geolocation"></iframe>


</center>

The code can be found in my github repository for this post, here:

 (<https://github.com/akwohlgenant/akwohlgenant.github.io/blob/master/_posts/>)

View full-screen on shinyapps.io website:

 (<https://andro-wohlgenant.shinyapps.io/US_RigCountsByState/>)


<span>Photo by <a href="https://unsplash.com/@worksite?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">WORKSITE Ltd.</a> on <a href="https://unsplash.com/s/photos/oil-rig?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>
