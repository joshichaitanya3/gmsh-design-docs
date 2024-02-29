## `gen.py` 

This file, under `gmsh/api/gen.py`, has the following structure:

```python
// gen.py
from GenApi import * # Contains all the relevant objects and methods
...
api = API(version_major, version_minor, version_patch)

gmsh = api.add_module('gmsh', 'top-level functions')

doc = '''Initialize the Gmsh API ... ''' 
gmsh.add('initialize', doc, None, iargcargv(), ibool('readConfigFiles', 'true', 'True', 'true'), ibool('run', 'false', 'False'))

doc = '''Return 1 if the Gmsh API is initialized, and 0 if not.'''
gmsh.add('isInitialized', doc, oint)
...
option = gmsh.add_module('option', 'option handling functions')

doc = ''' ... '''
option.add(...)
...
model = gmsh.add_module('model', ...)
...
model.add('add', doc, None, istring('name'))
...
mesh = model.add_module('mesh', 'mesh functions') # Submodule of 'model'
...
api.write_c()
api.write_python()
...
```
*Outline of `gmsh/api/gen.py`*

The top level API object collects all the modules and submodules and writes them to various target languages. Its `add_module` method creates and returns a `Module` object, which has its own `add_module` method. The `add` method of the `Module` object takes in the name of the method, its documentation, the return type, and the input and output arguments. The I/O arguments are passed as objects of the `arg` class. Note that these are target-language agnostic (aside from providing default values for some languages), and the separate `write` methods of the `API` object handle the language-specific details. This allows `gen.py` to remain the same for all languages, and only the `GenApi.py` file needs to be modified for each language. 

In light of this, we will aim to reuse the original `gen.py` *as is*, and only modify `GenApi.py` for our purposes.