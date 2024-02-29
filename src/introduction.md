# Introduction

This is a design document for creating an extension that enables the use of the open-source mesh generation library [`gmsh`]("https://gmsh.info/") in [Morpho]("https://github.com/Morpho-lang/morpho").

The extension is available [here](https://github.com/Morpho-lang/morpho-gmsh) and works with Morpho version 0.6.0. The version of Gmsh used at the time of writing is 4.12.2.

## Example use

Here is an example of how the `gmsh` module can be used.

```javascript
import gmsh

var gm = GmshLoader("t1.msh") // Load a .msh (gmsh's format) file in Gmsh
gm.launch() // Launch the Gmsh app, where the mesh can now be modified
gm.exportToMorpho("t1Modified.mesh") // Save the modified mesh in Morpho's .mesh format
```

_Excerpt from `examples/fromFile.morpho`_

The Morpho Gmsh API itself can be used by importing the `gmshapi` module:

```javascript
import gmshapi
import plot

gmshInitialize(0,0,0)
var coneHeight = 1
var coneRadius = 1
var coneTag = gmshModelOccAddCone(0,0,0, 0,0,coneHeight, coneRadius,0, -1, 2*Pi)
gmshModelOccSynchronize()
gmshOptionSetNumber("Mesh.MeshSizeMax", 0.2)
gmshModelMeshGenerate(3)
var m = GmshLoader().buildMorphoMesh()
Show(plotmesh(m, grade=1))
gmshFinalize()
```

_Excerpt from `examples/occ.morpho`_

This allows for a different implementation of the `gmsh` module or other functionalities.

## Organization

The installation instructions are in [Chapter 2](installation.md). A simple example of a Morpho wrapper extension is discussed in [Chapter 3](anatomy_of_extension.md). The structure of the Gmsh API is discussed in [Chapter 4](gmsh_api.md). The creation of Morpho bindings for the Gmsh API is discussed in [Chapter 5](api_autogeneration.md). In [Chapter 6](gmsh_in_morpho.md), we will discuss the implementation of the `gmsh` module in Morpho, which will use the Morpho Gmsh API to generate higher-level functionality. While this could also have been auto-generated similar to the Python API, we create this manually in a "Morphonic" object-oriented style. We document some to-do's in [Chapter 7](to_dos.md).

## Acknowledgements

The design of this Morpho-specific binding was improved by inspirations from a [Haskell Binding to the Gmsh API by Antero Marjamäki](https://github.com/Ehtycs/haskell-gmsh/).

_This is a pretty unusual piece of code for me, which required understanding the organization of the parent `gmsh` codebase. This document is rather verbose, and is mostly so for my own benefit. Apologies if it beats around the bush too much!_
