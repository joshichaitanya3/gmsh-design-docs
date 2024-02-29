## Capturing Morpho inputs

To see how we handle inputs, let's continue with our example of the `gmshModelSetCurrent` function that we looked at in [genApi.py](../gmshApi/genApi.md). This function takes in a ```char``` array containing the desired name for the current model and sets it internally, returning nothing. Hence, this function has one `String` input and no outputs. Below is its declaration in the C-API, `gmshc.h`:

```c
/* Set the current model to the model with name `name'. If several models have
 * the same name, select the one that was added first. */
GMSH_API void gmshModelSetCurrent(const char * name,
                                  int * ierr);
```
*`gmsh/api/gmshc.h`*

(Note: the last argument for all the C-API functions is a reference to an integer. This stores the error code.) Here is how it is added to the API from `gen.py`:

```python
doc = '''Set the current model to the model with name `name'. If several models have the same name, select the one that was added first.'''
model.add('setCurrent', doc, None, istring('name'))
```
*`gmsh/api/gen.py`*

In Morpho, we want a call signature like
```javascript
gmshModelSetCurrent(name)
```

To execute this, we need to do a few things:
1. Check that the number of arguments passed to the _Morpho_ ```gmshModelSetCurrent``` method is 1. Otherwise, raise an error.
2. Then check that this argument is indeed a Morpho String. Otherwise, raise an error.
3. If all is well, then _capture the input_ into a C-string, or a ```char``` array.
4. Only then can we actually call the C-function.
5. Return ```nil``` since this function doesn't output anything.

Phew, that's a bunch! But we are going to automate this process. 

Based on the list above, we want this method to be wrapped in Morpho something like this:

```c
// myextension.c
#include <stdio.h>
#include <morpho/morpho.h>
#include <morpho/builtin.h>
// Let's not forget to include the gmsh C API header file!
#include <gmshc.h>
// Let's include our own header file
#include "gmshapi.h"

value MorphoGmshModelSetCurrent(vm *v, int nargs, value *args) {
    // Check whether only 1 argument is supplied and raise error if not
    if (nargs != 1) {
        morpho_runtimeerror(v, GMSH_NARGS_ERROR);
        return MORPHO_NIL;
    } 
    // Check whether the argument is a Morpho String and raise error if not
    if (!MORPHO_ISSTRING(MORPHO_GETARG(args, 0))) {
        morpho_runtimeerror(v, GMSH_ARGS_ERROR); 
        return MORPHO_NIL; 
    }
    // If we don't raise error up to this point, we have the right number and kind of input. Capture the input Morpho string into a char array.
    const char * name = MORPHO_GETCSTRING(MORPHO_GETARG(args, 0)); 
    // Actually call the C function
    int ierr;
    gmshModelSetCurrent(name,
                      &ierr);
    // Since there is no output, we return nil
    return MORPHO_NIL;
}

```
*A wrapper for `gmshMorphoSetCurrent`*

Let's see how we can automate this process. To check the number of arguments, we note that the arguments are collected by the `Module` object as a list:
```python
class Module:
    def __init__(self, name, doc):
        self.name = name
        self.doc = doc
        self.fs = []
        self.submodules = []

    def add(self, name, doc, rtype, *args):
        self.fs.append((rtype, name, args, doc, []))
```
*From `gmsh/api/GenApi.py`*

Further, all the output arguments have the `out` property set to `True` (this is the last argument to the `arg` and its subclasses' initializer). We can use this to grab and sort the arguments:

```python
def write_morpho(self):
    ...
    def write_module(module, c_namespace):
        ...
        for rtype, name, args, doc, special in module.fs:

            iargs = list(a for a in args if not a.out)
            oargs = list(a for a in args if a.out)
```
*From `gmsh/api/GenApi.py`*

Thus, the code for the number of arguments check is as simple as this:
```python
nargsCheck =  INDENTL1 + f"if (nargs != {len(iargs)})"+ " {\n" \
            + INDENTL2 + "morpho_runtimeerror(v, GMSH_NARGS_ERROR);\n" \
            + INDENTL2 + "return MORPHO_NIL;\n" \
            + INDENTL1 + "} \n"
