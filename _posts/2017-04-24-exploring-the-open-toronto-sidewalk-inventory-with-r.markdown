---
layout: post
title:  "Exploring the Open Toronto Sidewalk Inventory with R"
description: "Open data lets you link together different sets in interesting ways! This article shows how to map Toronto sidewalk data by neighbourhood, using R, City of Toronto data, and Google Maps"
---

There's growing interest in opening up civic datasets, to let local communities do interesting and new analyses. This article shows how to map [Toronto sidewalks][cotsi] by neighbourhood, using [City of Toronto open data][cot], Google Maps, and R.

[cotsi]:    http://www1.toronto.ca/wps/portal/contentonly?vgnextoid=3cdcfb292f426410VgnVCM10000071d60f89RCRD&vgnextchannel=1a66e03bb8d1e310VgnVCM10000071d60f89RCRD
[cot]:    http://www1.toronto.ca/wps/portal/contentonly?vgnextoid=9e56e03bb8d1e310VgnVCM10000071d60f89RCRD

<figure>
<a href="/assets/BayStCorridor.png">
<img src="/assets/BayStCorridor.png" alt="Bay St Corridor sidewalks" style="width:550px">
</a>
</figure>


## Background

I took part in the Open Toronto [Open Data Book Club][odbc] last week. This is a neat idea where an open dataset is chosen at the start of the month, and data scientists bring along their analyses to the meetup for a show-and-tell. Discussion takes place around what was good about the dataset, what could be improved, and what further analysis could take place.

[odbc]:   https://www.meetup.com/opentoronto/

The dataset for [last week's meeting][odbc-si] was the [City of Toronto Sidewalk Inventory][cotsi], which lists the type of sidewalk present on all streets in Toronto. It codes street segments based on whether there is sidewalk on both sides, one side only, or no sidewalk, and also includes information on pathways, laneways, and other walkable spaces.

[odbc-si]:  https://www.meetup.com/opentoronto/events/236673636/

I presented colour coded maps of sidewalks by city neighbourhood, giving a visualisation of how good the sidewalk coverage is in each part of town. This article breaks down the R code to do this. 

## Getting the neighbourhood data

First step was to load in the [Toronto neighbourhoods dataset][cotnhs], which is another open dataset curated by the City of Toronto. The neighbourhoods are provided as a Shapefile, which can be read in R using the `readShapeSpatial` function from the `maptools` library. 

[cotnhs]:    http://www1.toronto.ca/wps/portal/contentonly?vgnextoid=04b489fe9c18b210VgnVCM1000003dd60f89RCRD&vgnextchannel=75d6e03bb8d1e310VgnVCM10000071d60f89RCRD

{% highlight r %}
areas <- readShapeSpatial("neighbourhoods2", CRS("+proj=longlat +datum=WGS84"))
{% endhighlight %}

When I first downloaded the neighbourhoods Shapefile and tried to load it, I got the following error:

{% highlight r %}
> readShapeSpatial("NEIGHBORHOODS_WGS84"))
Error in getinfo.shape(fn) : Error opening SHP file
{% endhighlight %}

To solve this, I opened the Shapefile up in [QGIS][qgis] and re-saved it (without doing anything else), and this fixed the problem for me (nice 😎).

[qgis]:   http://qgis.org/en/site/


## Getting the sidewalk data

Next, I had to load in the [sidewalk data][cotsi]. This is also provided as a Shapefile, so I used `readShapeSpatial` again. However, the data is currently tagged with the wrong projection information, so I added a custom projection string:

{% highlight r %}
projection <- "+proj=tmerc +lat_0=0 +lon_0=-79.5 +k=0.9999 +x_0=304800 +y_0=0 +ellps=clrk66 +units=m +no_defs"
sidewalks <- readShapeSpatial("Sidewalk_Inventory_wgs84", proj4string=CRS(projection))
{% endhighlight %}

