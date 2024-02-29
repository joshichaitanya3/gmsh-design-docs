# Skeleton of a Morpho wrapper extension

To see how a simple wrapper extension can be written, let's assume we want to wrap a C function `integer_add` that adds two integers and returns the output as an integer. Here is how we would write the wrapper in C:

## C file

```c
// intadd.c
#include <stdio.h>
#include <morpho/morpho.h>
#include <morpho/builtin.h>
#include "intadd.h"

// The C function we wish to wrap. This could be in a saparate file, in which case we would include its header file here, but we are keeping it simple for now.
int integer_add(int a, int b) {
    return a + b;
}

// The Morpho veneer function
value MorphoIntegerAdd(vm *v, int nargs, value *args) {
    // Check the number of arguments
    if (nargs != 2) {
        morpho_runtimeerror(v, INTADD_NARGS_ERROR);
        return MORPHO_NIL;
    }
    // Check the types of the arguments
    if (!MORPHO_ISINTEGER(args[0])) {
        morpho_runtimeerror(v, INTADD_TYPE_ERROR);
        return MORPHO_NIL;
    }
    if (!MORPHO_ISINTEGER(args[1])) {
        morpho_runtimeerror(v, INTADD_TYPE_ERROR);
        return MORPHO_NIL;
    }
    // Call the C function and return the result
    int a = MORPHO_GETINTEGERVALUE(args[0]);
    int b = MORPHO_GETINTEGERVALUE(args[1]);
    return MORPHO_INTEGER(integer_add(a, b));
}

void myextension_initialize(void) {
    builtin_addfunction(MORPHO_INTEGERADD_FUNCTION, MorphoIntegerAdd, BUILTIN_FLAGSEMPTY);
    morpho_defineerror(INTADD_NARGS_ERROR, ERROR_HALT, INTADD_NARGS_ERROR_MSG);
    morpho_defineerror(INTADD_ARGS_ERROR, ERROR_HALT, INTADD_ARGS_ERROR_MSG);
}

void myextension_finalize(void) {
    // Nothing to do here
}
```

The `value` object is the basic Morpho object that can hold any type of value like Lists, integers, floats, Strings, etc., even including `nil`. Every _Morpho_ veneer function returns a _Morpho_ `value` object. Further, `nargs` is the number of arguments supplied, and `args` is a list of Morpho `value`'s. We can see that we have performed type checking on the arguments and raised errors if the wrong number or type of arguments are supplied. We have also captured the arguments into C variables and called the C function. Finally, we have returned the result as a Morpho `value` object.

All _Morpho_ extensions must provide an initialize function, named EXTENSIONNAME_initialize. In this function, the morpho API is used to define functions and/or classes implemented, and set up any global data as necessary. Since we will be wrapping functions, our extension does not have any classes. Here, we add a function to the runtime that will be visible to user code as `MORPHO_INTEGERADD_FUNCTION`. This is defined as a macro instead of hardcoding the function name to make it easier to change the function name in the future or use it elsewhere. The last argument to `builtin_addfunction` is a flag that tells the morpho runtime whether the function is a method or a standalone function. Since this is a standalone function, we pass `BUILTIN_FLAGSEMPTY`. Similarly, we define errors that will be raised in the veneer function, whose name and message are also macros.

The finalize function has a similar naming convention, and is not strictly necessary (as here). This function should deallocate or close anything created by your extension that isn’t visible to the morpho runtime: we will actually use it here to finalize gmsh itself.

The macros are defined in our extension's header file `intadd.h`:

## Header File

```c
// intadd.h
#include <stdio.h>
#include <morpho/morpho.h>
#include <morpho/builtin.h>

#define INTADD_NARGS_ERROR "IntAddNArgsError"
#define INTADD_NARGS_ERROR_MSG "Wrong number of arguments supplied to integer_add"

#define INTADD_TYPE_ERROR "IntAddTypeError"
#define INTADD_TYPE_ERROR_MSG "integer_add requires two integer arguments"

#define MORPHO_INTEGERADD_FUNCTION "addInts"
```

Note that from the last line, the function is given the name `addInts`, so it would be called that _in Morpho_:

```javascript
var a = 3;
var b = 4;
var c = addInts(a, b);
```

However, there is nothing stopping us from calling it `integer_add` itself. Having

```c
#define MORPHO_INTEGERADD_FUNCTION "integer_add"
```

would make the last line

```javascript
var c = integer_add(a, b);
```

This is in fact what we will do for our Gmsh extension. We will call the functions in the C API by the same names as in the original Gmsh API. This will make it easier for users to use the Morpho APIs without having new names.

Our extension is almost ready, save for documenting the function. This is done in Morpho using the Markdown format in the `share/help` directory. We will see how to do this in the next chapter. For our simple extension, it would look like this:

## Help file

```markdown
[comment] # Path: share/help/intadd.md
[comment]: # (Intadd help)
[version]: # (0.1)

# Intadd

[tagIntadd]: # "Intadd"

The `intadd` extension provides a method to add two integers.

[showsubtopics]: # "subtopics"

## addInts

[tagaddints]: # "AddInts"

The `addInts` function adds two integers and returns the result.

    var c = addInts(1, 2)
    print c // 3
```

This makes it so that the user can access the help for this module in the REPL using

```javascript
>help intadd // or
>help intadd.addInts // etc
```

This is how we will document our Gmsh extension as well. For every gmsh function, we need the veneer, the initialization, the header file declaration and the documentation. We will see how to do this in the next chapter.
