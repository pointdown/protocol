*(this description derived from a mailing list post by Chris Foster, chris42f@gmail.com)*

I've also been prototyping a viewer and processor for arbitrary sized
point clouds in my point cloud project
(https://github.com/c42f/displaz).  I've been aiming for much the same
thing as everyone else (apparently!): a standard level of detail
representation which can be loaded incrementally by various
visualization systems, the most obvious being javascript/webgl.
However, I'm doing prototyping in displaz (a desktop application)
because it's a productive environment for me personally to get
something going quickly.

Here's a brain dump of where I'm at currently.  Apologies for the length!

I think I've been coming at the problem from a slightly different
angle - I'm firstly trying to nail down what is required for scalable
high quality visualization before deciding on a format.  To support
the research I do have a file format (currently called "hcloud" in the
displaz source) but it's intentionally very bare bones in the
interests of being as simple as possible.  I've just started reviewing
the literature for point cloud compression techniques and how they
relate to the needs of a streaming format in the last few weeks.

# DATA SET

Currently only local file names are supported, but I was planning to
support remote sources as simple file URLs on a server supporting
range-based GETs (for example, an Amazon S3 bucket).  With careful
ordering of the data within the file, parts of the LoD structure which
are needed together by the client can also be nearby in the file which
would allow for fairly efficient streaming with basically zero server
infrastructure.  I haven't yet spent much time thinking about how this
would play with authentication and proxy caching but I think the
general scheme can be made to work.

# METADATA

Obviously required, but I haven't tried to design this yet (the
existing hcloud header is a simple binary format not recommended for
general consumption ;-) ).  Vague plan would be to go with a simple
extensible metadata format (json would be ok I guess) with a magic
number, version and header size prefix.

# TREE STRUCTURE

hcloud contains an octree where each interior node has an NxNxN cube
of points in sparse format, with the leaf nodes holding the original
points.  An index is stored separately at the end of the file so that
arbitrarily large amounts of point data can be rendered into a single
file in a single streaming read from a point database.  This part
works quite nicely already and the file creation tool "dvox" is able
to process an arbitrary number of points in something like
O(log(numPoints)) memory, and O(numPoints) compute cost.  From memory
I tried about 1e9 points before I got bored waiting for the database
load; I'd give some actual stats, but I'm on holiday at the moment and
away from my dev box.

# PAYLOAD

Per above, each interior node contains a sparse set of NxNxN points
where N is defined in the header, and the leaf nodes contain a list of
the original points.  In the prototype each point has X,Y,Z,coverage
and lidar intensity.

Obviously the format needs to be flexible enough to support colour,
and I was intending to design header metadata to support arbitrary
point attributes, with certain "blessed" attributes (colour and
intensity, possibly others) as a base requirement to be supported by
readers.

# POINT SELECTION - All the points are contained in the leaf node tiles at
level N. Each tile at level N-1 contains 1/4 of the points contained in the
level N nodes. The “decimation” process currently used just naively takes
every 4th point, but I would cal this an implementation detail: the tiling
scheme does not mandate how many points should be in a tile or how they are
chosen.

LoD creation for the interior nodes is where I've done the most work
so far.  The hcloud format contains voxels as interior nodes of the
tree, formed by rendering the points of the child nodes from a
preferred direction and taking occlusion into account.  For the
prototype the preferred direction is viewing from +z which works well
for aerial lidar, but a practical format for terrestrial lidar will
probably need more than one preferred direction).

Viewing LoD creation as a rendering task rather than a data selection
task is important for the best visual results because it correctly
antialiases the points, just like the pixel filtering which is
standard in image pyramid creation.  For image pyramids, it's enough
to average adjacent sets of four pixels to produce reasonable results
at the next coarser level of detail.  Unfortunately averaging the
eight nighbouring voxels in the 3D case often produces bad results
because it ignores occlusion between points.  This is particularly
noticeable if you have layered data such as vegetation (dark) over
ground returns (bright).  Averaging in this case produces
inconsistencies when you switch between levels and nasty banding when
the nodes within a given level cut through an inclined layered
surface.

An obvious alternative to averaging is to randomly choose a single
point to represent those at the higher level of detail.  Unfortunately
this replaces the systematic bias of averaging with noise, which is
particularly objectionable at low detail levels.

# COMMENTS, ASSUMPTIONS, PROBLEMS, ETC:

* As you can see, I'm coming at this from a rendering perspective.  As
far as I can tell, good results for general 3D data will require
specifying the preferred viewing direction(s) somehow in the stream,
which isn't something I've tried to do yet.  However, what I have so
far works very well for aerial lidar.

* I'd really like arbitrary point attributes because they provide
great flexibility, but they do have downsides:

- A standard needs to strongly recommend a base set of attributes
and their semantics, otherwise you get the .ply format where there's a
proliferation of subtly different interpretations of "standard" files.

- A compression scheme has no inherent insight into arbitrary
attributes which makes correlation modelling harder and will probably
damage compression ratios unless some basic attribute semantics are
included in the metadata.

- There's a little added implementation complexity.  Actually I
doubt this is really worse than formats such as las where there's a
bit of a combinatoric explosion of special case formats which
inevitably don't do quite what you want.

Further random thoughts in no particular order:

* My file format sucks in detail (the points are at least 10x larger
than they need to be), but I think the philosophy behind the data
layout is useful: it's an approximate breadth-first traversal of the
LoD tree in Morton order which I think gives reasonably good data
locality.  Data locality is a quality-of-implementation issue rather
than a format design issue because the octree index is stored
explicitly.

* The data streaming requirements of image pyramids are a useful point
of comparison.  A high quality jpeg weighs in around 2 bits per pixel,
and significantly less than this still gives adequate quality.  It
seems that current point cloud compression techniques are about 10x
worse per point.  Surprisingly this cost seems to be mostly the cost
of compressing colour information and other auxiliary attributes
rather than position itself [citation needed - see below].

* It would be great if we could collaborate on a literature review of
both point cloud rendering and compression techniques.  On the
rendering side, there's a lot to be learned from the high end
rendering literature, particularly from people trying to do point
based global illumination.  I'll need to scrape together a list of
what I've read, but again, I'm on holiday which is making it a bit
hard.  IIRC a good place to start is "Coherent Out-of-Core Point-Based
Global Illumination" by Kontkanen et al..  On the compression side of
things, the laszip paper is an obvious place to start.  Great as it
is, I suspect laszip isn't quite the right thing here: somewhat lossy
compression would probably be acceptable for a significant bitrate
improvement, and we should really exploit the tree structure which can
achieve remarkable compression rates for the position attribute.

* To support the incremental mosaicing of large and growing datasets
(let's say at least 1e12 points), parallel processing for creation of
the LoD structure is an absolute requirement.  For a file format, this
can be achieved fairly simply by splitting the problem into tiles, and
bringing all tiles together into a higher level pyramid where leaf
nodes point to the URLs of subpyramids.