The second issue I ran into is that the sidewalk position co-ordinates are given in [eastings and northings][eandn], whereas the neighbourhood positions are in latitude and longitude. Thankfully, there is an `spTransform` function in the library `rgdal` that can convert between the two!

{% highlight r %}
sw.lat.long <- spTransform(sidewalks, CRS("+proj=longlat +datum=WGS84"))
{% endhighlight %}

[eandn]:    https://en.wikipedia.org/wiki/Easting_and_northing

Now I have my neighbourhoods and sidewalks on the same co-ordinate system, I can start doing some analysis!

## Finding sidewalks inside a given neighbourhood

The Toronto neighbourhoods have a unique 3-digit code, as well as unique names. One of the core downtown neighbourhoods is the Bay Street Corridor, with code `076`. You can get all the data associated with this one area as follows:

{% highlight r %}
nh <- areas[areas@data$AREA_S_CD=="076", ]
cat(paste0("Area is ", nh$AREA_NAME, "\n"))
nh.pts <- fortify(nh)
{% endhighlight %}

What does that last line do? Well, the neighbourhood `nh` is an object of type `SpatialPolygonsDataFrame`: a complex data structure that includes co-ordinate and projection information, the area name and code, and a representation of the polygon defining the neighbourhood boundary. The `fortify` function essentially flattens this object into a tabular format, where each row represents a single point in the polygon. The table has columns for latitude and longitude of the point, amongst other features. This tabular format is needed for plotting on the map later, so I'll return to it shortly.

In the meantime, the next step is to find which of the sidewalks exist inside the chosen neighbourhood. The `over` function can do this for us! 

{% highlight r %}
inside <- !is.na(over(sw.lat.long, as(nh, "SpatialPolygons")))
sub <- subset(sw.lat.long, inside)
{% endhighlight %}

As long as you downgrade the neighbourhood object to a `SpatialPolygons` first, `over` returns one entry per sidewalk, either 1 (inside) or NA (outside the neighbourhood polygon). If you just pass in `nh` as is, you end up with a table of results with one row per sidewalk, but columns for each of the attributes in `nh`. The values are still either 1 or NA, just repeated across the columns.

I convert this to a list of true and false values with `!is.na`, then use that list to subset out only those sidewalks that are inside the neighbourhood. We're getting close!

## Colour coding the sidewalks

The original sidewalk dataset codes each entry by the type of sidewalk in place:

 1. Sidewalk on south side only
 2. Sidewalk on north side only
 3. No sidewalk on either side
 4. Sidewalk on west side only
 5. Sidewalk on east side only
 6. Pending (Roadway under construction)
 7. Sidewalk on both sides
 8. Not used
 9. Not used
 10. Laneway without sidewalks
 11. Walkway (pedestrian path documented in the City road data)
 12. Pathway (pedestrian path not documented by the City)
 13. Not applicable (a feature where no sidewalk is expected, e.g. expressway)

There are 13 different codes, and I don't want my maps to have such fine-grained detail. I'm just interested in whether the road segment has sidewalks on both sides, one side, or none, and I want to colour code those as green, amber, and red. So I mapped these sidewalk codes into my own scheme as follows:

{% highlight r %}
sdwlk <- c("ok", "ok", "bad", "ok", "ok", "other", "good", NA, NA, "other", "good", "good", "other")
{% endhighlight %}

I decided to include a fourth category called other, for the situations where red/amber/green isn't really applicable. I'll colour these blue so they can be seen without distorting the impression of sidewalk coverage.

Next comes the most complex bit of code. I mentioned before that the `fortify` function is used to convert a complex `SpatialPolygonsDataFrame` object into a more-straightforward tabular format of points which can be drawn on a map. This includes latitude and longitude, but loses the sidewalk code attribute `SDWLK_Code`. I need to assign a known unique id to each of the sidewalks, so that I can merge the sidewalk code information back in to the fortified points. I then derive a sidewalk coverage indicator for each point based on the sidewalk code and my coverage scheme `sdwlk`:

