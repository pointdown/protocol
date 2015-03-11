*(this description derived from a mailing list post by Markus Schuetz, mschuetz@potree.org)*

# Data Structure

We're using a multi resolution octree. Each octree node
contains a subset of the point cloud covering the respective area.
The root node (level 0) contains a very coarse representation of the whole
dataset, e.g. 20.000 points. With each level down the octree,
the resolution increases but the covered area decreases.

The name/ID of each node indicates its exact position in the octree.
The root node is named r.
After that, to get the ID of a node we simply append its index (0-7)
to its parents ID.
This means that the first child of the root node(index: 0) has the ID
r0. The eight child of the root node (index 7) has the ID r7.
The third child of r0 (index 2) has ID r02, and so on.

Here are some example nodes:

- level 0: http://potree.org/demo/potree_2014.12.30/resources/pointclouds/lion_takanawa_las/data/r.las

- level 1: http://potree.org/demo/potree_2014.12.30/resources/pointclouds/lion_takanawa_las/data/r0.las

- level 2: http://potree.org/demo/potree_2014.12.30/resources/pointclouds/lion_takanawa_las/data/r03.las

The data format itself is not important. I'm usualy using a binary dump with
coordinates encoded in 3x32 bit integers with scale and offset, as well as color encoded in 4 bytes but
las and laz are also possible.

Here is the same dataset with nodes stored in binary dump, las or laz:

- http://potree.org/demo/potree_2014.12.30/examples/lion.html
- http://potree.org/demo/potree_2014.12.30/examples/lion_las.html
- http://potree.org/demo/potree_2014.12.30/examples/lion_laz.html


# Metadata

Metadata is stored in a cloud.js file which contains bounding box, spacing used for the indexing, scale
and hierarchy. The problem with the hierarchy is, that if there is a large amount of nodes, the metadata
can grow to >10mb.
I've recently started to split the hierarchy as well and only load parts of the hierarchy that are needed.
For example, if the octree has a depth of 12, then it is sufficient to initially load 7 levels of
the hierarchy. Once the users zooms closer into a certain area of the point cloud, additional levels
of hierarchy are loaded. Right now I'm experimenting with loading the next 7 levels of
hierarchy for all visible nodes of level 6.
Instead of one large 10mb file, we now get 1000 small ~10kb files which are loaded as needed.


# Rendering

During rendering, the octree is traversed breadth first and closer to camera first.
Nodes outside the view frustum as well as nodes whose bounding sphere is small when projected onto the
screen are skipped. If a point count limit is reached, traversal stops and the visible nodes up
to that point are rendered. If some nodes are visible but their points have not been loaded yet,
a XMLHttpRequest is created to load the node file from the server.
Since points are stored in all nodes, root, inner and leaf nodes, all of them are rendered if they
are visible. That way, the user already gets to see a low resolution version of the point cloud in
areas where higher resolutions have not been loaded, yet. In order to avoid holes in areas with lover
resolution, point size is adapted accordingly.
See this example: http://potree.org/demo/potree_2014.12.30/examples/matterhorn.html
Reducing the point count limit, using the "points(m)" slider, will increase point sizes.


# Additional Information

- The server is only needed to host files. There is no server side application running.
Each node is stored in its own file and during traversal of the hierarchy,
the client finds out which files it needs to request from the server.
