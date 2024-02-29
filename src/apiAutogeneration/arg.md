# The arg class

In the simple example in the last chapter, we saw the processing of the input and output arguments that we needed to do to create the veneer. In the Gmsh API generator, all arguments are initialized as the `arg` class or one of its subclasses. We simplify this object to strip off most of the boilerplate code for the other language APIs and add in attributes for the Morpho API. Our `arg` class has the following structure:

```python
# Removing all the Python, Julia and Fortran related attributes, but
# keeping the function call signatures the same in order to use gen.py
# without editing.
class arg:
    """ Basic datatype of an argument. Every datatype inherits this constructor
    and also default behaviour from here.

    Compared to the original, this object only stores the C-related information (to actually call the C-API functions) and Morpho-specific information.

    Subclasses of this constructor will have methods for processing inputs/outputs, etc.

    """
    def __init__(self, name, value, python_value, julia_value, cpp_type,
                 c_type, out):
        self.c_type = c_type
        self.name = name
        if (name == "value"): # value is a builtin type in Morpho, so change it slightly
            self.name = "cvalue"
            name = "cvalue"
        if (name == "v"): # Similarly, v is used for the virtual machine (vm) in Morpho
            self.name = "cv"
            name = "cv"
        self.value = value
        self.out = out # Boolean to indicate if the argument is an output
        self.c = c_type + " " + name
        self.texi_type = ""
        self.texi = name + ""
        # self.morpho will be used in place of self.c while calling the C-API functions, since the inputs will be slightly different while calling from the Morpho wrappers.
        self.morpho = name
        # self.morpho_object specifies how to name the Morpho Objects corresponding to the args. For instance, a string would have morpho_object as name + "_str", so that it is declared as `objectstring * name_str = ...`
        self.morpho_object = name

```

As noted in the comments in the code, the `self.morpho` attribute will be used in place of `self.c` while calling the C-API functions, since the inputs will be slightly different while calling from the Morpho wrappers. The `self.morpho_object` attribute specifies how to name the Morpho Objects corresponding to the args. For instance, a string would have morpho_object as name + "\_str", so that it is declared as `objectstring * name_str = ...`.

The original `GenApi.py` file creates all the objects as _instances_ of the `arg` class, like so:

```python
# Original GenApi.py
def iint(name, value=None, python_value=None, julia_value=None):
    a = arg(name, value, python_value, julia_value, "const int", "const int",
            False)
    a.python_arg = "c_int(" + name + ")"
    a.julia_ctype = "Cint"
    a.fortran_types = ["integer, intent(in)"]
    a.fortran_c_api = ["integer(c_int), value, intent(in)"]
    a.fortran_call = f"{name}=optval_c_int({value}, {name})" if value is not None else f"{name}=int({name}, c_int)"
    a.texi_type = "integer"
    return a
```

In contrast, we will create all other arguments as subclasses or sub-subclasses of this `arg` object. We will see in the next chapters that our way allows us to write additional _methods_ to the subclasses to help write the Morpho API.