```

Here, the `GMSH_NARGS_ERROR` will be defined in the `gmshapi.h` header file, together with the error message `GMSH_NARGS_ERROR_MSG` like so:
```c
// morpho-gmsh/src/gmshapi.h
#define GMSH_NARGS_ERROR "GmshNargsErr"
#define GMSH_NARGS_ERROR_MSG "Incorrect Number of arguments for Gmsh function. Check the help for this function."
```

The error itself will be initialized in the `gmshapi.c` file in the `gmshapi_initialize` function as follows:
```c
// morpho-gmsh/src/gmshapi.c
morpho_defineerror(GMSH_NARGS_ERROR, ERROR_HALT, GMSH_NARGS_ERROR_MSG);
```

Now, for each input argument, we need to (a) check its type and (b) capture it into a C variable. We will do this by endowing all input arguments with a `capture_input` method that will return the C code to do this. To do this, we first make a subclass of `arg` called `inputArg` which defines this method:

```python
# morpho-gmsh/api/GenApi.py
class inputArg(arg):
    """
    Basic datatype for an input argument, inherited from `arg`. Provides some extra attributes and methods to process the input.
    """
    def __init__(self, name, value, python_value, julia_value, cpp_type, c_type, out):
        super().__init__(name, value, python_value, julia_value, cpp_type, c_type, out)

        # C-code to generate specific runtime error if the arguments are not correct. To-do: Currently, all inputs return the same error. Change this so that the error is function-specific or more helpful in general.
        self.runTimeErr = " {morpho_runtimeerror(v, GMSH_ARGS_ERROR); return MORPHO_NIL; }\n"
        # morphoTypeChecker is the builtin (or otherwise) Morpho function to check whether the input value is of the correct type.
        self.morphoTypeCheker = "MORPHO_ISINTEGER" # For default, using the integer case
        # morphoToCConverter is the builtin (or otherwise) Morpho function to grab the correct C-type from the input Morpho value.
        self.morphoToCConverter = "MORPHO_GETINTEGERVALUE"


    def capture_input(self, i):
        """
        capture_input(i)

        Generates C-code to check the (i+1)th input argument to the Morpho function and convert it to an appropriate C-type.
        """
        
        # Here, we check for a single object as defualt, which can be reused for anything that's not a list: `iint`, `idouble, `istring`, etc.
        chk = INDENTL1 + f"if (!{self.morphoTypeCheker}(MORPHO_GETARG(args, {i}))) " + self.runTimeErr
        inp = INDENTL1 + f"{self.c_type} {self.name} = {self.morphoToCConverter}(MORPHO_GETARG(args, {i})); \n"
        return chk + inp

```
We see that in addition to the attributes of `arg`, we add two more: `morphoTypeChecker` and `morphoToCConverter`. These are the names of the Morpho functions that will check the type of the input and convert it to a C-type, respectively. The `capture_input` method then uses these to generate the C-code to do this.

While the default values for these are for an integer, we can override these in the subclasses. For example, the `istring` object is initialized as follows:
```python
# morpho-gmsh/api/GenApi.py
class istring(inputArg):
    def __init__(self, name, value=None, python_value=None, julia_value=None, cpp_type="const std::string &", c_type="const char *", out=False):
        super().__init__(name, value, python_value, julia_value, cpp_type, c_type, out)
        self.texi_type = "string"
        self.morphoTypeCheker = "MORPHO_ISSTRING"
        self.morphoToCConverter = "MORPHO_GETCSTRING"
```

More complex objects, like `ivectorint` (which captures a list of integers), will have a more complex `capture_input` method. This allows us just process all the inputs simply as:
```python
# morpho-gmsh/api/GenApi.py
# Inside the `write_module` method

# Capture all the inputs
for i,iarg in enumerate(iargs):
    self.fwrite(f, iarg.capture_input(i))
```

In the next chapter, we will look at how we handle output arguments.