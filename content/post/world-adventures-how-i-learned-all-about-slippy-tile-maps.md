---
timeToRead: 5
authors: ["Chip Keyes"]
title: "World Adventures: How I Learned All About Slippy Tile Maps"
excerpt: 'I have to say; it wasn’t until recently that I really began to appreciate the effort that goes into those generic interactive maps that you see in widgets or...'
date: 2017-08-24
hero: ''
draft: false

---
I have to say; it wasn’t until recently that I really began to appreciate the effort that goes into those generic interactive maps that you see in widgets or full sites like Google Maps, Craigslist, and many others.

When I started working on my current project (building out a React client-side application for some pretty standard data entry), I had no idea that I would work my way into learning about all the pieces that go into making those widgets do what they do.

My hope is that my experience with this rather impressive system might pique someone else’s interest in map rendering for the web. I also want to provide a 1000 foot view of a pretty complex system. There are a lot walkthroughs out there that give you step-by-step terminal commands to get a tile server up and running, but I want to provide a little more practical knowledge about what each piece actually does.

#### The Problem

Imagine with me that you’ve been tasked to create a map that allows your end user to zoom in and out of the map display, and move it to display different locations.

Again — imagine with me that you’ve been tasked with creating a map that will allow an end user to zoom in and out, move around to different locations around the world, AND see real data about roads, houses, buildings, and businesses. This, as anyone who has used Google Maps, Craigslist, Yelp, or virtually any other location-based tool will know, is not an unheard of build request. Whether it’s basic navigation, data presentation, or viewing specific locations, there are plenty of scenarios in which someone would want a fully interactive map on their site.

At the surface, one might think this request is easily solved by pulling up an image of a world map and calling it done. But, remember, we’ve also got to account for different zoom levels and the ever changing data that needs to be displayed (think road closures and building changes).

As you can see, this is a much more difficult problem than it might initially appear to be.

Enter Open Street Maps…

### The Curator

#### OpenStreetMaps

