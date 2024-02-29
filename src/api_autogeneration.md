# API auto-generation

Now that we understand how the original Gmsh API is generated, let's work towards creating Morpho bindings for it. Since our Morpho API is going to be wrapping the *C* API functions around a Morpho veneer (as Morpho is written in C), our `GenApi.py` script will auto-generate the *veneers* for the C functions. (Since Morpho does not have a native Foreign Function Interface (FFI) yet, we will have to write the veneers in C. Although, the machinery we build here could be useful for building a future FFI for Morpho.) In addition, we also want inline documentation for the Morpho API, which should also be auto-generated from the original Gmsh API documentation.

Hence, our `GenApi.py` script will have to do the following tasks:

1. Generate the Morpho veneers for the C functions.
1. Create the header file with the function names and error names for the Morpho API.
1. Generate the inline documentation for the Morpho API.

In this chapter, we will first sketch out a simple example of how a veneer extension can be written for Morpho. Then we will discuss how we can auto-generate the veneers and the inline documentation for the Morpho using our `GenApi.py` script. 
