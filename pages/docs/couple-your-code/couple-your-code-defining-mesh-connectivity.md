---
title: Step 8 – Mesh connectivity 
permalink: couple-your-code-defining-mesh-connectivity.html
keywords: api, adapter, projection, mapping, edges, triangles
summary: "So far, our coupling mesh is only a cloud of vertices. This is sufficient for most of the numerical methods that preCICE offers. For some features, however, preCICE also needs to know how vertices are connected to each other. In this step, you learn how to define this so-called mesh connectivity."
---


The most important example where mesh connectivity is needed is the `nearest-projection` mapping, where the mesh we project _into_ needs mesh connectivity. For a consistent mapping, this is the mesh _from_ which you map. For a conservative mapping, the mesh _to_ which you map. More information is given on the [mapping configuration page](configuration-mapping).

There are two types of connectivity, which depend on the type of coupling.
For surface coupling in 2D, mesh connectivity boils down to defining edges between vertices. In 3D, you need to define triangles and / or quads.
For volume coupling in 2D, mesh connectivity boils down to defining triangles and / or quads between vertices. In 3D, you need to define tetrahedra introduced in version `2.5.0`.

All kind of connectivity can be built up directly from vertices. Triangles and quads also allow us to define them using edge IDs.

```cpp
int setMeshEdge (int meshID, int firstVertexID, int secondVertexID);
void setMeshTriangle (int meshID, int firstEdgeID, int secondEdgeID, int thirdEdgeID);
void setMeshTriangleWithEdges (int meshID, int firstVertexID, int secondVertexID, int thirdVertexID);
void setMeshQuad(int meshID, int firstEdgeID, int secondEdgeID, int thirdEdgeID, int fourthEdgeID);
void setMeshQuadWithEdges(int meshID, int firstVertexID, int secondVertexID, int thirdVertexID, int fourthVertexID);
void setMeshTetrahredron(int meshID, int firstVertexID, int secondVertexID, int thirdVertexID, int fourthVertexID);
```

* `setMeshEdge` defines a mesh edge between two vertices and returns an edge ID.
* `setMeshTriangle` defines a mesh triangle by three edges.
* `setMeshTriangleWithEdges` defines a mesh triangle by three vertices and also creates the edges in preCICE on the fly. Of course, preCICE takes care that no edge is defined twice. Please note that this function is computationally more expensive than `setMeshTriangle`.
* `setMeshQuad` defines a mesh quad by four edges.
* `setMeshQuadWithEdges` defines a mesh quad by four vertices and also creates the edges in preCICE on the fly. Again, preCICE takes care that no edge is defined twice. This function is computationally more expensive than `setMeshQuad`.
* `setMeshTetrahredron` defines a mesh tetrahedron by four vertices.

If you do not configure any features in the preCICE configuration that require mesh connectivity, all these API functions are [no-ops](https://en.wikipedia.org/wiki/NOP_(code)). Thus, don't worry about performance. If you need a significant workload to already create this connectivity information in your adapter in the first place, you can also explicitly ask preCICE whether it is required:

```cpp
bool isMeshConnectivityRequired(int meshID);
```

{% warning %}
The API function `isMeshConnectivityRequired` is only supported since v2.3.
{% endwarning %}

Maybe interesting to know: preCICE actually does internally not compute with quads, but creates two triangles. [Read more](https://precice.discourse.group/t/highlights-of-the-new-precice-release-v2-1/274#2-1-using-quads-for-projection).

{% warning %}
Quads are only supported since v2.1. For older version, the methods only exist as empty stubs.
{% endwarning %}

The following code shows how mesh connectivity can be defined in our example. For sake of simplification, let's only define one triangle and let's assume that it consists of the first three vertices.

```cpp

[...]

int* vertexIDs = new int[vertexSize];
precice.setMeshVertices(meshID, vertexSize, coords, vertexIDs); 
delete[] coords;

int edgeIDs[3];
edgeIDs[0] = precice.setMeshEdge(meshID, vertexIDs[0], vertexIDs[1]);
edgeIDs[1] = precice.setMeshEdge(meshID, vertexIDs[1], vertexIDs[2]);
edgeIDs[2] = precice.setMeshEdge(meshID, vertexIDs[2], vertexIDs[0]);

if(dim==3)
  precice.setMeshTriangle(meshID, edgeIDs[0], edgeIDs[1], edgeIDs[2]);

[...]

```

## Changes in v3

Version 3 overhauls the definition of meshes.

Connectivity now consists of explicitly defined elements (elements created via calls to the API) and implicitly defined elements (elements additionally created by preCICE).
As an example, explicitly defining a triangle ABC via the API guarantees the existence of the implicit edges AB, AC, and BC.

Furthermore, all connectivity is defined only via vertex IDs. There are no more edge IDs to worry about.
The order of vertices also does not matter. Triangles BAC and CAB are considered duplicates and preCICE removes one of them during the deduplication step.

The API for defining individual connectivity elements looks as follows:

```cpp
void setMeshEdge(int meshID, int firstVertexID, int secondVertexID);
void setMeshTriangle(int meshID, int firstVertexID, int secondVertexID, int thirdVertexID);
void setMeshQuad(int meshID, int firstVertexID, int secondVertexID, int thirdVertexID, int fourthVertexID);
void setMeshTetrahredron(int meshID, int firstVertexID, int secondVertexID, int thirdVertexID, int fourthVertexID);
```

Each of the above functions is accompanied by a bulk version, which allows to set multiple elements in a single call.

```cpp
void setMeshEdges(int meshID, int size, int* vertices);
void setMeshTriangles(int meshID, int size, int* vertices);
void setMeshQuads(int meshID, int size, int* vertices);
void setMeshTetrahedra(int meshID, int size, int* vertices);
```