Open Street Maps (or OSM) is an open source database that stores all the geospatial data that its users need using their [map editor](https://www.openstreetmap.org/). As I mentioned above, this data contains information about buildings, businesses, roads, parks, waterways…and the list goes on.

Basically, if you can find it on their map, you can add information about it.

This is, in and of itself, a pretty cool tool, because OSM exposes their own tile server for public use. However, they take it a step further by making their data public and curating organized diff files. This allows you to both download the complete up-to-date world data and keep your data in sync with OSM’s database by regularly downloading and processing their diff files.

The file(s) you get from OSM are a specially compressed XML file that pushes upwards of 50GB+ in its compressed form for the whole world’s data. While you can literally (well, maybe not so literally) hold the whole world in your hands (or your computer), that XML data isn’t terribly useful in that format. Thus, we bring in our first tool in the long chain of turning a data dump into one of those beautiful maps you see pop up on your favorite site.

### The Engine

#### Imposm3 (or osm2pgsql)

The first step after downloading the massive data file is to process the XML into an extended PostgreSQL database (PostGIS) that can be queried more efficiently. As you might imagine, this is pretty ‘computationally expensive’ since it has to parse a heavily nested XML tree, put the data into a structured schema and insert it into the PostgreSQL database. When I say that it’s computationally expensive, I mean it will take anywhere from 4–14 days to process the entire world’s data, resulting in a database that’s close to 300GB.

The reason I listed both [Imposm3](https://github.com/omniscale/imposm3) and [osm2pgsql](https://github.com/openstreetmap/osm2pgsql) is because while osm2pgsql is the sort of standard de facto C++ library, I found that it takes significantly longer than the Imposm3 tool, which is written in Go (most likely due to Go’s ease of concurrent and parallel programming). I also found Imposm3 to be substantially easier to configure as well as easier to fetch and process the diff files. I actually didn’t finish the osm2pgsql process because Imposm3 finished so much faster (~4 days). Regardless of which tool you use, make sure it’s running on a fast computer with lots of space…and then go take a week vacation.

### The Artist

#### Mapnik

Finally! We have data in a PostgreSQL database that can be queried.

…Now what?

Well, now we need something to come in and turn the data into a nice pretty image that looks like a map. The API to handle this task is called [Mapnik](https://github.com/mapnik/mapnik). It’s a C++ library that can read geospatial data from a PostGIS database, combine it with some pre-rendered shape files (e.g., country borders) and a stylesheet to render a png image of the designated map region.

Of course, that sentence makes it sound a lot easier than it actually is. But this is where the magic happens. Our massive database is actually useful to us now that the data can be rendered into images.

You might be tempted to think we’ve made it —and we’ve come a long way from a compressed XML file. But, the fun is just beginning! While we’re now able to render individual tiles, we don’t have a way to fetch the ones we want to see or to stitch them together in a coherent way.

### The Butler

#### Apache Mod Tile/Renderd

Now that we have the ability to render a png at a specific zoom level and GPS location, we need a way to abstract away all the rendering logic in a way that our server can just take a request for an area and know exactly which tiles have to be rendered (or already have been and are cached).

Mod Tile is a mod for the Apache Server which does exactly this. In combination with the Renderd daemon, which orchestrates the tile expiring and cache layer, [ModTile](https://github.com/openstreetmap/mod_tile/) takes requests for a tile and passes that along so that Mapnik can render it or, if it’s been recently rendered, can grab it from the cache layer.

The request at this stage looks something like `<url>/tiles/<zoom>/<x-tile>/<y-tile>`. It took me a while to properly interpret the zoom, x, and y values in the URL, so I’ll quickly explain it here.

The main thing to keep in mind is that a tile server ultimately takes a map of the world and, depending on the zoom level, breaks it into a set number of tiles using a basic grid. So, at zoom level zero, there is exactly one tile which corresponds to tile (0, 0) — which is the whole world. At zoom level 1, there are four tiles corresponding to (0,0), (0,1), (1,0), and (1, 1) which break the world down into for tiles at a slightly zoomed in view. This goes on for as many zoom levels can be represented by the data.

This is all fine and dandy, except that it means we only have the ability to request a single tile at a time and, therefore, just show a single out of context chunk of the world map. Not super useful…but wait — there’s more.

### The Conductor

#### LeafletJS/OpenLayers

The final piece (which is actually a client for the tile server rather than a piece of it) is the javascript rendering libraries that abstract away the logic for displaying a given GPS range, as well as all the user interactions. As I mentioned, without these tools we can only request tiles individually—unless we go through the effort to take GPS bounds and request several tiles to fill that space at that zoom level. Then, if the user drags the map, we have to request a new bounding box which will potentially include some of the already rendered tiles, as well as a few new ones.

This is the scenario I described at the beginning, and this is exactly what [LeafletJS](http://leafletjs.com/) or [OpenLayers](https://openlayers.org/) do. These are Javascript API’s that allow all the tile requests to be abstracted away so that the front end developer can think in terms of GPS coordinates, rather than the more esoteric tile grid. They also expose many of the click events and allow additional layers (i.e., map markers, routes, etc.) to be placed on the tile map.

### In Conclusion…

So, there you have it. Start-to-finish of a tile (slippy) map server. Obviously, this is a bird’s eye view of the technology, and there is much that I don’t even understand yet — much less be able to explain in a short article. I’m also not sure how often a use case for building a tile server comes up, as most applications can just use Google’s map service or OpenStreetMaps directly. The scenario that inspired us to build one came from a lack of connectivity during the lifetime of the application. Therefore, to display maps without internet, we needed a tile server on our intranet.

At the very least I hope, like me, you have a greater appreciation for the effort that went into making the beautiful interactive world maps that we have come to expect on almost every site related to locations.
[sc name="post-signature"]