{% highlight r %}
sub@data$id <- rownames(sub@data)
sub.pts <- fortify(sub)
sub.pts <- merge(sub.pts, sub@data, by="id")
coverage <- sdwlk[sub.pts$SDWLK_Code]
sub.pts <- cbind(sub.pts, coverage)
{% endhighlight %}

At this stage, I have all the information needed to map the sidewalks!


## Mapping the result

The library `ggmap` lets you easily download Google Maps for areas at various zoom levels using `qmap`. The following code downloads a map, then plots the neighbourhood boundary and colour-coded sidewalks on top of it:

{% highlight r %}
toronto <- qmap("Toronto, Ontario", zoom=14)
toronto +geom_polygon(aes(x=long, y=lat, group=group, alpha=0.25), data=nh.pts, fill='white', show.legend=F) \
  +geom_polygon(aes(x=long, y=lat, group=group), data=nh.pts, color='black', fill=NA, show.legend=F) \
  +geom_path(aes(x=long, y=lat, group=group, color=coverage), data=sub.pts, show.legend=F) \
  +scale_colour_manual(values=c("good"="#00CC33", "ok"="goldenrod", "bad"="red", "other"="blue"))
{% endhighlight %}

You can see that this uses the fortified versions of both the neighbourhood and the sidewalks for plotting. By passing the `color=coverage` parameter to `geom_path`, I'm saying that I want the colour of each sidewalk to be dependent on its coverage code. However, I also need the `scale_colour_manual` part at the end of this line to assign specific colours to each code, otherwise a default colour scheme is used.

## Output

Now it's time to explore different neighbourhoods! The Bay Street Corridor is right in the heart of downtown Toronto, and as may be expected, the sidewalk coverage is very good. The Waterfront Communities are also a core area, and have similarly good coverage. However, once you get a bit further out from downtown, the coverage can drop off quite badly. The Maple Leaf neighbourhood, for example, shows a lot of red and amber.

<div class="tabs">
  <button class="tabs__tab current" onclick="showTab(event)" data-id="bay-street">Bay St Corridor</button>
  <button class="tabs__tab" onclick="showTab(event)" data-id="waterfront">Waterfront</button>
  <button class="tabs__tab" onclick="showTab(event)" data-id="maple-leaf">Maple Leaf</button>
</div>

<script type="text/javascript">
function showTab(event) {
  var tab = event.target;
  var tabs = document.querySelectorAll('.tabs__tab');
  var i, figure;

  for (i = 0; i < tabs.length; i++) {
    figure = document.getElementById(tabs[i].dataset.id);
    if (tabs[i] === tab) {
      tabs[i].classList.add('current');
      figure.classList.remove('hidden');
    } else {
      tabs[i].classList.remove('current');
      figure.classList.add('hidden');
    }
  }
}
</script>

<figure id="bay-street" class="square-figure">
<a href="/assets/BayStCorridor_normal.png">
<img src="/assets/BayStCorridor_normal.png" alt="Bay St Corridor sidewalks">
<figcaption>Bay St Corridor</figcaption>
</a>
</figure>

<figure id="waterfront" class="square-figure hidden">
<a href="/assets/Waterfront.png">
<img src="/assets/Waterfront.png" alt="Waterfront sidewalks">
<figcaption>Waterfront</figcaption>
</a>
</figure>

<figure id="maple-leaf" class="square-figure hidden">
<a href="/assets/MapleLeaf.png">
<img src="/assets/MapleLeaf.png" alt="Maple Leaf sidewalks">
<figcaption>Maple Leaf</figcaption>
</a>
</figure>

The full code is available on GitHub, so you can [go ahead and explore your own neighbourhood][github]!

[github]:   https://github.com/cowlet/open-toronto/blob/master/sidewalks.R

