*(this description derived from a mailing list post by Rafael Gaitán, rafa.gaitan AT mirage-tech.com)*


# Data Set

I agree with [the Rialto URL approach], but I'll add some more parameters, since there are multiple possible clients and different purposes, for instance our server supports dataset selection by url, but also the format, compression, and also the "pre-generated" tiles

For instance:

- http://server.pointcloudviz.com:9090/getTile/utah/0/0/0/0 -> Retrieves the level 0 of the utah dataset in what I called "compact Json" format, but other more verbose format can be developed

- http://server.pointcloudviz.com:9090/getTile/osgjs/utah/0/0/0/0 -> Retrieves the level 0 in OSGJS format, ready to be displayed on web clients using OSGJS

- http://server.pointcloudviz.com:9090/getTile/osgjsb/utah/0/0/0/0 -> Same as before but retrieves the OSGJS node structure and the data is requested later in binary format

- http://server.pointcloudviz.com:9090/getDBFile/utah/utah.plf -> Retrieves the index file ready to be used in the PointCloudViz(http://www.pointcloudviz.com) application.

- http://server.pointcloudviz.com:9090/getDBFile/utah/0/0/0/utah_L0_X0_Y0_Z0.bin -> Retrieves the data in binary format ready to be used in WEBGL and OSGJS.

The idea is a web services in which you can decide/provide what is more convinient for your client application.


# Metadata

I agree again [with the Rialto approach], but following the idea of WebService and using a similar OGC terminology we propose an API similar to:

- http://server.pointcloudviz.com:9090/getConfig -> Retrieves the configuration/metadata of the registered datasets

- http://server.pointcloudviz.com:9090/getCapabilities/autzen -> Retrieves the "capabilites" of a specific dataset

Additionally in our server /getConfig also sends the API, the formats and the compressors, which has been very handy when I was working developing the client:

```
"formats": [
    "osgjs",
    "osgjsb",
    "cjson"
],
"api": {
    "/getCapabilities/:db": "Retrieves the capabilities and statistics of the selected database",
    "/getConfig": "Retrieves the configuration and API",
    "/getDBFile/:db/*": "Serves direct original files of the database",
    "*": "Other files of the web",
    "/getTile/:format/:compressor/:db/:level/:x/:y/:z": "Retrieves a specific tile of the selected database in :format format",
    "/getTile/:db/:level/:x/:y/:z": "Retrieves a specific tile of the selected database in json format",
    "/": "Retrieves the index.html",
    "/getTile/:format/:db/:level/:x/:y/:z": "Retrieves a specific tile of the selected database in :format format"
},
"compressors": [
    "none",
    "gz"
]
```

# Tree Structure

[The Rialto approach] is perfectly valid for heightmaps and images, but poinclouds could be real 3D, not only aereal, but also from ground scanners (see: http://server.pointcloudviz.com:9090/?config=utah.osgjsb)

Our format is and adaptive Octree, which we have developed some tools to process unlimited size datasets (I've done some tests with 3Billion point in the pointcloudviz desktop application)

The idea is similar to the quadtree but using as designation 4 values: Level, X, Y, Z. How this is generated it's independent of the server as long as you can provide different levels of detail.

Our prototype uses a "lazy" approach and uses the pointcloudviz format for serving the tiles, but as long as the API is the same, the tiles on disk could be any format.


# Payload

[The Rialto approach] could be one of the formats, or even the default format using the described API. Also remember that pointclouds are not only "points", they also have a lot of other useful information as the classification, the intensity, etc etc that need to be sent/queried to the server.

Our approach sends always all the information per tile, since the number of requests to the server when using web clients were too many, and that gave us poor performance, but still more work need to be done on the server side.


# Point Selection

I agree [with the Rialto approach], this is something that the API/Server should solve when the request is performed. Our prototype still does not support that but it could be implemented computing the best level for the query and extracting the tiles that intersect that query, serving them in multiple tiles or one dataset.



# Comments, Assumptions, Problems, etc

* [The Rialto proposal] and mine are similar, so the same applies, Our preliminary work says that the system works, but scalability is something we need to work for.

* In our case, we are projection-independent, so we always start with a single root tile, which for cesium/WGS84 clients the data could be sent reprojected.

* That's something we need to work more in our server, since not always is "easy" or standardized the way that LAS files save internally the projection (if they have it).

* Using http compression (gzip) headers and a reasonable good server which supports it, will do the trick for plain text formats, or even binary formats. We are using that approach for the OSGJS/WebGL client, tiles are retrieved using that header, and the server sends them gzipped but is the browser who uncompresses it before being used by JavaScript.

* As I mentioned before the tilling system could be generated with las2tiles or whatever other technology as long as the API is preserved, same as any other OGC format.
