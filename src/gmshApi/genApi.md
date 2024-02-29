## `GenApi.py`

The language-agnostic nature of `gen.py`, while allowing itself to be clean and simple, transfers the complexity of the language-specific details to `GenApi.py`.

The original `gmsh` source code has a single `GenApi.py` file for all the target languages --- the `arg` objects have attributes for all the languages, and the `API` object has methods to write for each of them. 

To see this, let's look at an easy example function,  `gmshModelSetCurrent`, which sets the name of the current model --- this function has one input, a ```char *``` array, and no output. Here is how it is added to the API in `gen.py`:

```python
doc = '''Set the current model to the model with name `name'. If several models have the same name, select the one that was added first.'''
model.add('setCurrent', doc, None, istring('name'))

```
*From `gmsh/api/gen.py`*


Here is what `GenApi.py`'s `write_c` method writes in the C header file for this function:

```c
/* Set the current model to the model with name `name'. If several models have
 * the same name, select the one that was added first. */
GMSH_API void gmshModelSetCurrent(const char * name,
                                  int * ierr);
```
*From `gmsh/api/gmshc.h`*


and here are the relevant snippets from `GenApi.py` that generate this code:

```python
class arg:
    def __init__(self, name, value, python_value, julia_value, cpp_type,
                 c_type, out):
    ...
    # Note that self.c is generates the argument for the C function signature
    self.c = c_type + " " + name
    ...
...

def istring(name, value=None, python_value=None, julia_value=None):
    # Note that the arg is initialized with c_type = "const char *"
    a = arg(name, value, python_value, julia_value, "const std::string &",
            "const char *", False)
    a.python_arg = "c_char_p(" + name + ".encode())"
    ...
    a.texi_type = "string"
    return a

...
class API:
    ...
    def write_c(self):
        ...
        def write_module(module, c_namespace, cpp_namespace, indent):
            ...
            # Note that arg.c is used here to write the C function signature
            self.fwrite(f, 
              fnameapi + (",\n" + ' ' * len(fnameapi)).join(
                list((a.c for a in args + (oint("ierr"), )))) + ");\n")
        ...
    ...
...
```
*Snippets from `gmsh/api/GenApi.py`*

Let's start with the `istring` object. This is an object of the `arg` class. Note that in the initializer, the `c_type` passed is `"const char *"`. The `arg` initializer in turn uses this to set its attribute `c` to `c_type + " " + name`. This is used in the `write_c` method of the `API` object to write the call signature in the C header file. This is how the C function signature is written to the header file.

This procedure is followed for all input and output types. For example, there is an object `iint` for an input `int`, `ivectorsize` for a vector of `size_t`'s, etc. and similar for outputs, `ostring` for an output `char *` and so on. These objects have attributes that specify, among other things, the call signatures for various languages. For instance, if a function takes in an input vector of integers, then the `C` function needs _two_ inputs, (```int * list, size_t list_n```), where `list_n` encodes the size of the list. The same goes for outputs. 

We write our own `GenApi.py` that only writes the Morpho API, heavily adopting from the original. We will see how to do this in the next chapter.
