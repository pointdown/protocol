*(this description derived from a mailing list post by Michael P. Gerlek, mpg@flaxen.com)*

Rialto is a project within RadiantBlue to build a point cloud viewer on top of Cesium.


# DATA SET

Determined via the URL prefix, e.g. http://www.example.com/points/serpentmound/


# METADATA

A json file at the dataset URL root describes things including the tiling scheme, the bounding boxes of both the level-0 tile extents and the point cloud data, and the dimensions (name, datatype, min/mean/max).


# TREE STRUCTURE

Basic quadtree, where a rectangular node at level L reduces to 4 equally-sized nodes at level L+1. The node payload includes the point data itself plus a byte at the end that indicates which of the 4 children actually exists on disk; the root tile(s) are guaranteed to exist.
“Top-down” view only: that is, I use rectangles not cubes. The design is adapted straight from http://cesiumjs.org/data-and-assets/terrain/formats/heightmap-1.0.html.


# PAYLOAD

Each tile contains a set of points covering that tile’s spatial extent. The points are represented in raw form, in point order: X0, Y0, Z0, T0, X1, Y1, Z1, T1, etc. Data is stored little-endian in the datatype described in the JSON header.


# POINT SELECTION

All the points are contained in the leaf node tiles at level N. Each tile at level N-1 contains 1/4 of the points contained in the level N nodes. The “decimation” process currently used just naively takes every 4th point, but I would cal this an implementation detail: the tiling scheme does not mandate how many points should be in a tile or how they are chosen.


# COMMENTS, ASSUMPTIONS, PROBLEMS, ETC:

- I’m aiming at the simplest approach that could reasonably work, with the least cost to the server to generate and the client to interpret.

- I currently assume that level 0 has exactly two root tiles covering the whole globe, but I will generalize it to be able to have a single root tile for a given region.

- I was using web sockets, but switched back to http because I’m (now) assuming that number of bytes in a tile is “reasonable”, i.e. doesn’t justify the need for sockets

- No compression is used, although I’d like to gzip each tile. Just because.

- Using a “las2tiles” approach to build the tiles, based on PDAL. Should probably make that code into a Real PDAL Driver at some point.
