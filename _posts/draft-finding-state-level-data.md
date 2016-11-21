#Fun with State Level Data
Finding state level data can be a bit problematic, depending on what you are looking for.  Federal data is very accessible through several avenues such as **[[cite]]**, but there are no consistent overarching regulations governing access to state-level data such as state congressional districts. A search on www.data.gov for congressional districts returns some states, but not others.  

I'm not a data scientist or GIS guru, so finding the data I wanted was something of a challenge.  In this case, I wanted to obtain GEOJSON data on state congressional districts for the North Carolina General Assembly to be used in a [d3.js](https://d3js.org/) data visualization.  Federal data [[from where?]] is easily obtainable, but state data is inconsistent.  Several states have this data available on www.data.gov, but North Carolina does not.  There isn't a universal source of state-level data where users can say, write a script to pull data from an API for all 50 states and get consistent results.

After much google fu [[more detail?]] and some assistance from the, the best source of NC district data I could find was hidden away at the state legislature's [website](http://www.ncleg.net/geosrv/rest/services/HSC_Districts), which I discovered inadvertently by using NC State University's [GIS Search](https://www.lib.ncsu.edu/gis/index.html).  You can query their ArcGIS system at this location.  Still, not much guidance is provided as to ~how~ to query it in a useful fashion.  By digging around some more, you can retrieve a [map](http://www.arcgis.com/home/webmap/viewer.html?url=http%3A%2F%2Fwww.ncleg.net%2Fgeosrv%2Frest%2Fservices%2FHSC_Districts%2FNC_House_Districts_2011_Enacted%2FFeatureServer%2F0&source=sd) with additional data.  This map has a table associated with it, that reveals a mapping of district numbers to OBJECTID, one of the queriable fields of the API.  From an outsider's perspective, it seems somewhat haphazard and arbitrary (why not just make the district number the OBJECTID?), but we've got the data we need to query the API.

OBJECTID | District
--- | ---
15 | 110
16 | 111
17 | 112
18 | 113


Querying this returns pretty-printed JSON! [(example)](http://www.ncleg.net/geosrv/rest/services/HSC_Districts/NC_House_Districts_2011_Enacted/FeatureServer/0/query?where=&objectIds=3%2C+4&time=&geometry=&geometryType=esriGeometryEnvelope&inSR=&spatialRel=esriSpatialRelIntersects&distance=&units=esriSRUnit_Foot&relationParam=&outFields=&returnGeometry=true&maxAllowableOffset=&geometryPrecision=&outSR=&gdbVersion=&returnDistinctValues=false&returnIdsOnly=false&returnCountOnly=false&returnExtentOnly=false&orderByFields=&groupByFieldsForStatistics=&outStatistics=&returnZ=false&returnM=false&multipatchOption=&resultOffset=&resultRecordCount=&f=pjson)  So far, so good.

There's one problem with this .  I took an unwitting deep dive into the world of [geographic coordinate systems](http://resources.arcgis.com/en/help/main/10.1/003r/pdf/geographic_coordinate_systems.pdf), of which there are apparently many.


This might be a better data source:
https://tigerweb.geo.census.gov/tigerwebmain/TIGERweb_restmapservice.html
http://www.web-maps.com/gisblog/?cat=24
https://github.com/Schwanksta/python-arcgis-rest-query
