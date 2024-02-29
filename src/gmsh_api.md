# The Gmsh API

Broadly, `gmsh` is divided into 4 components: geometry, mesh, solver and post-processing (see the Gmsh reference manual). 
1. Geomery: A model in Gmsh is defined using its Boundary Representation (BRep). This geometry is specified through a built-in kernel (`geo`) and/or an _OpenCASCADE_ kernel (`occ`). This module allows us to define our domain and perform various operations like Boolean combinations, extrusions, translations etc. without actually meshing the domain.

2. Mesh: The mesh module provides various algorithms for generating conformal meshes for the domains. The basic entities used in Gmsh are lines, triangles, quadrangles, tetrahedra, prisms, hexahedra and pyramids, and the meshes are by default unstructured. This is great for Morpho since we can always generate a triangular (in 2D) or tetrahedral (in 3D) mesh from Gmsh and export to the Morpho Mesh format (this is done separately in the higher level module.) 

3. Solver: The solver module allows exchange of data with external solvers or other clients through the ONELAB interface.

4. Post-processing: The post-processing module allows viewing scalar/tensorial field data on top of the meshes.

Gmsh is written in C++, but provides APIs for several languages like C, Python, Julia and Fortran. In the API, the components mentioned above are organized into modules and submodules as follows:


- `gmsh`
  - `option`
  - `model`
    - `mesh`
      - `field`
    - `geo`
      - `mesh`
    - `occ`
      - `mesh`
  - `view`
    - `option`
  - `plugin`
  - `graphics`
  - `fltk`
  - `parser`
  - `onelab`
  - `logger`


In the `C` API, the function names are given using the lower camel case. For example, the `addPoint` function in the `geo` submodule of `model` will be called `gmshModelGeoAddPoint`. We maintain the same call signature for the low-level Morpho API.

The APIs are auto-generated using two scripts: 

1. [`gen.py`]("https://gitlab.onelab.info/gmsh/gmsh/blob/master/api/gen.py"): This file defines all the modules and submodules of the Gmsh library as noted above, and adds each method with its input and output (handled by special objects), along with its documentation to an `API` object. This object then writes the API files for all the languages through methods like ```write_julia()```, ```write_c()```, etc. This `API` object is defined in the next file:
2. [`GenApi.py`]("https://gitlab.onelab.info/gmsh/gmsh/blob/master/api/GenApi.py"): This is where the bindings are generated. The `Module` object is defined, which holds the name, documentation and a list of submodules, and provides a method to add (sub)modules to it. The `API` object provides methods to add modules, write to files, and write language-specific APIs. It also defines the `arg` object that handles the nature of the input and output arguments, providing apparatus for various languages.

In `morpho-gmsh`, we obtain `gen.py` from the stable version we desire, and provide our own `GenApi.py` that writes only the Morpho API. To use `gen.py` without modification, our ```write_python()``` does the job of producing the Morpho API, while the rest of them (```write_julia()```, etc) just ```pass```.

Now, let's take a deep dive into these two files and see how we can use them to generate the Morpho API.