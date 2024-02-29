# Some To-do's

Here are some desirable features not implemented yet.

## The `gmshapi` module

1. The `ivoidstar` (pointer) input needs to be handled. This occurs only in one function as of version 4.12.2, the `gmshModelOccImportShapesNativePointer` method. As this is not implemented yet, we skip this method in the API.
2. Same as above, but for the `isizefun` input (a function input). This also occurs for one function, `gmshModelMeshSetSizeCallback`, that we skip currently.
3. The `ivoidstar` and `isizefun` need to be implemented also as subclasses of `inputArg`, so that we can skip any new function that has this input by looking for the input's class name. (or just finish 1. and 2.!)
4. Handle functions which return both directly (an `int`, for example, in `gmshIsInitialized`) and via pointers. Currently, we skip them (again, only one function in the FLTK module, so doesn't affect our use case all that much right now).
5. Add more specific `Error`'s. Currently, there is a single error for incorrect type of arguments, so it is not very helpful.
6. Currently, _all_ the methods fall under `gmshapi`'s subtopics, whereas it would be nice if the help is also organized by the submodules. We need to add this functionality.

## The `gmsh` module

The `gmsh` module needs to be made in a "Morphonic" way, in the style of Morpho's own `meshgen` module. This requires a considerable thought on the design, and remains to be implemented. Currently, the `gmsh` module simply provides _one_ (pretty central) functionality: converting gmsh meshes to Morpho meshes!
