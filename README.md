For ease of comparing and contrasting approaches, to describe a format we suggest following roughly this decomposition:

1. **Data set access:** how does the client reach the server (e.g. http urls)?

2. **Metadata:** how does the system describe the underlying actual point data (e.g. json file at root)?

3. **Data structure:** how are the points grouped and arranged on the server (e.g. hierarchical tiles)?

4. **Payload:** for a unit of data transfer, how are the points arranged (e.g. all dimensions in row-order)?

5. **Point selection:** how does the system decide which points to put into which piece of the data structure (e.g. log2 decimation)?

6. **Other comments, assumptions, problems, etc**

This decomposition somewhat presupposes a vaguely static tile-hirearchy approach. If this is not the case, the system can be described along different dimensions.

The several formats we know of today for streaming point cloud data are described here:

* [pointcloudviz](pointcloudviz.md)

* [escience/Netherlands](escience.md)

* [Displaz](displaz.md)

* [potree](potree.md)

* [Rialto](rialto.md)
