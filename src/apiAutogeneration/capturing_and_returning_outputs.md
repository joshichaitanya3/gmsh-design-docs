# Capturing and returning the C outputs

Handling outputs is going to be a _little_ more involved than the inputs. We need to do three things:

1. Initialize pointers to the outputs so that they can be passed to the `gmsh` C functions.
2. Capture the outputs from the C functions into a Morpho value format.
3. Return the captured outputs.

Similar to the input, we create an `outputArg` object, inheriting from `arg`, that defines methods for each of these operations:

```python
# GenApi.py
class outputArg(arg):
    """
    Basic datatype for an output argument, inherited from `arg`.
    Defines some extra methods to process the output.
    """

    # Initialize pointers to the outputs so that they can be passed to the gmsh C functions.
    def init_output(self):
        return str("")

    # Capture the outputs from the C functions into a Morpho value format.
    def capture_output(self):
        return str("")

    # Return the captured outputs.
    def return_output(self):
        return str("")
```

The specific output types then inherit from `outputArg` and override the corresponding methods.

We will again choose a simple function to illustrate this, albeit with a composite return object: a list of integers:

```c
/* Get the list of all fields. */
GMSH_API void gmshModelMeshFieldList(int ** tags, size_t * tags_n,
                                     int * ierr);
```

Here, we have no inputs and a single output `tags` that's a list of integers. Here is how it is added to the API from `gen.py`:

```python
# gen.py
doc = '''Get the list of all fields.'''
field.add('list', doc, None, ovectorint('tags'))
```

The `oarg` class is initialized with certain handy attributes to help with the output processing. In addition to the ones we have seen before, we add a new `self.cToMorphoConverter` attribute to help convert the C outputs to Morpho outputs. E.g.:

```python
# GenApi.py
class ovectorint(oint):
    def __init__(self, name, value=None, python_value=None, julia_value=None):
        arg.__init__(self, name, value, python_value, julia_value, "std::vector<int> &",
                "int **", True)
        self.c = "int ** " + self.name + ", size_t * " + self.name + "_n"
        self.morpho = "&" + self.name + ", &" + self.name + "_n"
        self.texi_type = "vector of integers"
        self.morpho_object = self.name + "_list"
        self.elementType = "int"
        self.cToMorphoConverter = "MORPHO_INTEGER"

```

The `init_output` method initializes pointers to the outputs so that they can be passed to the gmsh C functions:

```python
# GenApi.py, class ovectorint
    def init_output(self):
        return INDENTL1 \
                + f"{self.elementType}* {self.name};\n" \
                + INDENTL1 \
                + f"size_t {self.name}_n;\n"
```

The above will generate the C code:

```c
    int* tags;
    size_t tags_n;
```

The `capture_output` method captures the outputs from the C functions into a Morpho value format:

```python
# GenApi.py, class ovectorint
    def capture_output(self):
        return INDENTL1 + f"value {self.name}_value[(int) {self.name}_n];\n" \
            + INDENTL1 + f"for (size_t j=0; j<{self.name}_n; j++) " +"{ \n" \
            + INDENTL2 + f"{self.name}_value[j] = {self.cToMorphoConverter}({self.name}[j]);\n" \
            + INDENTL1 + "}\n" \
            + INDENTL1 + f"objectlist* {self.morpho_object} = object_newlist((int) {self.name}_n, {self.name}_value);\n"
```

The above will generate the C code:

```c
    value tags_value[(int) tags_n];
    for (size_t j=0; j<tags_n; j++) {
        tags_value[j] = MORPHO_INTEGER(tags[j]);
    }
    objectlist* tags_list = object_newlist((int) tags_n, tags_value);
```

The `return_output` method returns the captured outputs:

```python
# GenApi.py, class ovectorint
    def return_output(self):
        return INDENTL1 + "value out;\n" \
            + INDENTL1 + f"if ({self.name}_list) " + "{\n" \
            + INDENTL2 + f"out = MORPHO_OBJECT({self.name}_list);\n" \
            + INDENTL2 + f"morpho_bindobjects(v, 1, &out);\n" \
            + INDENTL1 + "}\n" \
            + INDENTL1 + "return out;\n"
```

which generates

```c
    value out;
    if (tags_list) {
        out = MORPHO_OBJECT(tags_list);
        morpho_bindobjects(v, 1, &out);
    }
    return out;
```

> **Note**: A few Gmsh C functions have a non-void return type, and hence return >something directly instead of through pointers. For example,
>
> ```c
> /* Return 1 if the Gmsh API is initialized, and 0 if not. */
> GMSH_API int gmshIsInitialized(int * ierr);
> ```
>
> These are handled directly while writing the module.

<div class="warning">

The handling of cases where the function returns outputs both directly and via pointers is not implemented yet. But as of 4.12.2 (at the time of writing), there is only one function, `gmshFltkSelectEntities`, in the entire API that does this, and since it is an FLTK-related function, we will probably not use it through the API in Morpho. This should still be implemented in the future.

</div>

Here are the Morpho values types returned for the corresponding C types:

| C-Type   | Morpho-Type      |
| -------- | ---------------- |
| `int`    | `MORPHO_INTEGER` |
| `size_t` | `MORPHO_INTEGER` |
| `double` | `MORPHO_FLOAT`   |
| `char *` | `MORPHO_STRING`  |

All vector outputs are returned as Morpho `List`'s. For example, an `ovectordouble` would be returned as a Morpho `List` of Morpho `Float`s, `ovectorvectorint` would be `List` of `List` of `MORPHO_INTEGER`'s, and so on. For the list of lists, we need a separate operation to collect all the outputs and return it. We write functions for this inside our `write_morpho` method:

```python
def collect_list_of_outputs(oargs):
    l = INDENTL1 + f"value outval[{len(oargs)}];\n"
    for j, oarg in enumerate(oargs):
        l += INDENTL1 + f"outval[{j}] = MORPHO_OBJECT({oarg.morpho_object});\n"
    l += INDENTL1 + f"objectlist* outlist = object_newlist({len(oargs)}, outval);\n"
    self.fwrite(f, l)
    return

def return_list_of_outputs():
    l = INDENTL1 + "value out;\n" \
      + INDENTL1 + f"if (outlist) " + "{\n" \
      + INDENTL2 + f"out = MORPHO_OBJECT(outlist);\n" \
      + INDENTL2 + f"morpho_bindobjects(v, 1, &out);\n" \
      + INDENTL1 + "}\n" \
      + INDENTL1 + "return out;\n"
    self.fwrite(f, l)
```
