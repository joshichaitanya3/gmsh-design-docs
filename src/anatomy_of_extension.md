# Anatomy of the Gmsh Extension

As is described in the devguide, a _Morpho_ extension can either be

1. Simply a `.morpho` file that we want to be able to import from anywhere, or
2. An added functionality written in C and loaded as a shared library (imported the same way as a `.morpho` extension), or
3. A combination of both.

Normally, we would have used the Gmsh C API to write an extension, hand-picking the functionality we most desire (category \#2 above). But we are in luck, as the original Gmsh API is auto-generated, and the code to do that renders itself well to a better approach: creating a _Morpho_ API for Gmsh (by auto-generating Morpho bindings code in C), which we can then use to create a Gmsh module for Morpho written in Morpho itself.

We use Gmsh's own API generator code to create low-level Morpho bindings to the Gmsh functions (loaded as `gmshapi`). Higher-level functionality, including exporting to a Morpho mesh, is then implemented in a Morpho module (loaded as simply `gmsh`). Ideally, the end-user should only need to import the `gmsh` module, but the `gmshapi` module is also available for those who want to use the low-level Gmsh API directly (via `import gmshapi`).

Thus, our Gmsh extension is an example of category \#3: a C extension that wraps the Gmsh C API, and a `.morpho` file that uses this extension to create the `gmsh` module.

## Directory Structure

Here is the directory structure of the Gmsh extension:

```
.
├── CMakeLists.txt
├── LICENSE
├── README.md
├── apigenerator
│   ├── api
│   │   ├── GenApi.py
│   │   └── gen.py
│   └── generate.py
├── examples
│   ├── fromFile.morpho
│   ├── occ.morpho
│   ├── ...
├── lib
│   └── gmshapi.so
├── share
│   ├── help
│   │   ├── gmsh.md
│   │   └── gmshapi.md
│   └── modules
│       └── gmsh.morpho
└── src
    ├── gmshapi.c
    └── gmshapi.h
```

The `.morpho` extension file(s) go in the `share/modules` directory. The C source and header files go in the `src` directory. The shared library (which will be generated upon compiling the C source) goes in the `lib` directory. The documentation to be made available for inline help goes in the `share/help` directory. This should follow the _Morpho_ inline documentation Markdown format (see the devguide). The documentation for the higher level `gmsh` module is in `gmsh.md`, and the documentation for the low level `gmshapi` module is in `gmshapi.md` (which is auto-generated).

The `apigenerator` directory contains the scripts used to generate the Morpho API from the Gmsh C API. The `generate.py` script is used to generate the Morpho API from the Gmsh C API. It downloads `gen.py` from a stable Gmsh release and runs it using our own `GenApi.py` file. This creates the following three files:

1. `src/gmshapi.c` (the C source file for the extension)
2. `src/gmshapi.h` (the C header file for the extension)
3. `share/help/gmshapi.md` (the inline documentation for the low level Morpho API)

Building the project using Cmake generates the shared library `lib/gmshapi.so` from the C source file.
