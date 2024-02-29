# Installation instructions

1. This extension requires `gmsh` to be installed as a dynamic library. Download the source code of the latest stable release of gmsh from https://gitlab.onelab.info/gmsh/gmsh/-/releases.

1. Build gmsh from source with the dynamic build flag turned on:
    ```bash
    cd build
    cmake -DENABLE_BUILD_DYNAMIC=1 ..
    make
    make install
    ```
1. In `apigenerator/generate.py`, set the correct version of gmsh that you have downloaded by modifying line 7:

    ```python
    api_version = "gmsh_4_12_2" # For example
    ```

1. Run the file:

    ```bash
    python generate.py
    ```

1. Navigate to `morpho-gmsh`

1. Build the extension by running the following:
    ```bash
    mkdir build
    cd build
    cmake -DCMAKE_BUILD_TYPE=Release ..
    make install
    ```

1. Add the extension path to `~/.morphopackages`:
    ```bash
    cd ..
    pwd >> ~/.morphopackages
    ```

1. Try out the test example
    ```bash
    cd examples
    morpho6 testgmshapi.morpho
    ```
