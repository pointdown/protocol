*(this description derived from a mailing list post by P.J.M.vanOosterom at tudelft.nl)*

It is great to see the interest, on the pointdown mail list, in
standardized streaming of point clouds. This message expresses some of
our ideas on the topic as a result of the (Netherlands) eScience project
on the management of massive point clouds. We have discussed possible
standardization of point clouds several time with the project partners
(Fugro, Oracle, Netherlands eScience Center, TU Delft and
Rijkswaterstaat) and considered two closely related levels of
standardization:

1. Database SQL (Structure Query Language) extension for point
clouds, and

2. Web Point Cloud Services (WPCS) for progressive transfer of point
clouds.

From our internal discussions we concluded that (initial) emphasis
should be on web services. The end-users, from industry, government, or
research, want interoperability and be able to combine point cloud data
from various sources with functionality in tools form other vendors/
developers, or even own developed specific purpose applications.  As the
focus is on geographic applications, these point cloud standards should
fit and be complimentary to existing standards from the Open Geospatial
Consortium (OGC) and/or the International Organisation for
Standardisation/Technical Committee 211 (ISO TC211). Quite a number of
aspects have to be covered, resp. taken into consideration:

1. actual point representation xyz (a lot, used SRS, various base
data types: int, float, double, number, or perhaps even varchar..)

2. attributes per point (0 or more; e.g. intensity I, color RGB or
classification, or ...); note that this might conceptually correspond to
a 4D, 5D or higher dimensional point),

3. fast access (based on spatial coherence) -> some blocking scheme
(in 2D, 3D, …)

4. space efficient encoding/storage/transfer -> compression
techniques (exploiting spatial cohesion)

5. data pyramid (LoD, multi-scale/vario-scale, perspective) support,
combined with streaming/ progressive transfer

6. temporal aspect, options for time per point (costly) or block
(less refined)

7. selection accuracies (blocks, individual points on 2D, 3D or nD
query range/ other selection geometries)

8. operators, functionality for specifying some manipulation,
deriving attributes, specifying specific subsets, etc.

Assume that the standardization idea gets enough momentum, we would be
very interested in active participation in such an effort (and would be
in favour of have ISO or OGC as environment).

*PS. We are approaching the issue from the data management perspective,
more info on our activities on pointclouds.nl*
